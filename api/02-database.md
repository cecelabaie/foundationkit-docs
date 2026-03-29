# Database

[← Retour au sommaire](./SUMMARY.md)

Ce document présente la structure de la base de données PostgreSQL utilisée par l'API.

## Vue d'ensemble

L'application utilise **Prisma ORM** pour gérer la base de données PostgreSQL. Le schéma est défini dans `prisma/schema.prisma` et les migrations sont gérées automatiquement.

### Technologies

- **Base de données** : PostgreSQL
- **ORM** : Prisma
- **Client généré** : `generated/prisma`

### Configuration Prisma

```typescript
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
}
```

> **Note** : Dans ce projet, l’URL de connexion est définie dans le fichier **`prisma.config.ts`** à la racine du projet API. Exemple de contenu :

```typescript
// prisma.config.ts
import dotenv from 'dotenv';

dotenv.config({
  path: ['.env.production', '.env.local', '.env'],
});

import { defineConfig, env } from 'prisma/config';

export default defineConfig({
  schema: 'prisma/schema.prisma',
  migrations: { path: 'prisma/migrations' },
  datasource: { url: env('DATABASE_URL') },
});
```

> **Note** : La variable `DATABASE_URL` est lue depuis `.env.production` en priorité, puis `.env.local`, puis `.env`.

Le client Prisma est généré dans le dossier `generated/prisma` pour permettre une gestion centralisée et éviter les conflits avec `node_modules`.

## Énumérations

### Gender

Définit les genres disponibles pour les utilisateurs.

```typescript
enum Gender {
  male
  female
  other
}
```

### Role

Définit les rôles applicables aux utilisateurs. Un utilisateur peut avoir plusieurs rôles.

```typescript
enum Role {
  admin
  user
}
```

## Tables

### User (users)

Table principale contenant les informations des utilisateurs.

#### Champs

| Champ | Type | Description | Contraintes |
|-------|------|-------------|-------------|
| `id` | String (UUID) | Identifiant unique | PK, unique, auto-généré |
| `firstName` | String | Prénom | Obligatoire |
| `lastName` | String | Nom | Obligatoire |
| `username` | String | Nom d'utilisateur | Obligatoire, unique |
| `email` | String | Adresse email | Obligatoire, unique |
| `dateOfBirth` | DateTime | Date de naissance | Obligatoire, format Date |
| `gender` | Gender | Genre | Obligatoire, enum |
| `password` | String | Mot de passe hashé | Obligatoire |
| `role` | Role[] | Rôles de l'utilisateur | Array, défaut: [] |
| `isVerified` | Boolean | Compte vérifié | Défaut: false |
| `notificationEmail` | Boolean | Notifications par email | Défaut: false |
| `needToReconnect` | Boolean | Nécessite reconnexion | Défaut: false |
| `dateLastLogin` | DateTime? | Date de dernière connexion | Nullable |
| `signupToken` | String? | Hash SHA-256 du token de vérification email | Nullable, mis à null après vérification |
| `createdAt` | DateTime | Date de création | Auto, défaut: now() |
| `updatedAt` | DateTime | Date de mise à jour | Auto-update |

#### Relations

- `profilePicture` → **UserMedia** (1:1) : Image de profil
- `revokedTokens` → **RevokedToken[]** (1:N) : Tokens révoqués
- `refreshTokens` → **RefreshToken[]** (1:N) : Tokens de rafraîchissement
- `resetPasswordToken` → **ResetPasswordToken[]** (1:N) : Tokens de réinitialisation

#### Index

- `id` : Index unique
- `username` : Index unique
- `email` : Index unique

#### Utilisation

```typescript
// Créer un utilisateur
const user = await prisma.user.create({
  data: {
    firstName: 'John',
    lastName: 'Doe',
    username: 'johndoe',
    email: 'john@example.com',
    dateOfBirth: new Date('1990-01-01'),
    gender: 'male',
    password: hashedPassword,
    role: ['user'],
  },
});

// Récupérer un utilisateur avec ses relations
const user = await prisma.user.findUnique({
  where: { email: 'john@example.com' },
  include: {
    profilePicture: {
      include: { media: true },
    },
    refreshTokens: true,
  },
});
```

### Media (media)

Table stockant les métadonnées des fichiers uploadés (images de profil, documents, etc.).

#### Champs

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

#### Relations

- `userMedia` → **UserMedia[]** (1:N) : Associations avec utilisateurs

#### Utilisation

Les médias peuvent être **publics** ou **privés** :
- **Public** : Stockés dans `/public/`, accessibles directement via URL
- **Privé** : Stockés dans `/private/`, nécessitent authentification

```typescript
// Créer un média
const media = await prisma.media.create({
  data: {
    name: 'profile-picture.jpg',
    url: '/users/profilePicture/uuid.jpg',
    mimeType: 'image/jpeg',
    size: 245678,
    isPrivate: false,
  },
});
```

### UserMedia (user_media)

