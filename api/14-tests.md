# Tests

[← Retour au sommaire](./SUMMARY.md)

Ce document présente l'architecture et l'organisation des tests e2e (end-to-end) dans l'API. Le projet utilise une base réutilisable pour simplifier la création et la maintenance des tests.

## Vue d'ensemble

Le projet utilise **Jest** et **Supertest** pour les tests e2e. Les tests sont organisés dans le dossier `test/` et suivent une structure modulaire avec des utilitaires centralisés pour éviter la duplication de code.

## Relation avec les diagrammes

Les tests e2e sont étroitement liés aux diagrammes de flux (flowcharts) qui documentent chaque endpoint. Pour plus de détails sur les diagrammes, voir [Diagrammes](./03-diagrams.md).

### Processus de développement recommandé

**En bonne pratique**, le processus de développement d'une route doit suivre cet ordre :

1. **Créer le diagramme** : Documenter le flux complet de la route dans `api/diagrams/`
   - Tous les cas d'usage (succès, échecs, validations)
   - Tous les branchements et conditions
   - Tous les codes de statut HTTP retournés
   - Toutes les opérations en base de données
   - Le diagramme définit le comportement attendu

2. **Écrire les tests** : Créer les tests e2e en se référant au diagramme dans `test/`
   - Un test par cas d'usage documenté dans le diagramme
   - Un test par branchement et condition
   - Un test par code de statut HTTP
   - Vérifier que tous les chemins du diagramme sont testés
   - Les tests échouent initialement (comportement attendu)

3. **Implémenter la route** : Développer le code pour faire passer les tests
   - Implémenter selon le diagramme et pour faire passer les tests
   - Itérer jusqu'à ce que tous les tests passent
   - Le code doit correspondre au diagramme et aux tests

### Avantages de cette approche

- **Design clair** : Le diagramme définit le comportement avant l'implémentation
- **Couverture complète** : Tous les cas d'usage du diagramme sont testés
- **Documentation vivante** : Les tests et le code reflètent exactement le diagramme
- **Maintenance facilitée** : Modifier le diagramme, puis adapter les tests, puis adapter la route
- **Qualité garantie** : Impossible d'oublier un cas d'usage (il est dans le diagramme et testé)
- **Révision facilitée** : Vérifier que le code correspond au diagramme et que les tests passent

### Exemple de correspondance diagramme / tests / route

Pour la route `auth/login`, le diagramme (`api/diagrams/auth/login/login.txt`) définit d'abord le comportement :
- ✅ Validation des données (email, password)
- ✅ Vérification des credentials
- ✅ Génération des tokens
- ✅ Révoquation des tokens précédents
- ✅ Définition des cookies sécurisés
- ✅ Gestion de `needToReconnect`

Les tests (`test/auth/login.spec.ts`) sont ensuite écrits en se référant au diagramme :
- ✅ Test avec email manquant → 400
- ✅ Test avec password manquant → 400
- ✅ Test avec mauvais credentials → 401
- ✅ Test avec bons credentials → 200 + tokens
- ✅ Test de révoquation des tokens précédents
- ✅ Test de durée de vie des tokens
- ✅ Test de sécurité des cookies
- ✅ Test de gestion de `needToReconnect`

La route est ensuite implémentée pour correspondre au diagramme et faire passer tous les tests.

Cette correspondance garantit que le diagramme, les tests et la route restent synchronisés et que tous les cas d'usage sont couverts.

## Structure des tests

Les tests sont organisés par domaine dans le dossier `test/` :

### Organisation par domaine

**Chaque domaine a son propre dossier** dans `test/`, correspondant à la structure des domaines dans `src/domains/`. Chaque route a son propre fichier de test.

### Nommage des fichiers

Les fichiers de test suivent la convention suivante :
- **Nom du fichier** : `{nom-de-la-route}.spec.ts`
- **Nom de la route** : Correspond au chemin de l'endpoint (ex: `auth/login` → `login.spec.ts`)
- **Extension** : `.spec.ts` pour les tests e2e

