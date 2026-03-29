# Traductions

[← Retour au sommaire](./SUMMARY.md)

Ce document présente le système d'internationalisation (i18n) de l'API. Le projet utilise **nestjs-i18n** pour gérer les traductions des messages d'erreur et de succès selon la langue de l'utilisateur.

## Vue d'ensemble

L'API utilise **nestjs-i18n** pour gérer les traductions. Le système détecte automatiquement la langue de l'utilisateur via le header `Accept-Language` et retourne les messages dans la langue appropriée.

### Avantages de cette architecture

- **Multilingue** : Support de plusieurs langues (français, anglais)
- **Détection automatique** : Langue détectée via le header `Accept-Language`
- **Messages traduits** : Tous les messages d'erreur et de succès sont traduits
- **Validation traduite** : Les messages de validation sont automatiquement traduits
- **Maintenance facilitée** : Fichiers JSON organisés par domaine

## Configuration

### Module I18n dans AppModule

Le module I18n est configuré dans `src/domains/app/module/app.module.ts` :

```typescript
I18nModule.forRootAsync({
  useFactory: (configService: ConfigService) => {
    const srcI18n = join(process.cwd(), 'src', 'i18n');
    const distI18n = join(process.cwd(), 'dist', 'i18n');
    const path = existsSync(srcI18n) ? srcI18n : distI18n;
    return {
      fallbackLanguage: configService.getOrThrow<string>('FALLBACK_LANGUAGE'),
      loaderOptions: { path, watch: true },
    };
  },
  resolvers: [new AcceptLanguageResolver({ matchType: 'strict' })],
  inject: [ConfigService],
}),
```

> **Note** : `join` est importé de `node:path`, `existsSync` de `node:fs`. Le chemin `src/i18n` est utilisé en développement, `dist/i18n` après build.

### Configuration du resolver

**AcceptLanguageResolver** : Détecte la langue depuis le header HTTP `Accept-Language`

- **`matchType: 'strict'`** : Correspondance stricte avec la langue demandée
- Si la langue n'est pas disponible, utilise la langue de fallback (`FALLBACK_LANGUAGE`)

### Langue de fallback

La langue de fallback est définie dans la variable d'environnement `FALLBACK_LANGUAGE` (par défaut : `fr` ou `en`).

### I18nValidationPipe

Le `I18nValidationPipe` est configuré globalement dans `src/main.ts` :

```typescript
app.useGlobalPipes(
  new I18nValidationPipe({
    transform: true,
    whitelist: true,
    forbidNonWhitelisted: true,
  }),
);
```

Ce pipe permet de traduire les messages de validation des DTOs.

## Structure des fichiers de traduction

### Organisation par domaine

Les fichiers de traduction sont organisés par domaine dans `src/i18n/{lang}/` :

```
src/i18n/
├── fr/                    # Traductions françaises
│   ├── auth.json          # Messages d'authentification
│   ├── common.json        # Messages communs
│   ├── mail.json          # Messages d'email
│   ├── media.json         # Messages de médias
│   ├── reset-password.json # Messages de réinitialisation
│   ├── revoked-token.json # Messages de tokens révoqués
│   └── user.json          # Messages utilisateur
└── en/                    # Traductions anglaises
    ├── auth.json
    ├── common.json
    ├── mail.json
    ├── media.json
    ├── reset-password.json
    ├── revoked-token.json
    └── user.json
```

### Structure des fichiers JSON

Chaque fichier JSON suit une structure hiérarchique organisée par fonctionnalité :

```json
{
  "domain": {
    "action": {
      "success": "Message de succès",
      "error": {
        "errorType": "Message d'erreur"
      },
      "validation": {
        "field": {
          "rule": "Message de validation"
        }
      }
    }
  }
}
```

**Exemple** : `src/i18n/fr/user.json`

```json
{
  "register": {
    "success": "Utilisateur créé avec succès",
    "validation": {
      "email": {
        "isEmail": "L'email doit être une adresse email valide",
        "isNotEmpty": "L'email ne peut pas être vide",
        "isNotExist": "L'email existe déjà"
      },
      "password": {
        "isString": "Le mot de passe doit être une chaîne de caractères",
        "minLength": "Le mot de passe doit contenir au moins {constraints.0} caractères"
      }
    },
    "error": {
      "failedToCreateUser": "Erreur lors de la création de l'utilisateur"
    }
  }
}
```

## Utilisation dans le code

### 1. Dans les services

**Injection du service** :

