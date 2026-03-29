# Media

[← Retour au sommaire](./SUMMARY.md)

Ce document présente le système de gestion des médias (images, fichiers) dans l'API. Le système permet l'upload, le stockage et la gestion de fichiers avec support des médias publics et privés.

## Vue d'ensemble

L'API utilise un système de médias robuste qui permet :
- **Upload de fichiers** avec validation et traitement automatique
- **Redimensionnement d'images** avec Sharp pour optimiser les performances
- **Stockage séparé** entre médias publics et privés
- **Gestion des métadonnées** en base de données avec Prisma
- **Sécurité** avec protection contre les attaques de path traversal

### Technologies utilisées

- **Sharp** : Traitement et redimensionnement d'images
- **Prisma** : Gestion des métadonnées en base de données
- **Express** : Servir les fichiers statiques publics
- **Multer** : Upload de fichiers (intégré dans NestJS)
- **ParseFilePipe** : Validation des fichiers uploadés

## Structure de la base de données

### Table Media (media)

Stocke les métadonnées de tous les fichiers uploadés.

| Champ | Type | Description | Contraintes |
|-------|------|-------------|-------------|
| `id` | String (UUID) | Identifiant unique | PK, auto-généré |
| `name` | String | Nom du fichier | Obligatoire |
| `url` | String | Chemin d'accès au fichier | Obligatoire |
| `mimeType` | String | Type MIME du fichier | Obligatoire |
| `size` | Int | Taille en octets | Obligatoire |
| `isPrivate` | Boolean | Fichier privé ou public | Défaut: false |
| `createdAt` | DateTime | Date de création | Auto, défaut: now() |
| `updatedAt` | DateTime | Date de mise à jour | Auto-update |

### Table UserMedia (user_media)

Table de liaison entre utilisateurs et médias pour les images de profil.

| Champ | Type | Description | Contraintes |
|-------|------|-------------|-------------|
| `id` | String (UUID) | Identifiant unique | PK, auto-généré |
| `userId` | String | ID de l'utilisateur | FK → User, unique |
| `mediaId` | String | ID du média | FK → Media |
| `createdAt` | DateTime | Date de création | Auto, défaut: now() |
| `updatedAt` | DateTime | Date de mise à jour | Auto-update |

### Relations

- `User.profilePicture` → **UserMedia** (1:1) : Image de profil
- `Media.userMedia` → **UserMedia[]** (1:N) : Associations avec utilisateurs
- `UserMedia.user` → **User** : Utilisateur propriétaire
- `UserMedia.media` → **Media** : Média associé (CASCADE on delete)

## Types de médias

### Enum mediaTypeEnum

```typescript
export enum mediaTypeEnum {
  PROFILE_PICTURE = 'users_profilePicture',
}
```

Les types de médias définissent la structure des dossiers et l'organisation des fichiers :
- `users_profilePicture` → Dossier `users/profilePicture/`

## Structure des fichiers

### Organisation des dossiers

```
api/
├── public/                    # Médias publics
│   └── users/
│       └── profilePicture/    # Images de profil publiques
├── private/                   # Médias privés
│   └── users/
│       └── profilePicture/    # Images de profil privées
```

### Nommage des fichiers

Les fichiers sont nommés selon le pattern :
```
{mediaType}_{timestamp}_{randomString}.{extension}
```

Exemple : `users_profilePicture_1703123456789_abc123def.jpg`

## Public

Les médias publics sont accessibles directement via URL sans authentification.

### Configuration

Dans `main.ts`, les fichiers publics sont servis via Express :

```typescript
app.useStaticAssets(join(process.cwd(), MEDIA_FOLDER.PUBLIC), {
  prefix: '/public/', // accès via /public/monfichier.jpg
});
```

### Accès

- **URL** : `{API_URL}:{API_PORT}/public/users/profilePicture/image.jpg`
- **Authentification** : Non requise
- **Stockage** : Dossier `api/public/`
- **Base de données** : `isPrivate = false`

### Utilisation

```typescript
// Upload d'un média public
const mediaData = await mediaService.uploadMedia(
  file,
  mediaTypeEnum.PROFILE_PICTURE,
  { maxWidth: 256, maxHeight: 256, quality: 85 },
  false // isPrivate = false
);

// Sauvegarde en base
const media = await mediaService.createMedia(mediaData);
```

## Private

Les médias privés nécessitent une authentification et des vérifications d'autorisation pour être accessibles.

### Configuration

Les médias privés sont stockés dans le dossier `api/private/` et ne sont pas servis directement par Express.

### Accès

- **URL** : `{API_URL}:{API_PORT}/media/private/{mediaId}`
- **Authentification** : Requise via cookie `access_token` (JWT), comme pour le reste de l'API
- **Stockage** : Dossier `api/private/`
- **Base de données** : `isPrivate = true`