### Structure complète

```
test/
├── auth/                    # Tests d'authentification (domaine auth)
│   ├── login.spec.ts        # Route: POST /auth/login
│   ├── logout.spec.ts       # Route: POST /auth/logout
│   ├── refresh-session.spec.ts  # Route: POST /auth/refresh-session
│   └── validate-session.spec.ts # Route: POST /auth/validate-session
├── user/                    # Tests utilisateur (domaine user)
│   ├── profile.spec.ts      # Route: GET /user/profile
│   ├── register.spec.ts     # Route: POST /user/register
│   ├── update/
│   │   ├── update.spec.ts   # Route: POST /user/update
│   │   └── [fichiers de test] # Fichiers nécessaires aux tests (images, etc.)
│   └── update-password.spec.ts # Route: POST /user/update-password
├── reset-password/          # Tests de réinitialisation (domaine reset-password)
│   ├── reset-password.spec.ts      # Route: POST /reset-password/forgot
│   ├── update-password.spec.ts     # Route: POST /reset-password/update
│   └── validate.spec.ts           # Route: POST /reset-password/validate
├── test-setup.ts            # Configuration de base réutilisable
├── test-helpers.ts          # Fonctions utilitaires
├── cleaning.ts              # Nettoyage des données
├── jest-e2e.json            # Configuration Jest
└── README.md                # Documentation des utilitaires
```

## Configuration Jest

### Fichier de configuration

Le fichier `test/jest-e2e.json` configure Jest pour les tests e2e :

```json
{
  "moduleFileExtensions": ["js", "json", "ts"],
  "rootDir": ".",
  "roots": ["test"],
  "testEnvironment": "node",
  "testRegex": ".spec.ts$",
  "testPathIgnorePatterns": [
    "/node_modules/",
    "/dist/"
  ],
  "transform": {
    "^.+\\.(t|j)s$": "ts-jest"
  },
  "moduleNameMapper": {
    "^src/(.*)$": "<rootDir>/../src/$1",
    "^generated/(.*)$": "<rootDir>/../generated/$1"
  }
}
```

### Option expérimentale pour les uploads de fichiers

**Important** : Pour que les tests avec upload de fichiers fonctionnent correctement, il est nécessaire d'utiliser l'option expérimentale `--experimental-vm-modules` de Node.js.

Cette option est nécessaire car Jest utilise les modules VM de Node.js pour exécuter les tests, et les uploads de fichiers (multipart/form-data) nécessitent cette fonctionnalité expérimentale.

### Commandes de test

Les commandes disponibles dans `package.json` :

```bash
# Lancer tous les tests (utilise --experimental-vm-modules)
npm run test:win    # Sur Windows
npm run test:linux  # Sur Linux/Mac

# Lancer les tests e2e (nécessite --experimental-vm-modules pour les uploads)
npm run test:e2e
```

## Base réutilisable

### 1. **test-setup.ts** - Initialisation de l'application

Ce fichier contient la logique d'initialisation commune pour tous les tests.

#### Interface TestApp

L'interface `TestApp` contient tous les éléments nécessaires pour les tests. Voici chaque propriété et son utilisation :

```typescript
export interface TestApp {
  app: INestApplication;           // Application NestJS
  userService: UserService;        // Service utilisateur
  prismaService: PrismaService;    // Service Prisma
  agent: any;                      // Agent SuperTest avec cookies
  createdUserId: string;           // ID de l'utilisateur créé
  moduleFixture: TestingModule;   // Module de test
  createdUser: User | null;        // Utilisateur créé
}
```

