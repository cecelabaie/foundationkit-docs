# Swagger

[← Retour au sommaire](./SUMMARY.md)

Ce document présente l'utilisation de Swagger/OpenAPI dans le projet API. Swagger est utilisé pour générer automatiquement une documentation interactive de l'API, permettant aux développeurs de comprendre et tester les endpoints facilement.

## Vue d'ensemble

Le projet utilise **@nestjs/swagger** pour générer automatiquement une documentation OpenAPI 3.0 complète. La documentation est accessible via une interface web interactive et peut être exportée en JSON/YAML.

### Avantages

- **Documentation automatique** : Génération automatique à partir du code
- **Interface interactive** : Test des endpoints directement depuis le navigateur
- **Validation des schémas** : Vérification des types de données
- **Export** : Possibilité d'exporter la spécification OpenAPI
- **Authentification intégrée** : Support JWT Bearer Token

## Configuration principale

### Setup dans main.ts

La configuration Swagger est définie dans `src/main.ts` :

```typescript
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

const apiUrl = configService.getOrThrow<string>('API_URL');
const apiPort = configService.getOrThrow<string>('API_PORT');

const config = new DocumentBuilder()
  .setTitle('API Authentication')
  .setDescription("API avec système d'authentification JWT")
  .setVersion('1.0')
  .addTag('auth', "Endpoints d'authentification")
  .addBearerAuth(
    {
      type: 'http',
      scheme: 'bearer',
      bearerFormat: 'JWT',
    },
    'JWT-auth',
  )
  .addServer(`${apiUrl}:${apiPort}`)
  .build();

const documentFactory = () => SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api', app, documentFactory);
```

### Configuration du plugin NestJS

Le projet utilise le plugin Swagger de NestJS configuré dans `nest-cli.json` :

```json
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "@nestjs/swagger/plugin",
        "options": {
          "classValidatorShim": true,
          "introspectComments": true,
          "skipAutoHttpCode": true
        }
      }
    ]
  }
}
```

**Options du plugin** :
- **classValidatorShim** : Intègre automatiquement les décorateurs `class-validator` dans la documentation Swagger. Cette option génère automatiquement des schémas basés sur les validations définies dans les DTOs (comme `@IsEmail`, `@MinLength`, etc.), assurant que la documentation reflète fidèlement les contraintes de validation de l'API.
- **introspectComments** : Analyse les commentaires JSDoc présents dans le code pour enrichir la documentation générée. Permet d'ajouter des descriptions détaillées aux endpoints, paramètres et réponses directement depuis les commentaires TypeScript.
- **skipAutoHttpCode** : Empêche Swagger d'attribuer automatiquement des codes HTTP aux réponses. Cette option est utilisée car le projet définit des réponses personnalisées avec des codes HTTP spécifiques via les DTOs de réponse standardisés (voir [Réponses](./06-responses.md)), garantissant que la documentation reflète précisément le comportement de l'API.

## Accès à la documentation

### Interface web

La documentation Swagger est accessible via l'interface web :

- **URL locale** : `{API_URL}:{API_PORT}/api`
- **URL de production** : `{API_URL}:{API_PORT}/api`

### Export de la spécification

La spécification OpenAPI peut être exportée :

- **JSON** : `{API_URL}:{API_PORT}/api-json`
- **YAML** : `{API_URL}:{API_PORT}/api-yaml`

## Décorateurs Swagger utilisés

### 1. **@ApiTags** - Groupement des endpoints

Groupe les endpoints par domaine dans l'interface Swagger :

```typescript
@ApiTags('user')
@Controller('user')
export class UserController {
  // ...
}

@ApiTags('auth')
@Controller('auth')
export class AuthController {
  // ...
}
```

### 2. **@ApiOperation** - Description des endpoints

Décrit le fonctionnement d'un endpoint :

```typescript
@ApiOperation({
  summary: 'User connection',
  description: 'Authenticate a user with his email and password. Returns a valid JWT access token (15 minutes) and a refresh token in HTTP-only cookies.',
})
async login(@Body() loginDto: AuthLoginBodyDTO) {
  // ...
}
```

### 3. **@ApiProperty** - Documentation des propriétés

**Rôle** : `@ApiProperty` est utilisé **uniquement pour la documentation Swagger**. Il ne valide pas les données - c'est le rôle des décorateurs `class-validator` comme `@IsEmail`, `@IsString`, etc.

**Fonction** : Génère la documentation OpenAPI pour les propriétés des DTOs, incluant les descriptions, exemples, types et contraintes.