```typescript
import { I18nService } from 'nestjs-i18n';

@Injectable()
export class UserService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly i18n: I18nService,
  ) {}

  async findOneByUsername(username: string): Promise<User | undefined> {
    const user = await this.prisma.user.findUnique({
      where: { username },
    });
    
    if (!user) {
      throw new NotFoundException(
        this.i18n.t('user.service.userNotFound')
      );
    }
    return user;
  }
}
```

**Utilisation** :

```typescript
// Traduction simple
this.i18n.t('user.service.userNotFound')
// Retourne : "Utilisateur non trouvé" (fr) ou "User not found" (en)

// Traduction avec paramètres personnalisés
this.i18n.t('common.throttler.too-many-requests', {
  args: { ttl: 5 }
})
// Retourne : "Trop de requêtes, veuillez réessayer dans 5 minutes" (fr)
```

### 2. Dans les contrôleurs

**Injection du service** :

```typescript
import { I18nService } from 'nestjs-i18n';

@Controller('user')
export class UserController {
  constructor(
    private readonly userService: UserService,
    private readonly i18n: I18nService,
  ) {}

  @Get('profile')
  async getProfile(@Req() req: RequestWithCookies) {
    const user = await this.userService.findOne(payload.sub);
    
    return ResponseUtil.success(
      HttpStatus.OK,
      this.i18n.t('user.profile.success'),
      user,
    );
  }
}
```

**Exemple avec messages d'erreur** :

```typescript
@Post('register')
async register(@Body() userRegisterDto: UserRegisterBodyDTO) {
  if (userRegisterDto.password !== userRegisterDto.confirmPassword) {
    const validationErrors: ValidationViolation[] = [
      {
        field: 'password',
        message: this.i18n.t('user.register.validation.password.isNotEqual'),
      },
      {
        field: 'confirmPassword',
        message: this.i18n.t('user.register.validation.confirmPassword.isNotEqual'),
      },
    ];
    throw new BadRequestException(
      ResponseUtil.validationError(validationErrors),
    );
  }
  
  await this.userService.create(userRegisterDto);
  return ResponseUtil.success(
    HttpStatus.CREATED,
    this.i18n.t('user.register.success'),
  );
}
```

### 3. Dans les DTOs (validation)

Les messages de validation dans les DTOs utilisent les clés de traduction. Il existe deux façons de gérer les paramètres selon le type de décorateur :

#### Clé simple (sans paramètres)

Pour les décorateurs **sans paramètres** (`@IsString()`, `@IsEmail()`, `@IsNotEmpty()`, etc.) :

```typescript
import { IsEmail, IsNotEmpty, IsString } from 'class-validator';

export class UserRegisterBodyDTO {
  @IsString({
    message: 'user.register.validation.email.isString',
  })
  @IsEmail({}, {
    message: 'user.register.validation.email.isEmail',
  })
  @IsNotEmpty({
    message: 'user.register.validation.email.isNotEmpty',
  })
  email: string;
}
```

#### i18nValidationMessage (avec paramètres)

Pour les décorateurs **avec paramètres** (`@MinLength(8)`, `@MaxLength(50)`, `@Min(18)`, `@IsEnum()`, etc.), utilisez **`i18nValidationMessage()`** pour injecter automatiquement les valeurs des contraintes dans les messages.

**Exemple rapide** :

```typescript
import { MinLength } from 'class-validator';
import { i18nValidationMessage } from 'nestjs-i18n';

@MinLength(8, {
  message: i18nValidationMessage('user.register.validation.password.minLength'),
})
password: string;
```

**Important** : Pour les détails complets, exemples et guide d'utilisation, voir la section [Traductions avec paramètres](#traductions-avec-paramètres).

### 4. Utilisation de I18nContext

**I18nContext** permet d'accéder au service i18n sans injection de dépendance :

```typescript
import { I18nContext } from 'nestjs-i18n';

// Dans un guard, interceptor, ou filtre
const i18n = I18nContext.current();
if (i18n) {
  return i18n.t('common.throttler.too-many-requests', {
    args: { ttl: ttlMinutes },
  });
}
```

**Exemple dans le ThrottlerGuard** :

```typescript
errorMessage: (context, details) => {
  const ttlMinutes = Math.ceil(details.ttl / 1000 / 60);
  const i18n = I18nContext.current();

  if (i18n) {
    return i18n.t('common.throttler.too-many-requests', {
      args: { ttl: ttlMinutes },
    });
  }
  return `Too many requests, please try again in ${ttlMinutes} minutes`;
}
```

## Traductions avec paramètres

Il existe **deux types de paramètres** dans les traductions :