| Propriété | Rôle |
|---|---|
| `app` | Serveur HTTP — utiliser avec `request(testApp.app.getHttpServer())` pour les requêtes sans cookies |
| `agent` | Agent SuperTest qui conserve les cookies — utiliser pour les requêtes authentifiées |
| `userService` | Manipuler les utilisateurs en test (`setNeedToReconnect`, `findById`…) |
| `prismaService` | Accès direct à la base de données (vérifications, nettoyage) |
| `createdUserId` | ID de l'utilisateur créé automatiquement à l'initialisation |
| `createdUser` | Objet utilisateur complet créé à l'initialisation |
| `moduleFixture` | Récupérer des services supplémentaires via `.get<T>()` |

#### Fonctions initTestApp() et cleanupTestApp()

- `initTestApp()` — crée le module de test, initialise l'application, crée un utilisateur de test en base et retourne un `TestApp`
- `cleanupTestApp(testApp, deleteUser?, email?)` — nettoie les données et ferme l'application

→ Voir [`test/test-setup.ts`](../../../api/test/test-setup.ts)

### 2. **test-helpers.ts** - Fonctions utilitaires

- `loginUser(testApp)` — connecte l'utilisateur de test et retourne `{ accessToken, refreshToken, cookieHeader }`
- `decodeJWT(token)` — décode le payload d'un JWT sans vérification de signature
- `verifyCookieSecurity(cookieHeader)` — vérifie que les cookies `access_token` et `refresh_token` ont les attributs `HttpOnly` et `SameSite=Strict`

→ Voir [`test/test-helpers.ts`](../../../api/test/test-helpers.ts)

### 3. **cleaning.ts** - Nettoyage des données

- `testUser` — objet utilisateur de test avec email et username horodatés pour garantir l'unicité
- `cleanupTestUser(userId, prismaService, deleteUser, email?)` — supprime les access tokens révoqués, refresh tokens, tokens de reset password, logs d'emails (si fourni), et optionnellement l'utilisateur lui-même

→ Voir [`test/cleaning.ts`](../../../api/test/cleaning.ts)

### Stratégie de nettoyage

Il existe **deux niveaux de nettoyage** dans les tests :

#### 1. **Nettoyage local/intermédiaire** (après chaque test)

Le nettoyage local est effectué **après chaque test individuel** pour nettoyer les données créées pendant le test, tout en conservant l'utilisateur de base pour les autres tests.

**Utilisation** : `cleanupTestUser(userId, prismaService, false)` - avec `deleteUser: false`

**Quand l'utiliser** :
- Après un test qui crée des tokens (login, refresh, etc.)
- Après un test qui modifie l'état de l'utilisateur et qui pourrait affecter les tests suivants
- Pour nettoyer les tokens/réinitialisations créés pendant un test spécifique
- Pour remettre l'utilisateur dans un état propre pour le test suivant

**Ce qui est nettoyé** :
- ✅ Les access tokens révoqués
- ✅ Les refresh tokens
- ✅ Les tokens de réinitialisation de mot de passe
- ✅ Les logs d'emails (si email fourni)
- ❌ L'utilisateur lui-même (conservé pour les autres tests)

**Exemple** :

```typescript
it('✅ devrait retourner 200 avec token valide', async () => {
  // Se connecter (crée des tokens)
  loginResult = await loginUser(testApp);

  const response = await testApp.agent
    .post('/auth/validate-session')
    .expect(200);

  // Nettoyage local : nettoie les tokens créés mais garde l'utilisateur
  await cleanupTestUser(testApp.createdUserId, testApp.prismaService, false);

  expect(response.body.message).toBe('Session ok');
});
```

#### 2. **Nettoyage global** (en fin de suite de tests)

Le nettoyage global est effectué **une seule fois à la fin de tous les tests** dans `afterAll`, pour tout nettoyer, y compris l'utilisateur de base.

**Utilisation** : `cleanupTestApp(testApp, true)` - avec `deleteUser: true` (par défaut)

**Quand l'utiliser** :
- Toujours dans `afterAll` pour nettoyer toutes les données de la suite de tests
- Pour supprimer l'utilisateur de test créé lors de l'initialisation
- Pour fermer l'application de test