```typescript
export class UserRegisterBodyDTO {
  @ApiProperty({
    description: "Prénom de l'utilisateur",
    example: 'John',
    minLength: 4,
  })
  @IsString({ message: 'user.register.validation.first-name.is-string' })
  @MinLength(4, { message: i18nValidationMessage('user.register.validation.first-name.min-length') })
  firstName: string;

  @ApiProperty({
    description: "Adresse email de l'utilisateur",
    example: 'user@example.com',
    format: 'email',
  })
  @IsEmail({}, { message: 'user.register.validation.email.is-email' })
  email: string;
}
```

**Important** : La validation réelle est assurée par `class-validator`, tandis que `@ApiProperty` ne fait que documenter pour Swagger.

### 4. **@ApiBody** - Documentation du corps de requête

Décrit le format du corps de la requête :

```typescript
@ApiBody({
  description: 'User information',
  type: UserRegisterBodyDTO,
})
async register(@Body() userRegisterDto: UserRegisterBodyDTO) {
  // ...
}
```

### 5. **@ApiResponse** - Documentation des réponses

Documente les différentes réponses possibles :

```typescript
@ApiResponse({
  status: 201,
  description: 'User created successfully',
  type: UserRegisterResponseDTO,
})
@ApiResponse({
  status: 400,
  description: 'Validation failed',
  type: ValidationExceptionResponseDTO,
})
@ApiResponse({
  status: 500,
  description: 'Internal server error',
  type: InternalServerErrorExceptionResponseDTO,
})
async register(@Body() userRegisterDto: UserRegisterBodyDTO) {
  // ...
}
```

### 6. **@ApiBearerAuth** - Authentification JWT

Indique qu'un endpoint nécessite une authentification :

```typescript
@ApiBearerAuth()
@Get('profile')
async getProfile(@Req() req: RequestWithCookies) {
  // ...
}
```

## Décorateurs personnalisés

### Pattern des décorateurs d'endpoints

Le projet utilise des décorateurs personnalisés qui combinent plusieurs décorateurs Swagger :

```typescript
export function RegisterEndpoint(path: string) {
  return applyDecorators(
    Post(path),
    Public(),
    ApiOperation({
      summary: 'Register a user',
      description: 'Register a user',
    }),
    ApiBody({
      description: 'User information',
      type: UserRegisterBodyDTO,
    }),
    ApiResponse({
      status: 201,
      description: 'User created',
      type: UserRegisterResponseDTO,
    }),
    ApiResponse({
      status: 400,
      description: 'Validation failed',
      type: ValidationExceptionResponseDTO,
    }),
    ApiResponse({
      status: 500,
      description: 'Failed to create user',
      type: InternalServerErrorExceptionResponseDTO,
    }),
  );
}
```

### Avantages des décorateurs personnalisés

- **Réutilisabilité** : Même configuration pour plusieurs endpoints
- **Consistance** : Documentation uniforme
- **Maintenabilité** : Modification centralisée
- **Lisibilité** : Code plus propre dans les contrôleurs

## DTOs de réponse standardisés

### Structure commune

**Les DTOs de réponse de succès** héritent de `SuccessResponseDTO` :

```typescript
export class SuccessResponseDTO {
  @ApiProperty()
  statusCode: number;

  @ApiProperty()
  message: string;

  @ApiProperty()
  data?: unknown;
}

export class UserRegisterResponseDTO extends SuccessResponseDTO {}
```

### DTOs d'erreur

**Les DTOs de réponse d'erreur** héritent de `InternalServerErrorExceptionResponseDTO` :

```typescript
export class ValidationExceptionResponseDTO {
  @ApiProperty()
  statusCode: number;

  @ApiProperty()
  message: string;

  @ApiProperty()
  error: string;

  @ApiProperty({ type: [ValidationViolation] })
  violations: ValidationViolation[];
}

export class UnauthorizedResponseDTO extends InternalServerErrorExceptionResponseDTO {}
export class NotFoundResponseDTO extends InternalServerErrorExceptionResponseDTO {}
export class BadRequestResponseDTO extends InternalServerErrorExceptionResponseDTO {}
export class TooManyRequestResponseDTO extends InternalServerErrorExceptionResponseDTO {}
```

Pour plus de détails sur la structure des réponses, voir [Réponses](./06-responses.md).

## Authentification dans Swagger

### Configuration JWT

L'authentification JWT est configurée globalement dans `main.ts` :

```typescript
.addBearerAuth(
  {
    type: 'http',
    scheme: 'bearer',
    bearerFormat: 'JWT',
  },
  'JWT-auth',
)
```

### Fonctionnement de l'authentification

**Important** : L'authentification dans ce projet utilise des **cookies** et non des headers Bearer. Le décorateur `@ApiBearerAuth` est utilisé uniquement pour la **documentation Swagger** et indique que l'endpoint nécessite une authentification.

