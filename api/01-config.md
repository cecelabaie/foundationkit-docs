# Configuration

[← Retour au sommaire](./SUMMARY.md)

Ce document présente les différentes configurations du projet API.

## Variables d'environnement

L'API utilise plusieurs variables d'environnement pour sa configuration. Ces variables doivent être définies dans un fichier `.env.production`, `.env.local` ou `.env` à la racine du projet API. Si plusieurs fichiers existent, `.env.production` a la priorité la plus haute, puis `.env.local`, puis `.env`.

### Description des variables

| Variable | Description | Exemple | Obligatoire |
|----------|-------------|---------|-------------|
| `NODE_ENV` | Environnement d'exécution | `development`, `production`, `test` | Oui |
| `API_PORT` | Port d'écoute de l'API | `3001` | Oui |
| `API_URL` | URL de base de l'API (sans le port) | `http://localhost` | Oui |
| `APP_URL` | URL de l'application frontend | `http://localhost:3000` | Oui |
| `APP_NAME` | Nom de l'application (utilisé dans les emails) | `My project` | Oui |
| `DATABASE_URL` | URL de connexion PostgreSQL | `postgresql://user:password@localhost:5432/db` | Oui |
| `JWT_SECRET` | Secret pour la signature des JWT | Chaîne aléatoire sécurisée | Oui |
| `SMTP_HOST` | Serveur SMTP pour l'envoi d'emails | `smtp.gmail.com` | Oui |
| `SMTP_PORT` | Port du serveur SMTP | `587` | Oui |
| `SMTP_USER` | Nom d'utilisateur SMTP | `user@gmail.com` | Oui |
| `SMTP_PASS` | Mot de passe SMTP | Mot de passe ou app password | Oui |
| `SMTP_FROM_NAME` | Nom de l'expéditeur | `My Application` | Oui |
| `SMTP_FROM_ADDRESS` | Email de l'expéditeur | `noreply@myapp.com` | Oui |
| `FALLBACK_LANGUAGE` | Langue par défaut de l'application | `fr` | Oui |

### Accès aux valeurs

Dans n'importe quel service, contrôleur ou module :

```typescript
import { ConfigService } from '@nestjs/config';

constructor(private configService: ConfigService) {}

const apiUrl = this.configService.get<string>('API_URL');
// ou avec exception si la variable n'existe pas
const apiPort = this.configService.getOrThrow<string>('API_PORT');
```

## configuration.ts

Le fichier `src/config/configuration.ts` centralise l'accès aux variables d'environnement dans l'application.

### Structure

```typescript
export default () => ({
  API_URL: process.env.API_URL,
  APP_URL: process.env.APP_URL,
  APP_NAME: process.env.APP_NAME,
  NODE_ENV: process.env.NODE_ENV,
  API_PORT: process.env.API_PORT,
  SMTP_HOST: process.env.SMTP_HOST,
  SMTP_PORT: process.env.SMTP_PORT,
  SMTP_USER: process.env.SMTP_USER,
  SMTP_PASS: process.env.SMTP_PASS,
  SMTP_FROM_NAME: process.env.SMTP_FROM_NAME,
  SMTP_FROM_ADDRESS: process.env.SMTP_FROM_ADDRESS,
  JWT_SECRET: process.env.JWT_SECRET,
  DATABASE_URL: process.env.DATABASE_URL,
  FALLBACK_LANGUAGE: process.env.FALLBACK_LANGUAGE,
});
```

### Utilisation dans l'application

Le fichier est chargé automatiquement dans le module principal via `ConfigModule` :

```typescript
ConfigModule.forRoot({
  isGlobal: true,      // Configuration accessible dans tous les modules
  cache: true,         // Mise en cache des variables pour meilleures performances
  load: [configuration], // Chargement du fichier de configuration
  validationOptions: {
    allowUnknown: false, // Rejette les variables non définies
    abortEarly: true,    // Arrête à la première erreur
  },
})
```

## Rate Limiting (Throttler)

L'API implémente un système de limitation de requêtes (rate limiting) pour protéger contre les abus et les attaques par déni de service.

### Configuration globale

Le throttler est configuré dans `AppModule` avec des valeurs différentes selon l'environnement :