### Endpoint privé

```typescript
@PrivateMediaEndpoint('private/:mediaId')
async privateMedia(@Param('mediaId') mediaId: string, @Res() res: Response) {
  const media = await this.mediaService.findMedia(mediaId, {
    includeUserMedia: true,
  });
  
  if (!media) {
    throw new NotFoundException(this.i18n.t('media.private.not-found'));
  }

  // ⚠️ IMPORTANT : Vous devez implémenter vos propres vérifications de sécurité
  // Exemples de vérifications à ajouter :
  
  // 1. Vérifier que l'utilisateur est propriétaire du média
  // if (media.userMedia && media.userMedia.userId !== currentUser.id) {
  //   throw new UnauthorizedException('Accès refusé');
  // }
  
  // 2. Vérifier les permissions selon le contexte métier
  // if (media.productMedia && !hasProductAccess(media.productMedia.productId)) {
  //   throw new UnauthorizedException('Accès refusé');
  // }
  
  // 3. Vérifier les rôles utilisateur
  // if (media.isPrivate && !userHasRole('admin')) {
  //   throw new UnauthorizedException('Accès refusé');
  // }

  const outputPath = this.mediaService.getPathMedia(media);
  return res.sendFile(outputPath);
}
```

⚠️ **Sécurité critique** : Le contrôleur actuel ne contient **aucune vérification d'autorisation**. Vous devez absolument implémenter vos propres contrôles de sécurité avant de permettre l'accès aux médias privés.

**Comportement actuel** : Dans l'implémentation livrée, si aucune résolution d'accès n'est ajoutée dans le contrôleur (ex. vérification de propriété, rôle, etc.), l'endpoint retourne **401 Unauthorized** et n'envoie aucun fichier. Il faut ajouter la logique d'autorisation puis appeler `res.sendFile(outputPath)` pour servir le fichier.

### Sécurité

Le système implémente plusieurs mesures de sécurité :

1. **Path Traversal Protection** : Vérification que le chemin ne sort pas du dossier autorisé
2. **Authentification** : Cookie `access_token` (JWT) requis pour accéder aux médias privés
3. **Autorisation** : Vérifications personnalisées selon le contexte d'utilisation

```typescript
getPathMedia(media: MediaDTO) {
  const outputPath = join(process.cwd(), media.url);
  const baseDir = join(
    process.cwd(),
    media.isPrivate ? MEDIA_FOLDER.PRIVATE : MEDIA_FOLDER.PUBLIC,
  );
  
  if (!outputPath.startsWith(baseDir)) {
    throw new UnauthorizedException(
      this.i18n.t('media.get-path-media.error.path-traversal-detected'),
    );
  }
  return outputPath;
}
```

## Traitement des images

### Redimensionnement automatique

Le système utilise **Sharp** pour optimiser automatiquement les images :

```typescript
private async resizeImage(buffer: Buffer, options: ResizeOptions = {}): Promise<Buffer> {
  const { maxWidth = 1000, maxHeight = 1000, quality = 85 } = options;

  let sharpInstance = sharp(buffer);
  const metadata = await sharpInstance.metadata();

  // Redimensionner si nécessaire
  if (metadata.width > maxWidth || metadata.height > maxHeight) {
    sharpInstance = sharpInstance.resize(maxWidth, maxHeight, {
      fit: 'inside',
      withoutEnlargement: true,
    });
  }

  // Optimiser selon le format
  switch (metadata.format) {
    case 'jpeg':
      return await sharpInstance.jpeg({ quality }).toBuffer();
    case 'png':
      return await sharpInstance.png({ compressionLevel: 8 }).toBuffer();
    case 'webp':
      return await sharpInstance.webp({ quality }).toBuffer();
    default:
      return await sharpInstance.jpeg({ quality }).toBuffer();
  }
}
```

### Options de redimensionnement

| Option | Type | Défaut | Description |
|--------|------|--------|-------------|
| `maxWidth` | number | 1000 | Largeur maximale en pixels |
| `maxHeight` | number | 1000 | Hauteur maximale en pixels |
| `quality` | number | 85 | Qualité JPEG (1-100) |

### Contraintes appliquées à l'image de profil

Les paramètres utilisés pour `POST /user/update` sont les suivants :

| Contrainte | Valeur |
|---|---|
| Taille max avant upload | **5 MB** |
| Types MIME acceptés | `image/jpeg`, `image/png`, `image/webp` |
| Dimensions max après traitement | **256 × 256 px** |
| Qualité après recompression | **85 %** |

