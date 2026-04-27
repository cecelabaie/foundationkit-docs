# Configuration

Ce document présente les différentes configurations du projet frontend.

## next.config.ts

Le fichier `next.config.ts` configure Next.js avec des optimisations pour la performance, la sécurité et l'expérience utilisateur. Les variables d'environnement sont chargées depuis le **premier fichier trouvé** dans l'ordre : `.env.production`, `.env.local`, `.env`.

- **`npm run dev`** : ignore `.env.production`, charge `.env.local` en priorité puis `.env`
- **`npm run build`** : charge `.env.production` en priorité, puis `.env.local`, puis `.env`

### Principales fonctionnalités

- **Optimisation des performances**
  - **Compression gzip/brotli** : Compresse automatiquement les fichiers JavaScript, CSS et HTML pour réduire leur taille, accélérant les temps de chargement
  - **Tree-shaking des icônes** : Utilise `optimizePackageImports` pour éliminer les icônes non utilisées de @iconify/react et lucide-react
  - **Analyse de bundle** : Permet d'identifier les packages trop volumineux avec `@next/bundle-analyzer` quand `ANALYZE=true`

- **Sécurité renforcée**
  - **X-Frame-Options** : Empêche le site d'être affiché dans une iframe (protection contre le clickjacking)
  - **Content-Security-Policy (CSP)** : Limite les sources de contenu pour bloquer les attaques XSS et les injections
  - **Strict-Transport-Security (HSTS)** : Force l'utilisation de HTTPS pour toutes les connexions
  - **Permissions-Policy** : Désactive l'accès aux API sensibles comme la caméra ou le microphone
  - **Masquage de "X-Powered-By"** : Évite de révéler la technologie utilisée (Next.js) aux attaquants potentiels

- **Optimisation du cache**
  - **Cache longue durée** : Configure un cache d'un an (31536000s) pour les assets statiques qui changent rarement
  - **Cache spécifique** : Configuration adaptée par type de ressource (favicon, images, etc.)

- **Redirections et rewrites**
  - **Redirection HTTP vers HTTPS** : Force l'utilisation de HTTPS en production pour la sécurité

### Images

Les images provenant de l'API sont autorisées par défaut via `images.remotePatterns`, qui utilise `NEXT_PUBLIC_SERVER_URL` pour autoriser les chemins `/public/**` et `/private/**`.

**Important :** La directive CSP `img-src` limite par défaut les images à HTTPS uniquement. HTTP est activé automatiquement dans `img-src` lorsque `NEXT_PUBLIC_SERVER_URL` utilise le protocole HTTP (ex. en développement avec `http://localhost:3001`), afin que les images de profil et autres ressources de l'API s'affichent correctement.

Pour les images venant d'un site externe (CDN, banques d'images comme Pexels, etc.), il faut les ajouter explicitement dans `remotePatterns` avec le `protocol`, `hostname` et `pathname` correspondants.

## tsconfig.json

Configuration TypeScript pour le projet Next.js avec App Router.

### Points clés