```typescript
ThrottlerModule.forRootAsync({
  useFactory: (configService: ConfigService) => {
    const nodeEnv = configService.getOrThrow<string>('NODE_ENV');
    const throttleConstants = createThrottleConstants(nodeEnv);

    return {
      throttlers: [{
        ttl: throttleConstants.TTL_GENERAL,
        limit: throttleConstants.LIMIT_GENERAL,
      }],
      errorMessage: (context, details) => {
        const ttlMinutes = Math.ceil(details.ttl / 1000 / 60);
        const i18n = I18nContext.current();
        
        if (i18n) {
          return i18n.t('common.throttler.too-many-requests', {
            args: { ttl: ttlMinutes },
          });
        }
        return `Too many requests, please try again in ${ttlMinutes} minutes`;
      },
    };
  },
  inject: [ConfigService],
})
```

### Limites par défaut

Les limites sont définies dans `src/domains/common/constants/constants.ts` :

| Endpoint | TTL (prod) | TTL (test) | Limite (prod) | Limite (test) |
|----------|------------|------------|---------------|---------------|
| Général | 1 minute | 1 seconde | 25 requêtes | 1000 requêtes |
| Auth Login | 1 minute | 1 seconde | 25 requêtes | 1000 requêtes |
| Auth Logout | 1 minute | 1 seconde | 25 requêtes | 1000 requêtes |
| Reset Password | 1 minute | 1 seconde | 25 requêtes | 1000 requêtes |
| Reset Password Validate | 1 minute | 1 seconde | 25 requêtes | 1000 requêtes |

### Configuration personnalisée par endpoint

Pour appliquer une limite spécifique à un endpoint :

```typescript
@Throttle({ default: { ttl: THROTTLE_TTL.TTL_AUTH_LOGIN, limit: THROTTLE_TTL.LIMIT_AUTH_LOGIN } })
@Post('login')
async login() {
  // ...
}
```

Pour désactiver le throttler sur un endpoint spécifique :

```typescript
@SkipThrottle()
@Get('public-data')
async getPublicData() {
  // ...
}
```

### Comportement

- Le compteur est par **adresse IP**
- Les requêtes au-delà de la limite reçoivent une erreur **429 Too Many Requests**
- Le message d'erreur est traduit selon la langue de l'utilisateur

## CORS (Cross-Origin Resource Sharing)

La configuration CORS permet au frontend d'accéder à l'API depuis un domaine différent.

### Configuration

Dans `src/main.ts` :

```typescript
const apiUrl = configService.getOrThrow<string>('API_URL');
const appUrl = configService.getOrThrow<string>('APP_URL');

app.enableCors({
  origin: [appUrl],    // Origine autorisée (URL du frontend)
  credentials: true,   // Autorise l'envoi de cookies
});
```

### Points clés

- **origin** : Liste des URLs autorisées à accéder à l'API
  - En production : uniquement l'URL du frontend
  - En développement : possibilité d'utiliser `'*'` pour autoriser toutes les origines
- **credentials: true** : Essentiel pour permettre l'envoi des cookies (refresh tokens, access tokens)
- Les cookies sont utilisés pour l'authentification JWT

### Sécurité

⚠️ **Important** :
- Ne jamais utiliser `origin: '*'` avec `credentials: true` en production
- Toujours spécifier explicitement les domaines autorisés
- Les cookies HTTP-only protègent contre les attaques XSS

## nest-cli.json

Configuration du CLI NestJS et du processus de build.

### Options principales

- **sourceRoot** : Répertoire source (`src`)
- **deleteOutDir** : Supprime le dossier de sortie avant chaque build
- **plugins** : 
  - `@nestjs/swagger/plugin` : Génération automatique de la documentation Swagger
    - `classValidatorShim` : Ajoute automatiquement les décorateurs de validation à Swagger
    - `introspectComments` : Utilise les commentaires TypeScript pour la documentation
    - `skipAutoHttpCode` : Ne génère pas automatiquement les codes HTTP
- **assets** : Fichiers à copier lors du build
  - `i18n/**/*` : Copie tous les fichiers de traduction
  - `watchAssets: true` : Recharge les fichiers en mode watch

### Génération Swagger

Le plugin Swagger permet de générer automatiquement la documentation API sans avoir à ajouter manuellement les décorateurs `@ApiProperty()` sur chaque propriété.

## tsconfig.json

Configuration TypeScript pour le projet NestJS.

### Particularités NestJS