Table de liaison entre utilisateurs et médias pour les images de profil.

#### Champs

| Champ | Type | Description | Contraintes |
|-------|------|-------------|-------------|
| `id` | String (UUID) | Identifiant unique | PK, auto-généré |
| `userId` | String | ID de l'utilisateur | FK → User, unique |
| `mediaId` | String | ID du média | FK → Media |
| `createdAt` | DateTime | Date de création | Auto, défaut: now() |
| `updatedAt` | DateTime | Date de mise à jour | Auto-update |

#### Relations

- `user` → **User** : Utilisateur propriétaire
- `media` → **Media** : Média associé (CASCADE on delete)

#### Index

- `userId` : Index unique (un seul média de profil par utilisateur)

#### Suppression en cascade

Lorsqu'un média est supprimé, l'entrée `UserMedia` est automatiquement supprimée grâce à `onDelete: Cascade`.

```typescript
// Associer un média à un utilisateur
const userMedia = await prisma.userMedia.create({
  data: {
    userId: user.id,
    mediaId: media.id,
  },
});

// Mise à jour de l'image de profil
const updatedUserMedia = await prisma.userMedia.update({
  where: { userId: user.id },
  data: { mediaId: newMedia.id },
});
```

### RefreshToken (refresh_tokens)

Table stockant les tokens de rafraîchissement (token aléatoire, stocké hashé en SHA-256) pour maintenir les sessions utilisateur.

#### Champs

| Champ | Type | Description | Contraintes |
|-------|------|-------------|-------------|
| `id` | String (UUID) | Identifiant unique | PK, auto-généré |
| `hashedToken` | String | Token hashé | Obligatoire, unique |
| `userId` | String | ID de l'utilisateur | FK → User |
| `createdAt` | DateTime | Date de création | Auto, défaut: now() |
| `expiresAt` | DateTime | Date d'expiration | Obligatoire |
| `revokedAt` | DateTime? | Date de révocation | Nullable (null = valide) |
| `ip` | String? | Adresse IP de création | Nullable |
| `userAgent` | String? | Informations navigateur/OS | Nullable |

#### Relations

- `user` → **User** : Utilisateur propriétaire

#### Index

- `hashedToken` : Index unique

#### Cycle de vie

1. **Création** : Lors du login ou du refresh
2. **Vérification** : À chaque demande de refresh
3. **Révocation** : Lors du logout (revokedAt = now())
4. **Expiration** : Automatique après 24h

```typescript
// Créer un refresh token
const refreshToken = await prisma.refreshToken.create({
  data: {
    hashedToken: hash(token),
    userId: user.id,
    expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24h
    ip: req.ip,
    userAgent: req.headers['user-agent'],
  },
});

// Révoquer un token
await prisma.refreshToken.update({
  where: { id: tokenId },
  data: { revokedAt: new Date() },
});

// Récupérer les tokens actifs
const activeTokens = await prisma.refreshToken.findMany({
  where: {
    userId: user.id,
    revokedAt: null,
    expiresAt: { gt: new Date() },
  },
});
```

### RevokedToken (revoked_tokens)

Table stockant les access tokens révoqués avant leur expiration naturelle.

#### Champs

| Champ | Type | Description | Contraintes |
|-------|------|-------------|-------------|
| `id` | String (UUID) | Identifiant unique | PK, auto-généré |
| `token` | String | Hash SHA-256 du JWT révoqué | Obligatoire, unique, Text |
| `userId` | String | ID de l'utilisateur | FK → User |
| `revokedAt` | DateTime | Date de révocation | Obligatoire |
| `expirationDate` | DateTime | Date d'expiration originale | Obligatoire |

#### Relations

- `user` → **User** : Utilisateur propriétaire

#### Index

- `token` : Index unique

#### Utilisation

Cette table permet d'invalider immédiatement un access token (notamment lors du logout) sans attendre son expiration naturelle de 15 minutes.

```typescript
// Révoquer un access token (le token est hashé avant stockage)
const hashedToken = hashToken(accessToken); // SHA-256
await prisma.revokedToken.create({
  data: {
    token: hashedToken,
    userId: user.id,
    revokedAt: new Date(),
    expirationDate: new Date(Date.now() + 15 * 60 * 1000), // 15min
  },
});

// Vérifier si un token est révoqué (hasher le token reçu puis rechercher)
const revokedToken = await prisma.revokedToken.findUnique({
  where: { token: hashToken(accessToken) },
});

if (revokedToken) {
  throw new UnauthorizedException('Token révoqué');
}
```

#### Nettoyage

Les tokens expirés peuvent être nettoyés périodiquement pour optimiser la base :

```typescript
// Supprimer les tokens expirés
await prisma.revokedToken.deleteMany({
  where: {
    expirationDate: { lt: new Date() },
  },
});
```

### ResetPasswordToken (reset_password_tokens)

Table gérant les tokens de réinitialisation de mot de passe.