**Ce qui est nettoyé** :
- ✅ Les access tokens révoqués
- ✅ Les refresh tokens
- ✅ Les tokens de réinitialisation de mot de passe
- ✅ Les logs d'emails (si email fourni)
- ✅ **L'utilisateur lui-même** (supprimé)
- ✅ Fermeture de l'application NestJS

**Exemple** :

```typescript
describe('auth/login (e2e)', () => {
  let testApp: TestApp;

  beforeAll(async () => {
    // Crée l'utilisateur de test automatiquement
    testApp = await initTestApp();
  });

  afterAll(async () => {
    // Nettoyage global : nettoie TOUT, y compris l'utilisateur
    await cleanupTestApp(testApp, true); // true = supprimer l'utilisateur
  });

  // ... tous les tests ...
});
```

### Comparaison des deux stratégies

| Aspect | Nettoyage local | Nettoyage global |
|--------|-----------------|------------------|
| **Quand** | Après chaque test individuel | Une fois dans `afterAll` |
| **Fonction** | `cleanupTestUser(userId, prismaService, false)` | `cleanupTestApp(testApp, true)` |
| **Utilisateur** | Conservé | Supprimé |
| **Tokens** | Nettoyés | Nettoyés |
| **Application** | Non fermée | Fermée |
| **Objectif** | Remettre l'utilisateur dans un état propre | Nettoyer complètement la suite de tests |

### Bonnes pratiques

1. **Nettoyage local** :
   - Utiliser après les tests qui créent des tokens (login, refresh)
   - Utiliser après les tests qui modifient l'état de l'utilisateur
   - Toujours avec `deleteUser: false` pour conserver l'utilisateur

2. **Nettoyage global** :
   - Toujours utiliser dans `afterAll`
   - Toujours avec `deleteUser: true` (par défaut)
   - Une seule fois par suite de tests

3. **Combinaison** :
   - Les deux peuvent être combinés : nettoyage local après chaque test + nettoyage global dans `afterAll`
   - Le nettoyage global assure que tout est nettoyé même si un test échoue


## Création d'un test

### Structure de base d'un test

```typescript
import * as request from 'supertest';
import { TestApp, cleanupTestApp, initTestApp } from '../test-setup';

describe('mon-endpoint (e2e)', () => {
  let testApp: TestApp;

  beforeAll(async () => {
    testApp = await initTestApp();
  });

  afterAll(async () => {
    await cleanupTestApp(testApp, true); // true = supprimer l'utilisateur
  });

  it('✅ devrait faire quelque chose', async () => {
    const response = await testApp.agent
      .get('/mon-endpoint')
      .expect(200);

    expect(response.body.message).toBe('Success');
  });
});
```

### Test avec authentification

```typescript
import * as request from 'supertest';
import { TestApp, cleanupTestApp, initTestApp } from '../test-setup';
import { LoginResult, loginUser } from '../test-helpers';

describe('mon-endpoint-protégé (e2e)', () => {
  let testApp: TestApp;
  let loginResult: LoginResult;

  beforeAll(async () => {
    testApp = await initTestApp();
  });

  afterAll(async () => {
    await cleanupTestApp(testApp, true);
  });

  it('✅ devrait accéder à l\'endpoint protégé après connexion', async () => {
    // Se connecter et récupérer les tokens
    loginResult = await loginUser(testApp);

    // Utiliser l'agent qui conserve automatiquement les cookies
    const response = await testApp.agent
      .get('/mon-endpoint-protégé')
      .expect(200);

    expect(response.body.message).toBe('Success');
  });

  it('❌ devrait retourner 401 sans authentification', async () => {
    const response = await request(testApp.app.getHttpServer())
      .get('/mon-endpoint-protégé')
      .expect(401);

    expect(response.body.message).toBe('Token invalid or expired');
  });
});
```

## Bonnes pratiques

### 1. **Structure des tests**