- **emitDecoratorMetadata** et **experimentalDecorators** : Obligatoires pour le système de décorateurs de NestJS
- **noUnusedParameters: false** : Nécessaire car l'injection de dépendances peut créer des paramètres de constructeur non utilisés directement
- **commonjs** : NestJS fonctionne avec CommonJS par défaut

### tsconfig.build.json

Fichier spécifique pour le build de production :

```json
{
  "extends": "./tsconfig.json",
  "exclude": ["node_modules", "test", "dist", "**/*spec.ts"]
}
```

Exclut les fichiers de test et le dossier dist du build final.

## package.json

### Commandes principales

| Commande | Description |
|----------|-------------|
| `npm run build` | Compile l'application TypeScript vers JavaScript |
| `npm run format` | Formate le code avec Prettier |
| `npm run start` | Lance l'API en mode normal |
| `npm run start:dev` | Lance l'API en mode développement avec hot-reload |
| `npm run start:debug` | Lance l'API en mode debug |
| `npm run start:prod` | Lance l'API compilée en mode production |
| `npm run seed` | Génère des données de test avec Faker |
| `npm run lint` | Vérifie et corrige le code avec ESLint |
| `npm run test:win` | Exécute les tests (Windows) avec `--experimental-vm-modules` |
| `npm run test:linux` | Exécute les tests (Linux/Mac) avec `--experimental-vm-modules` |
| `npm run test:watch` | Exécute les tests en mode watch |
| `npm run test:cov` | Génère un rapport de couverture de tests |
| `npm run test:e2e` | Exécute les tests end-to-end via `test/jest-e2e.json` |

### Dépendances principales

#### Framework et Core

- **@nestjs/common, @nestjs/core** : Framework NestJS
- **@nestjs/platform-express** : Adaptateur Express pour NestJS
- **reflect-metadata** : Métadonnées pour les décorateurs (requis par NestJS)
- **rxjs** : Programmation réactive (utilisé par NestJS)

#### Configuration et Validation

- **@nestjs/config** : Gestion de la configuration
- **class-validator, class-transformer** : Validation et transformation des DTOs
- **cookie-parser** : Lecture des cookies

#### Base de données

- **@prisma/client** : Client Prisma pour PostgreSQL
- **@prisma/adapter-pg** : Adaptateur PostgreSQL natif pour Prisma
- **pg** : Driver PostgreSQL Node.js
- **prisma** (dev) : CLI Prisma pour migrations et génération

#### Authentification et Sécurité

- **@nestjs/jwt** : Gestion des JWT
- **bcrypt** : Hashage des mots de passe
- **crypto-js** : Opérations cryptographiques
- **@nestjs/throttler** : Rate limiting

#### Documentation

- **@nestjs/swagger** : Génération automatique de documentation OpenAPI/Swagger

#### Internationalisation

- **nestjs-i18n** : Système de traduction multi-langues

#### Logging

- **nest-winston** : Intégration NestJS de Winston
- **winston** : Système de logs structurés

#### Email

- **nodemailer** : Envoi d'emails via SMTP

#### Médias

- **sharp** : Traitement d'images (redimensionnement, compression)
- **file-type** : Détection de type MIME

#### Tests

- **jest** : Framework de tests
- **@nestjs/testing** : Utilitaires de test pour NestJS
- **supertest** : Tests HTTP end-to-end

#### Génération de données

- **@faker-js/faker** (dev) : Génération de données de test

## Swagger

La documentation Swagger est générée automatiquement et protégée par un token passé en query string (`?token={SWAGGER_TOKEN}`).

### Configuration dans main.ts

L’URL du serveur dans la spec OpenAPI est construite à partir de `API_URL` et `API_PORT` (lus via `ConfigService`) :

```typescript
const apiUrl = configService.getOrThrow<string>('API_URL');
const apiPort = configService.getOrThrow<string>('API_PORT');
// ...
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

### Fonctionnalités

- Documentation interactive des endpoints
- Test des requêtes directement depuis l'interface
- Authentification Bearer JWT intégrée
- Génération automatique des schémas à partir des DTOs
- Export JSON/YAML de la spécification OpenAPI

### Accès

- **URL locale** : `{API_URL}:{API_PORT}/api?token={SWAGGER_TOKEN}`
- **JSON spec** : `{API_URL}:{API_PORT}/api-json?token={SWAGGER_TOKEN}`

## Cookies et Sessions

L'application utilise des cookies pour stocker les tokens d'authentification.

### Configuration

Dans `main.ts` :

```typescript
import * as cookieParser from 'cookie-parser';