Ces contraintes sont encapsulées dans la factory `profilePicturePipe()` (`src/domains/media/pipes/profile-picture.pipe.ts`), qui retourne un `ParseFilePipe` préconfiguré avec les validateurs et la gestion d'erreurs i18n.

Les erreurs de validation sont localisées (i18n) et précisent le champ `profilePicture`, la taille actuelle et la taille maximum autorisée.

### URL des médias privés

Lorsqu'un profil est retourné (`GET /user/profile`), si l'image de profil est privée (`isPrivate = true`), l'URL stockée en base est remplacée à la volée par un chemin d'endpoint :

```typescript
// Dans user.service.ts
if (userMedia && userMedia.isPrivate) {
  userMedia.url = 'media/private/' + userMedia.id;
}
```

Le frontend doit alors appeler `GET /media/private/:mediaId` (route authentifiée) pour récupérer le fichier, au lieu d'accéder directement au chemin stocké.

## Gestion du cycle de vie

### Upload

1. **Validation** : Vérification du fichier et de ses métadonnées
2. **Traitement** : Redimensionnement et optimisation avec Sharp
3. **Stockage** : Sauvegarde dans le dossier approprié (public/private)
4. **Base de données** : Enregistrement des métadonnées

### Suppression

Le système implémente une suppression intelligente :

```typescript
async deleteMediaIfOrphan(mediaToDelete: MediaDTO & { id: string }) {
  const media = await this.findMediaUsing(mediaToDelete.id);
  const isOrphan = media && media.userMedia.length == 0;

  if (isOrphan) {
    await this.deleteMediaDatabase(mediaToDelete.id);
    try {
      await this.deleteMediaServer(mediaToDelete);
    } catch (error) {
      console.error('[deleteMediaIfOrphan] Error deleting file:', error);
    }
  }
}
```

### Nettoyage automatique

- **Suppression en cascade** : Lorsqu'un média est supprimé, les entrées `UserMedia` sont automatiquement supprimées
- **Suppression orpheline** : Les médias non utilisés sont automatiquement nettoyés
- **Gestion d'erreurs** : Les erreurs de suppression de fichiers n'interrompent pas le processus

## Upload de fichiers avec Multer et ParseFilePipe

### Configuration Multer

NestJS intègre automatiquement **Multer** pour la gestion des uploads de fichiers. Aucune configuration supplémentaire n'est nécessaire.

### Format des données : FormData

⚠️ **Important** : Les routes d'upload de fichiers dans une API REST doivent recevoir les données au format **`multipart/form-data`**.

#### Documentation Swagger

Pour que Swagger comprenne qu'il doit recevoir des données `multipart/form-data`, utilisez les décorateurs suivants :

```typescript
import { ApiConsumes, ApiBody } from '@nestjs/swagger';

export class TheMediaDTO = {
   @ApiProperty({
      type: 'string',
      format: 'binary',
      required: false,
      description: 'Photo de profil (fichier image)',
      })
  @IsOptional()
  profilePicture?: MediaDTO;
}

@Put('profile')
@ApiConsumes('multipart/form-data')
ApiBody({
      description: 'User profile to update + file',
      type: TheMediaDTO,
    }),
async updateProfile(
  // ...
) {
  // ...
}
```
### Décorateur @UploadedFile

Le décorateur `@UploadedFile` permet de récupérer un fichier uploadé dans un contrôleur.

**Pattern recommandé — factory function**

Lorsque la validation est complexe (validateurs multiples, `exceptionFactory` i18n), extraire la configuration dans une factory évite de surcharger le contrôleur :

```typescript
// src/domains/media/pipes/profile-picture.pipe.ts
export function profilePicturePipe(): ParseFilePipe {
  return new ParseFilePipe({
    validators: [...],
    fileIsRequired: false,
    exceptionFactory: (error) => { /* gestion i18n */ },
  });
}

// Dans le contrôleur
@UploadedFile(profilePicturePipe())
profilePicture: UploadedFileType,
```

**Pattern inline — cas simples**

Pour une validation basique sans `exceptionFactory`, l'inline reste acceptable :

```typescript
@UploadedFile(
  new ParseFilePipe({
    validators: [
      new FileTypeValidator({ fileType: /^image\/(jpeg|png|webp)$/ }),
      new MaxFileSizeValidator({ maxSize: 5 * 1024 * 1024 }),
    ],
    fileIsRequired: false,
  }),
)
file: UploadedFileType,
```

### Validateurs disponibles

#### FileTypeValidator

Valide le type MIME du fichier :

```typescript
new FileTypeValidator({
  fileType: /^image\/(jpeg|png|webp)$/, // Regex pour les types autorisés
})
```