- **Target ES2017** : Cible les navigateurs modernes tout en gardant une bonne compatibilité
- **Mode strict** : Active toutes les vérifications de type strictes pour éviter les erreurs courantes
- **Alias de chemins** : Configure `@/*` pour importer depuis `./src/*` et éviter les chemins relatifs complexes
- **JSX `react-jsx`** : utilise le [nouveau transform JSX](https://react.dev/blog/2020/09/22/introducing-the-new-jsx-transform) de React (`"jsx": "react-jsx"` dans `tsconfig.json`) — pas besoin d’importer React dans chaque fichier pour le JSX

## tailwind.config.ts

Configuration de Tailwind CSS, le framework CSS utilitaire utilisé dans le projet.

### Configuration

- **Chemins de contenu** : Indique à Tailwind quels fichiers analyser pour extraire les classes utilisées
- **Thème extensible** : Permet d'étendre le thème par défaut avec des couleurs, espacements, etc. personnalisés

## orval.config.ts

Configuration d'Orval, l'outil qui génère automatiquement des hooks React Query et des types TypeScript à partir d'une spécification OpenAPI (Swagger). Comme `next.config.ts`, il charge les variables d'environnement depuis le **premier fichier trouvé** dans l'ordre `.env.production`, `.env.local`, `.env`.

### Points clés

- **Source de données** : Utilise la variable d'environnement `SWAGGER_JSON` pour spécifier l'URL du fichier Swagger
- **Mode de sortie** : Utilise `tags-split` pour organiser le code généré par tags API (endpoints groupés logiquement)
- **Double génération** : Configure deux générateurs distincts :
  - `api` : Génère des hooks React Query pour les appels API
  - `apiZodSchemas` : Génère des schémas Zod pour la validation des données

### Configuration React Query

```typescript
output: {
  mode: 'tags-split',
  target: 'src/api/generated',
  schemas: 'src/api/generated/schemas',
  client: 'react-query',
  mock: false,
  baseUrl: {
    getBaseUrlFromSpecification: true,
  },
  // ...
}
```

- **target** : Chemin où les hooks seront générés
- **schemas** : Chemin pour les interfaces TypeScript
- **client** : Utilise React Query pour la gestion des requêtes et du cache
- **mock** : Désactive la génération de données fictives
- **baseUrl** : Récupère l'URL de base depuis la spécification Swagger

### Personnalisation avancée

```typescript
override: {
  query: {
    useQuery: true,
  },
  enumGenerationType: 'enum',
  mutator: './src/transformers/customMutator.ts',
}
```

- **useQuery** : Active la génération des hooks `useQuery`
- **enumGenerationType** : Génère des énumérations TypeScript plutôt que des unions de chaînes
- **mutator** : Utilise un client HTTP personnalisé (`customMutator.ts`) qui :
  - Configure Axios avec les credentials pour envoyer les cookies
  - Gère l'annulation des requêtes
  - Permet d'ajouter des intercepteurs globaux

### Génération de schémas Zod

```typescript
apiZodSchemas: {
  input: {
    target: SWAGGER_JSON,
  },
  output: {
    client: 'zod',
    target: 'src/api/generated/zod',
    mode: 'tags-split',
  },
}
```

- **client** : Utilise Zod pour la validation des données
- **target** : Chemin où les schémas Zod seront générés
- **mode** : Organise également les schémas par tags API


## eslint.config.mjs

Configuration ESLint moderne (format plat) qui définit les règles de qualité du code.

### Règles principales

- **TypeScript** : Règles spécifiques pour éviter les erreurs courantes en TypeScript
- **React** : Bonnes pratiques React comme l'utilisation correcte des hooks et des props
- **Organisation des imports** : Groupe les imports par catégorie et les trie alphabétiquement pour une meilleure lisibilité
- **Accessibilité** : Règles jsx-a11y pour garantir que l'application est accessible aux utilisateurs handicapés
- **Nettoyage automatique** : Détecte et permet de supprimer les imports non utilisés
- **Intégration Prettier** : Combine ESLint et Prettier pour un formatage cohérent

## package.json

### Port Next.js

Le script `dev` dans `package.json` lance Next.js avec l’option `-p` pour définir le port (ex. `next dev --turbopack -p 3000`). **Ce port doit être identique à `APP_PORT`** défini dans votre fichier `.env.local`. Sinon, les URL de l’application et la configuration peuvent être incohérentes.

### Commandes principales

| Commande | Description |
|----------|-------------|
| `npm run dev` | Lance le serveur de développement avec Turbopack (compilateur rapide) |
| `npm run build` | Construit l'application optimisée pour la production |
| `npm run build:win` | Régénère l'API, formate le CSS et construit l'application (Windows) |
| `npm run build:linux` | Régénère l'API, formate le CSS et construit l'application (Linux) |
| `npm run analyze` | Construit l'application avec visualisation de la taille des packages |
| `npm run start` | Démarre l'application construite en mode production |
| `npm run lint` | Vérifie le code avec ESLint pour détecter les problèmes |
| `npm run lint:fix` | Corrige automatiquement les problèmes ESLint quand possible |
| `npm run lint:generated:fix` | Corrige les problèmes ESLint dans le code généré par Orval |
| `npm run gen:win` | Régénère les types et hooks d'API avec Orval (Windows) |
| `npm run gen:linux` | Régénère les types et hooks d'API avec Orval (Linux) |


## Constantes

Le fichier `src/constants/constants.ts` centralise les constantes importantes de l'application :

### API_PATHS

Définit les chemins d'API pour les différentes fonctionnalités :

```typescript
export const API_PATHS = {
  AUTH: {
    VALIDATE_SESSION: '/auth/validate-session',
    REFRESH_SESSION: '/auth/refresh-session',
    LOGIN: '/auth/login',
    LOGOUT: '/auth/logout',
  },
  USER: {
    REGISTER: '/user/register',
    UPDATE: '/user/update',
    UPDATE_PASSWORD: '/user/update-password',
  },
  RESET_PASSWORD: {
    FORGOT: '/reset-password/forgot',
    VALIDATE: '/reset-password/validate',
    UPDATE: '/reset-password/update',
  },
};
```

### APP_PATHS

Définit les chemins de navigation de l'application :

```typescript
export const APP_PATHS = {
  LANDING_PAGE: NEXT_PUBLIC_APP_URL,

  // Routes non connectées
  HOME: '/',
  LOGIN: '/login',
  REGISTER: '/register',
  FORGOT_PASSWORD: '/forgot-password',
  NEW_PASSWORD: '/forgot-password/new-password',

  // Routes connectées
  PROFILE: '/profile',
  LOGOUT: '/logout',

  // Routes publiques
  SAMPLE: '/sample',
  NOT_FOUND: '/not-found',
};
```

### ROUTES_WITHOUT_RETRY

Liste des routes exclues du mécanisme de retry automatique en cas d'erreur 401. Ces routes sont généralement liées à l'authentification elle-même, où un retry créerait une boucle infinie.

Pour plus de détails sur le système de retry et la gestion des erreurs 401, consultez [data.md](./11-data.md) et [session.md](./12-session.md).

```typescript
export const ROUTES_WITHOUT_RETRY = [
  API_PATHS.AUTH.VALIDATE_SESSION,
  API_PATHS.AUTH.REFRESH_SESSION,
  API_PATHS.AUTH.LOGIN,
  API_PATHS.AUTH.LOGOUT,
  API_PATHS.USER.REGISTER,
  API_PATHS.USER.VERIFY_ACCOUNT,
  API_PATHS.RESET_PASSWORD.FORGOT,
  API_PATHS.RESET_PASSWORD.VALIDATE,
  API_PATHS.RESET_PASSWORD.UPDATE,
];
```

## Variables d'environnement

L'application utilise plusieurs variables d'environnement définies dans un fichier `.env.production`, `.env.local` ou `.env` à la racine du projet frontend. Le **premier fichier trouvé** dans l'ordre `.env.production`, `.env.local`, `.env` est chargé. En `npm run dev`, Next.js ignore `.env.production` donc c'est le premier fichier trouvé entre `.env.local` et `.env` qui est utilisé. En `npm run build`, c'est le premier fichier trouvé entre `.env.production`, `.env.local` et `.env`.

### Description des variables

| Variable | Description | Utilisation |
|----------|-------------|-------------|
| `APP_PORT` | Port de l'application frontend Next.js | Doit correspondre au port défini dans le script `dev` du `package.json` (option `-p`) |
| `NEXT_PUBLIC_APP_URL` | URL de l'application frontend | Accessible côté client, utilisée pour les URL absolues |
| `NEXT_PUBLIC_SERVER_URL` | URL du serveur backend | Accessible côté client, utilisée pour les appels API |
| `SWAGGER_TOKEN` | Token d'accès au Swagger | Requis pour accéder aux routes Swagger protégées côté API |
| `SWAGGER_JSON` | URL du fichier Swagger (inclut `?token={SWAGGER_TOKEN}`) | Utilisée par Orval pour générer le code TypeScript |
| `NODE_ENV` | Environnement d'exécution (development, production) | Détermine les optimisations et comportements spécifiques à l'environnement |

### Sécurité des variables d'environnement

- Les variables préfixées par `NEXT_PUBLIC_` sont exposées au client (navigateur)
- Les autres variables sont uniquement disponibles côté serveur
- Les variables sensibles (clés API, secrets) ne doivent jamais utiliser le préfixe `NEXT_PUBLIC_`