1. **Paramètres personnalisés** : Passés manuellement via `args` dans le code (services, contrôleurs)
2. **Contraintes automatiques** : Injectées automatiquement depuis les décorateurs de validation via `i18nValidationMessage()`

### 1. Paramètres personnalisés (dans le code)

Pour les traductions avec paramètres personnalisés, utilisez `i18n.t()` avec l'objet `args` :

```json
{
  "common": {
    "throttler": {
      "too-many-requests": "Trop de requêtes, veuillez réessayer dans {ttl} minutes"
    }
  }
}
```

```typescript
this.i18n.t('common.throttler.too-many-requests', {
  args: { ttl: 5 }
})
// Retourne : "Trop de requêtes, veuillez réessayer dans 5 minutes"
```

### 2. Contraintes automatiques (dans les DTOs)

Pour les décorateurs de validation qui ont des paramètres (`@MinLength(8)`, `@MaxLength(50)`, `@Min(18)`, `@IsEnum()`, etc.), utilisez **`i18nValidationMessage()`** pour injecter automatiquement les valeurs des contraintes dans les messages.

**Import** :

```typescript
import { i18nValidationMessage } from 'nestjs-i18n';
```

**Syntaxe dans les fichiers JSON** : Utilisez `{constraints.0}` pour le premier paramètre du décorateur :

- **`{constraints.0}`** : Premier paramètre du décorateur (index 0) - **Syntaxe utilisée dans le projet**
- **`{constraints.1}`** : Deuxième paramètre du décorateur (index 1)
- **`{$constraint0}`** : Alternative (fonctionne aussi)

**Note** : Il est possible d'utiliser une clé simple sans `i18nValidationMessage()` si la valeur de la contrainte est fixe et ne changera jamais (ex: `@MinLength(4)` avec un message en dur "4 caractères" dans le JSON). Cependant, `i18nValidationMessage()` est recommandé pour une meilleure maintenabilité.

#### Exemple avec @MinLength

```typescript
import { MinLength } from 'class-validator';
import { i18nValidationMessage } from 'nestjs-i18n';

export class UserRegisterBodyDTO {
  @MinLength(8, {
    message: i18nValidationMessage('user.register.validation.password.minLength'),
  })
  password: string;
}
```

**Fichier de traduction** :

```json
{
  "user": {
    "register": {
      "validation": {
        "password": {
          "minLength": "Le mot de passe doit contenir au moins {constraints.0} caractères"
        }
      }
    }
  }
}
```

**Résultat** : Le `8` de `@MinLength(8)` est automatiquement injecté dans `{constraints.0}`.

**Avantage** : Si vous changez `@MinLength(8)` en `@MinLength(10)`, la traduction s'adapte automatiquement sans modifier le fichier JSON.

#### Exemple avec @IsEnum et arguments personnalisés

Pour passer des arguments personnalisés en plus des contraintes automatiques :

```typescript
import { IsEnum } from 'class-validator';
import { i18nValidationMessage } from 'nestjs-i18n';
import { Gender } from 'generated/prisma';

export class UserRegisterBodyDTO {
  @IsEnum(Gender, {
    message: i18nValidationMessage('user.register.validation.gender.isEnum', {
      enumValues: Object.values(Gender).join(', '),
    }),
  })
  gender: Gender;
}
```

**Fichier de traduction** :

```json
{
  "user": {
    "register": {
      "validation": {
        "gender": {
          "isEnum": "Le genre doit être une valeur valide. Valeurs possibles : {enumValues}"
        }
      }
    }
  }
}
```

**Résultat** : `{enumValues}` est remplacé par la liste des valeurs de l'enum (ex: `"MALE, FEMALE"`).

### Quand utiliser i18nValidationMessage ?

**Règle simple** : Utilisez `i18nValidationMessage()` si le décorateur a des paramètres que vous voulez afficher dynamiquement dans le message.

| Décorateur | Paramètres ? | Utilisez `i18nValidationMessage` ? |
|------------|--------------|-------------------------------------|
| `@IsString()` | Non | ❌ Non : `message: 'clé'` |
| `@IsEmail()` | Non | ❌ Non : `message: 'clé'` |
| `@IsNotEmpty()` | Non | ❌ Non : `message: 'clé'` |
| `@MinLength(8)` | **Oui** (afficher "8") | ✅ **Recommandé** : `message: i18nValidationMessage('clé')` |
| `@MaxLength(50)` | **Oui** (afficher "50") | ✅ **Recommandé** : `message: i18nValidationMessage('clé')` |
| `@Min(18)` | **Oui** (afficher "18") | ✅ **Recommandé** : `message: i18nValidationMessage('clé')` |
| `@IsEnum(Gender)` | **Oui** (lister les valeurs) | ✅ **Recommandé** : `message: i18nValidationMessage('clé', { args })` |