**Comment ça fonctionne** :
1. **Authentification réelle** : Gérée par `AuthGuard` qui vérifie le cookie `access_token`
2. **Documentation Swagger** : `@ApiBearerAuth` indique visuellement que l'endpoint est protégé
3. **Test dans Swagger** : Vous devez fournir un token JWT valide pour tester les endpoints protégés

Pour plus de détails sur l'authentification et la session, voir [Session](./07-session.md).


### Utilisation dans les endpoints

```typescript
@ApiBearerAuth()  // Documentation uniquement
@Get('profile')
async getProfile(@Req() req: RequestWithCookies) {
  // L'authentification réelle est gérée par AuthGuard via les cookies
}
```

### Test des requêtes dans Swagger

Dans l'interface Swagger :
1. **Endpoints publics** : Testables directement sans authentification
2. **Endpoints protégés** : 
   - Cliquer sur le bouton "Authorize" 
   - Saisir un token JWT valide dans le format : `Bearer <token>`
   - Tester les endpoints protégés

**Note** : Le token doit être obtenu via l'endpoint de login (`/auth/login`) qui retourne un access token valide.

## Gestion des fichiers

### Upload de fichiers

Pour les endpoints d'upload, utiliser `@ApiConsumes` et `@ApiBody` :

```typescript
@ApiConsumes('multipart/form-data')
@ApiBody({
  description: 'Profile picture upload',
  schema: {
    type: 'object',
    properties: {
      profilePicture: {
        type: 'string',
        format: 'binary',
      },
    },
  },
})
async updateProfile(@UploadedFile() file: Express.Multer.File) {
  // ...
}
```

Pour plus de détails sur la gestion des médias et l'upload de fichiers, voir [Media](./09-media.md).

## Internationalisation et Swagger

### Documentation multilingue

La documentation Swagger est principalement en français, mais les messages d'erreur sont traduits selon la langue de l'utilisateur.

## Bonnes pratiques

### 1. **Documentation complète**

- Toujours documenter les propriétés des DTOs
- Fournir des exemples concrets
- Décrire les codes de réponse possibles

### 2. **Cohérence des réponses**

- Utiliser les DTOs de réponse standardisés
- Documenter tous les cas d'erreur
- Maintenir la structure des réponses

### 3. **Sécurité**

- Marquer les endpoints sensibles avec `@ApiBearerAuth`
- Documenter les limitations de rate limiting
- Indiquer les permissions requises

### 4. **Exemples réalistes**

- Utiliser des exemples de données réalistes
- Varier les exemples selon le contexte
- Tester les exemples fournis

### 5. **Organisation**

- Grouper les endpoints par domaine avec `@ApiTags`
- Utiliser des décorateurs personnalisés pour la réutilisabilité
- Maintenir la cohérence dans la documentation

## Commandes utiles

### Développement

```bash
# Démarrer l'API avec Swagger
npm run start:dev

# Accéder à Swagger
{API_URL}:{API_PORT}/api
```

### Export de la documentation

```bash
# Exporter en JSON
curl {API_URL}:{API_PORT}/api-json > api-docs.json

# Exporter en YAML
curl {API_URL}:{API_PORT}/api-yaml > api-docs.yaml
```

## Intégration avec le frontend

### Génération automatique

Le projet utilise `orval` pour générer automatiquement le client TypeScript à partir de la spécification OpenAPI :

```typescript
// orval.config.ts
export default {
  api: {
    input: {
      target: '{API_URL}:{API_PORT}/api-json',
    },
    output: {
      target: './src/api/generated',
      client: 'react-query',
    },
  },
};
```

### Importance pour la génération

**L'utilisation complète du système Swagger est primordiale** pour le bon fonctionnement de la génération côté frontend avec Orval :

1. **Décorateurs complets** : Tous les `@ApiProperty`, `@ApiResponse`, `@ApiOperation` sont nécessaires
2. **DTOs bien documentés** : Chaque propriété doit être documentée pour générer des types corrects
3. **Réponses standardisées** : Les DTOs de réponse permettent de générer des interfaces TypeScript cohérentes
4. **Authentification documentée** : Les décorateurs `@ApiBearerAuth` permettent de générer des clients avec gestion d'auth

### Avantages

- **Synchronisation** : Le client frontend est toujours à jour avec l'API
- **Type safety** : Types TypeScript générés automatiquement et corrects
- **Maintenance** : Pas de code client à maintenir manuellement
- **Cohérence** : Structure des données identique entre front et back

Pour plus de détails sur la génération automatique, voir [Génération](../front/09-generation.md).

Cette documentation Swagger complète permet aux développeurs de comprendre rapidement l'API, de la tester facilement et de générer automatiquement les clients frontend.

---

[← Retour au sommaire](./SUMMARY.md)