Types MIME courants :
- **Images** : `image/jpeg`, `image/png`, `image/webp`, `image/gif`
- **Documents** : `application/pdf`, `application/msword`
- **Archives** : `application/zip`, `application/x-rar-compressed`

#### MaxFileSizeValidator

Limite la taille du fichier :

```typescript
new MaxFileSizeValidator({
  maxSize: 5 * 1024 * 1024, // 5MB en bytes
})
```

Tailles courantes :
- **1MB** : `1024 * 1024`
- **5MB** : `5 * 1024 * 1024`
- **10MB** : `10 * 1024 * 1024`

### Options ParseFilePipe

| Option | Type | Défaut | Description |
|--------|------|--------|-------------|
| `validators` | Array | `[]` | Liste des validateurs à appliquer |
| `fileIsRequired` | boolean | `true` | Si le fichier est obligatoire |
| `exceptionFactory` | Function | - | Factory pour les erreurs personnalisées |

### Gestion des erreurs

Le système de validation génère automatiquement des erreurs structurées :

```typescript
// Erreur de type de fichier
{
  "statusCode": 400,
  "message": "Validation failed",
  "error": "Bad Request",
  "violations": [
    {
      "field": "profilePicture",
      "message": "Type de fichier invalide. Types acceptés: jpeg, png, webp"
    }
  ]
}

// Erreur de taille
{
  "statusCode": 400,
  "message": "Validation failed", 
  "error": "Bad Request",
  "violations": [
    {
      "field": "profilePicture",
      "message": "Fichier trop volumineux. Taille actuelle: 8.5MB, Maximum: 5MB"
    }
  ]
}
```

## Tests et limitations

### Tests E2E : Limitations connues

⚠️ **Problème important** : Il existe des limitations connues avec les tests E2E pour l'upload de fichiers dans NestJS.

## Utilisation dans l'application

### Service MediaService

Le service principal expose les méthodes suivantes :

- `uploadMedia()` : Upload et traitement d'un fichier
- `createMedia()` : Création d'une entrée en base de données
- `findMedia()` : Récupération d'un média
- `deleteMediaIfOrphan()` : Suppression conditionnelle
- `getPathMedia()` : Récupération du chemin sécurisé

### Controller MediaController

Le contrôleur expose l'endpoint pour les médias privés :

- `GET /media/private/:mediaId` : Accès aux médias privés avec authentification

### Module MediaModule

```typescript
@Module({
  imports: [],
  controllers: [MediaController],
  providers: [MediaService],
  exports: [MediaService],
})
export class MediaModule {}
```

## Suppression de la photo de profil

Le flag `deleteProfilePicture` dans `UserUpdateBodyDTO` permet de supprimer la photo de profil sans uploader une nouvelle image.

| Champ | Type | Description |
|-------|------|-------------|
| `deleteProfilePicture` | Boolean | Si `true`, supprime la photo de profil existante |

### Utilisation

```typescript
// Via FormData (multipart/form-data)
formData.append('deleteProfilePicture', 'true');
```

> ⚠️ **Note FormData** : Les valeurs booléennes envoyées via FormData sont reçues comme des chaînes (`"true"` / `"false"`). Le DTO utilise un décorateur `@Transform` pour convertir : `value === 'true' || value === true`.

---

## Bonnes pratiques

### Sécurité

- ✅ **Toujours valider** les fichiers uploadés avec `ParseFilePipe`
- ✅ **Utiliser des chemins sécurisés** pour éviter les path traversals
- ✅ **Implémenter des vérifications d'autorisation** pour les médias privés
- ✅ **Limiter la taille** des fichiers uploadés avec `MaxFileSizeValidator`
- ✅ **Valider les types MIME** avec `FileTypeValidator`
- ✅ **Implémenter des contrôles d'accès** personnalisés dans les endpoints privés

### Performance

- ✅ **Redimensionner automatiquement** les images
- ✅ **Optimiser la qualité** selon le format
- ✅ **Nettoyer les médias orphelins** régulièrement
- ✅ **Utiliser des formats modernes** (WebP quand possible)

### Maintenance

- ✅ **Logger les erreurs** de suppression de fichiers
- ✅ **Gérer les transactions** pour la cohérence des données
- ✅ **Documenter les types de médias** dans l'enum
- ✅ **Tester les cas d'erreur** (fichiers corrompus, permissions, etc.)

### Upload et validation

- ✅ **Utiliser `fileIsRequired: false`** pour les fichiers optionnels
- ✅ **Implémenter `exceptionFactory`** pour des messages d'erreur personnalisés
- ✅ **Valider les types MIME** avec des regex strictes
- ✅ **Limiter les tailles** selon le contexte d'utilisation
- ✅ **Tester les uploads** avec différents formats et tailles

---

[← Retour au sommaire](./SUMMARY.md)