- Utiliser `beforeAll` pour initialiser l'application
- Utiliser `afterAll` pour nettoyer les données
- Un test par cas d'usage (succès, échec, validation, etc.)

### 2. **Nommage des tests**

- Utiliser des descriptions claires : `✅ devrait faire X` pour les succès, `❌ devrait retourner Y` pour les échecs
- Préfixer avec des émojis pour améliorer la lisibilité (✅ ❌)

### 3. **Utilisation des agents**

- **`testApp.agent`** : Pour les requêtes qui doivent conserver les cookies (après login)
- **`request(testApp.app.getHttpServer())`** : Pour les requêtes sans cookies ou avec des cookies personnalisés

### 4. **Nettoyage des données**

- Toujours nettoyer les données créées pendant les tests
- Utiliser `cleanupTestApp()` dans `afterAll`
- Utiliser `cleanupTestUser()` pour nettoyer les données intermédiaires si nécessaire

### 5. **Tests d'authentification**

- Utiliser `loginUser()` pour se connecter
- Vérifier les codes de statut HTTP appropriés (200, 400, 401, etc.)
- Tester les cas d'erreur (token invalide, expiré, etc.)

### 6. **Tests de validation**

- Tester tous les cas de validation (champs manquants, invalides, etc.)
- Vérifier les messages d'erreur appropriés
- Tester les contraintes métier (unicité, format, etc.)

### 7. **Tests avec fichiers**

- Utiliser des fichiers de test dans le dossier du test
- Vérifier l'existence des fichiers avant de les utiliser
- Nettoyer les fichiers créés temporairement
- **Important** : Utiliser l'option `--experimental-vm-modules` pour que les tests avec upload de fichiers fonctionnent

**Exemple de test avec upload de fichier :**

```typescript
it('✅ Devrait pouvoir mettre à jour une photo de profil', async () => {
  const loginResult = await loginUser(testApp);

  const filePath = path.join(__dirname, './avatar-other.png');
  expect(fs.existsSync(filePath)).toBe(true);

  const response = await request(testApp.app.getHttpServer())
    .post('/user/update')
    .set('Cookie', `access_token=${loginResult.accessToken}`)
    .field('notificationEmail', 'false')
    .field('gender', 'male')
    .attach('profilePicture', filePath);

  expect(response.status).toBe(HttpStatus.OK);
});
```

**Note** : Pour exécuter ce type de test, utilisez une commande qui inclut `--experimental-vm-modules` :
```bash
# Sur Windows
set NODE_OPTIONS=--experimental-vm-modules && npm run test:e2e

# Sur Linux/Mac
NODE_OPTIONS=--experimental-vm-modules npm run test:e2e
```

### 8. **Vérification en base de données**

- Utiliser `testApp.prismaService` pour vérifier les données en base
- Vérifier les relations et les données associées
- Nettoyer les données après vérification

## Exemples de tests existants

### Test simple (login.spec.ts)

```typescript
describe('auth/login (e2e)', () => {
  let testApp: TestApp;
  let loginResult: LoginResult;

  beforeAll(async () => {
    testApp = await initTestApp();
  });

  afterAll(async () => {
    await cleanupTestApp(testApp, true);
  });

  it('✅ devrait se connecter avec succès avec de bons credentials', async () => {
    loginResult = await loginUser(testApp);

    expect(loginResult.accessToken).toBeDefined();
    expect(loginResult.refreshToken).toBeDefined();
    expect(loginResult.accessToken.length).toBeGreaterThan(0);
    expect(loginResult.refreshToken.length).toBeGreaterThan(0);
  });

  it('❌ devrait retourner 401 avec de mauvais credentials', async () => {
    const response = await request(testApp.app.getHttpServer())
      .post('/auth/login')
      .send({
        email: testUser.email,
        password: 'MauvaisMotDePasse',
      })
      .expect(401);

    expect(response.body.message).toBe('Invalid credentials');
  });
});
```