app.use(cookieParser());
```

### Types de cookies utilisés

1. **access_token** : JWT courte durée (15 minutes)
   - Utilisé pour authentifier les requêtes
   - Envoyé en cookie HTTP-only (comme le refresh token)

2. **refresh_token** : token aléatoire longue durée (24 h), stocké hashé en base
   - Envoyé en cookie HTTP-only
   - Utilisé pour renouveler l'access token
   - Plus sécurisé car non accessible en JavaScript

### Sécurité des cookies

Les cookies sont configurés avec :
- **httpOnly** : Non accessible via JavaScript (protection XSS)
- **secure** : Envoyé uniquement via HTTPS en production
- **sameSite** : Protection CSRF

## Validation des données

La validation est gérée automatiquement par `class-validator` via un pipe global.

### Configuration

Dans `main.ts` :

```typescript
app.useGlobalPipes(
  new I18nValidationPipe({
    transform: true,           // Transforme automatiquement les types
    whitelist: true,           // Supprime les propriétés non décorées
    forbidNonWhitelisted: true, // Rejette les propriétés inconnues
  }),
);
```

### Gestion des erreurs

Un filtre personnalisé (`ValidationExceptionFilter`) formate les erreurs de validation :

```typescript
app.useGlobalFilters(new ValidationExceptionFilter());
```

Les messages d'erreur sont traduits selon la langue de l'utilisateur via nestjs-i18n.

## Fichiers statiques

L'API sert des fichiers statiques (images de profil, médias) via Express.

### Configuration

Dans `main.ts` :

```typescript
app.useStaticAssets(join(process.cwd(), MEDIA_FOLDER.PUBLIC), {
  prefix: '/public/', // accès via /public/monfichier.jpg
});
```

### Structure des dossiers

- **public/** : Fichiers accessibles publiquement
  - `users/profilePicture/` : Images de profil publiques
- **private/** : Fichiers privés nécessitant une authentification

### Accès aux fichiers

- Public : `{API_URL}:{API_PORT}/public/users/profilePicture/image.jpg`
- Privé : Via endpoint protégé avec vérification d'authentification

## Logger (Winston)

L'API utilise **nest-winston** pour les logs applicatifs. La configuration est dans `src/config/logger.config.ts`.

### Format de log

```
2024-01-15T10:30:00.000Z [UserService] info: Utilisateur créé
```

| Champ | Description |
|---|---|
| `timestamp` | Date/heure ISO |
| `context` | Nom du service ou module NestJS (`Logger.name`) |
| `level` | Niveau de log (`info`, `warn`, `error`) |
| `message` | Message du log |
| `trace` | Stack trace (uniquement pour les erreurs) |

### Utilisation dans un service

```typescript
import { Logger } from '@nestjs/common';

@Injectable()
export class MonService {
  private readonly logger = new Logger(MonService.name);

  methode() {
    this.logger.log('Message info');
    this.logger.warn('Avertissement');
    this.logger.error('Erreur', error.stack);
  }
}
```

### Transports configurés

| Transport | Destination | Niveaux | Format |
|---|---|---|---|
| Console | Sortie standard | Tous | Coloré avec timestamp |
| File | `logs/error.log` | `error` uniquement | JSON |
| File | `logs/combined.log` | Tous | JSON |

Le dossier `logs/` est créé automatiquement à la racine du projet au premier démarrage.

---

## Constantes de l'application

Le fichier `src/domains/common/constants/constants.ts` centralise les constantes importantes.

### Durée de vie des tokens

```typescript
export const tokenExpirationTime = {
  accessToken: 15 * 60 * 1000,        // 15 minutes
  resetPasswordToken: 15 * 60 * 1000, // 15 minutes
  refreshToken: 24 * 60 * 60 * 1000,  // 24 heures
};
```

### Dossiers de médias

```typescript
export const MEDIA_FOLDER = {
  PUBLIC: 'public',
  PRIVATE: 'private',
};
```

Ces constantes sont utilisées partout dans l'application pour assurer la cohérence.

---

[← Retour au sommaire](./SUMMARY.md)