#### Champs

| Champ | Type | Description | Contraintes |
|-------|------|-------------|-------------|
| `id` | String (UUID) | Identifiant unique | PK, auto-généré |
| `token` | String | Hash SHA-256 du token de réinitialisation | Obligatoire, unique |
| `userId` | String | ID de l'utilisateur | FK → User |
| `expiresAt` | DateTime | Date d'expiration | Obligatoire |
| `used` | Boolean | Token déjà utilisé | Défaut: false |
| `revoked` | Boolean | Token révoqué | Défaut: false |
| `ip` | String? | Adresse IP de la demande | Nullable |
| `userAgent` | String? | Informations navigateur/OS | Nullable |
| `createdAt` | DateTime | Date de création | Auto, défaut: now() |
| `updatedAt` | DateTime | Date de mise à jour | Auto-update |

#### Relations

- `user` → **User** : Utilisateur demandant la réinitialisation

#### Index

- `token` : Index unique

#### Flux de réinitialisation

1. **Demande** : Création du token (expiration 15min)
2. **Validation** : Vérification du token
3. **Utilisation** : Changement du mot de passe (used = true)
4. **Révocation** : Annulation manuelle possible

```typescript
// Créer un token de réinitialisation (le token brut est envoyé par email, le hash est stocké en base)
const { token: rawToken, hashedToken } = createRandomToken(); // ou équivalent
await prisma.resetPasswordToken.create({
  data: {
    token: hashedToken,
    userId: user.id,
    expiresAt: new Date(Date.now() + 15 * 60 * 1000), // 15min
    ip: req.ip,
    userAgent: req.headers['user-agent'],
  },
});
// Envoyer rawToken par email (lien de réinitialisation)

// Vérifier et utiliser le token (hasher le token reçu depuis l'URL puis rechercher)
const token = await prisma.resetPasswordToken.findFirst({
  where: {
    token: hashToken(tokenFromEmail),
    used: false,
    revoked: false,
    expiresAt: { gt: new Date() },
  },
});

if (!token) {
  throw new BadRequestException('Token invalide ou expiré');
}

// Marquer comme utilisé
await prisma.resetPasswordToken.update({
  where: { id: token.id },
  data: { used: true },
});
```

### LoggerMail (logger_mail)

Table de logs pour tracer tous les emails envoyés par l'application.

#### Champs

| Champ | Type | Description | Contraintes |
|-------|------|-------------|-------------|
| `id` | String (UUID) | Identifiant unique | PK, auto-généré |
| `to` | String | Destinataire | Obligatoire |
| `subject` | String | Sujet de l'email | Obligatoire |
| `html` | String | Contenu HTML | Obligatoire |
| `accepted` | String[] | Emails acceptés par le serveur | Array |
| `rejected` | String[] | Emails rejetés par le serveur | Array |
| `response` | String? | Réponse du serveur SMTP | Nullable |
| `from` | String | Expéditeur | Obligatoire |
| `createdAt` | DateTime | Date d'envoi | Auto, défaut: now() |
| `updatedAt` | DateTime | Date de mise à jour | Auto-update |

#### Utilisation

Cette table permet de :
- Tracer tous les emails envoyés
- Déboguer les problèmes d'envoi
- Respecter les obligations légales de traçabilité
- Éviter les doublons d'envoi

```typescript
// Logger un email envoyé
await prisma.loggerMail.create({
  data: {
    to: user.email,
    subject: 'Réinitialisation de mot de passe',
    html: emailTemplate,
    accepted: mailResponse.accepted,
    rejected: mailResponse.rejected,
    response: mailResponse.response,
    from: 'noreply@myapp.com',
  },
});

// Récupérer l'historique des emails d'un utilisateur
const emailHistory = await prisma.loggerMail.findMany({
  where: { to: user.email },
  orderBy: { createdAt: 'desc' },
  take: 10,
});
```

## Commandes Prisma

### Migration

```bash
# Créer une nouvelle migration
npx prisma migrate dev --name nom_de_la_migration

# Appliquer les migrations en production
npx prisma migrate deploy

# Réinitialiser la base de données (⚠️ supprime toutes les données)
npx prisma migrate reset
```

### Génération

```bash
# Générer le client Prisma après modification du schéma
npx prisma generate
```

### Studio

```bash
# Ouvrir l'interface graphique Prisma Studio
npx prisma studio
```

Prisma Studio permet de visualiser et manipuler les données via une interface web sur `http://localhost:5555`.

### Seed (Données de test)

```bash
# Générer des données de test avec Faker
npm run seed
```

Le script de seed est situé dans `src/faker/service/scripts/seed.ts`.

## Sécurité

### Mots de passe

Les mots de passe sont **toujours hashés** avec bcrypt avant stockage.

### Injection SQL

Prisma protège automatiquement contre les injections SQL grâce à son système de requêtes paramétrées.

---

[← Retour au sommaire](./SUMMARY.md)