## Gestion des traductions dans le filtre de validation

Le `ValidationExceptionFilter` traduit automatiquement les messages de validation :

**Important** : `i18nValidationMessage()` génère automatiquement un format spécial (`"clé|{...args}"`) que le filtre sait parser et traduire avec les arguments.

Pour plus de détails, voir [Filters](./05-filters.md).

## Bonnes pratiques

### 1. **Organisation par domaine**

- Un fichier JSON par domaine (auth, user, reset-password, etc.)
- Structure hiérarchique cohérente dans tous les domaines
- Messages de succès, erreur et validation séparés

### 2. **Nommage des clés**

Toutes les clés sont en **kebab-case** et suivent une structure hiérarchique cohérente.

### 3. **Messages de validation**

- Toujours utiliser les clés de traduction dans les DTOs
- Ne jamais mettre de messages en dur dans les décorateurs
- **Utiliser `i18nValidationMessage()` pour les décorateurs avec paramètres** (`@MinLength`, `@MaxLength`, `@Min`, `@IsEnum`, etc.)
- Utiliser `{constraints.0}` dans les fichiers JSON pour les contraintes automatiques
- Utiliser `message: 'clé'` pour les décorateurs sans paramètres

### 4. **Cohérence entre langues**

- Maintenir la même structure dans tous les fichiers de langue
- S'assurer que toutes les clés existent dans toutes les langues
- Utiliser la langue de fallback si une clé manque

### 5. **Paramètres dans les traductions**

- Utiliser des noms de paramètres clairs : `{ttl}`, `{maxSize}`, `{enumValues}`, etc.
- Utiliser `{constraints.0}` pour les contraintes automatiques des décorateurs
- Documenter les paramètres nécessaires dans les commentaires
- Tester avec différentes valeurs de paramètres

## Détection de la langue

### Header Accept-Language

Le système détecte automatiquement la langue via le header HTTP `Accept-Language` :

```
Accept-Language: fr-FR,fr;q=0.9,en-US;q=0.8,en;q=0.7
```

**Ordre de priorité** :
1. Langue demandée dans le header (si disponible)
2. Langue de fallback (`FALLBACK_LANGUAGE`) si la langue demandée n'est pas disponible

## Variables d'environnement

### FALLBACK_LANGUAGE

La langue de fallback est définie dans `.env` :

```env
FALLBACK_LANGUAGE=fr
```

Cette langue est utilisée si :
- La langue demandée n'est pas disponible
- Le header `Accept-Language` n'est pas fourni
- Une clé de traduction manque dans la langue demandée

## Script de vérification des traductions

Le projet inclut un script d'audit qui vérifie la cohérence des traductions entre le code et les fichiers JSON.

### Emplacement

```
api/check/translation.mjs
```

### Lancer le script

Depuis le dossier `api/` :

```bash
node check/translation.mjs
```

### Ce que le script vérifie

1. **Clés manquantes en FR** — chaque appel `i18n.t('...')`, `i18nValidationMessage('...')` ou `message: '...'` dans le code a sa clé correspondante dans `src/i18n/fr/*.json`
2. **Clés manquantes en EN** — même vérification pour `src/i18n/en/*.json`
3. **Namespace introuvable** — le namespace (premier segment de la clé, ex: `auth`) correspond à un fichier JSON existant
4. **Clés mortes** — clés présentes dans les JSON mais jamais utilisées dans le code

### Exemple de sortie

```
════════════════════════════════════════════════════════════
       AUDIT DES TRADUCTIONS API  —  FR + EN
════════════════════════════════════════════════════════════

❌  CLÉ ABSENTE DU FICHIER EN  (1)
    "user.verify-account.success"
      /domains/user/controller/user.controller.ts:42

⚠️  CLÉ INUTILISÉE DANS FR (code mort)  (1)
    "auth.login.old-key"

════════════════════════════════════════════════════════════
  RÉSUMÉ
  Namespaces manquants   : 0
  Clés absentes du FR    : 0
  Clés absentes du EN    : 1
  Clés mortes FR         : 1
  Clés mortes EN         : 0
  Total problèmes        : 2
════════════════════════════════════════════════════════════
```

### Quand l'exécuter

**Obligatoirement** après toute ajout ou modification de clés de traduction (nouveaux fichiers JSON, nouvelles clés dans le code). Voir le [Guide IA](./16-agent-ia.md) pour l'intégration dans le workflow.

---

[← Retour au sommaire](./SUMMARY.md)