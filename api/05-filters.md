# Filters

[← Retour au sommaire](./SUMMARY.md)

Ce document explique le système de filtres d'exception, notamment le `ValidationExceptionFilter` qui intercepte les erreurs de validation.

> 💡 **Pour comprendre les différents types de réponses HTTP** (200, 400, 404, etc.), consultez **[responses.md](./06-responses.md)**.

## Concept de base

Les filtres d'exception (Exception Filters) interceptent les erreurs levées dans l'application et les transforment en réponses HTTP structurées. Ils permettent de :

- 🎯 **Standardiser les réponses d'erreur** : Format cohérent pour toutes les erreurs
- 🌍 **Internationaliser les messages** : Traduire les messages via i18n si besoin
- 📋 **Détailler les violations** : Informations précises sur les erreurs de validation
- 📌 **Mapper automatiquement l'erreur à un champ de formulaire** : Si c'est une erreur qui vient d'un formulaire, elle est automatiquement mappée à un champ et affichée sous celui-ci.
- 🔍 **Faciliter le débogage** : Réponses claires et exploitables côté frontend

## Comment ça marche ? (Vue simplifiée)

```
Client envoie une requête avec données invalides
         ↓
class-validator détecte les erreurs
         ↓
ValidationExceptionFilter intercepte les erreurs
         ↓
Traduit les messages (i18n) si besoin
         ↓
Formate la réponse JSON
         ↓
Retourne au client avec code 400 (Bad Request)
```

## ValidationExceptionFilter

Le `ValidationExceptionFilter` est le filtre principal qui intercepte les erreurs de validation des DTOs.

**Fichier** : `src/filters/validationException.filter.ts`

C'est le composant central qui transforme les erreurs de validation en réponses HTTP propres.

### À quoi ça sert ?

Imaginons qu'un utilisateur envoie :

```json
{
  "email": "pas-un-email",
  "password": "123"
}
```

**Avec le filtre**, l'API retourne :

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "error": "Bad Request",
  "violations": [
    {
      "field": "email",
      "message": "L'email doit être valide"
    },
    {
      "field": "password",
      "message": "Le mot de passe doit contenir au moins 8 caractères"
    }
  ]
}
```

### Comment il fonctionne ?

#### Étape 1 : Interception

```typescript
@Catch(I18nValidationException)
export class ValidationExceptionFilter implements ExceptionFilter {
```

Le décorateur `@Catch()` dit : "Attrape toutes les erreurs de type `I18nValidationException`".

#### Étape 2 : Récupération du contexte

```typescript
const ctx = host.switchToHttp();
const response = ctx.getResponse<Response>();
const i18n = I18nContext.current();
```

On récupère :
- La **réponse HTTP** pour pouvoir renvoyer le JSON
- Le **contexte i18n** pour traduire les messages si besoin

#### Étape 3 : Transformation des erreurs

Le filtre récupère pour chaque erreur le premier message des contraintes. Si le message contient `|`, il est interprété comme `clé_i18n|argsJson` : la clé et les arguments JSON sont extraits et la traduction est appelée avec `i18n.t(key, { args })`.

```typescript
const constraints = error.constraints ?? {};
const message: string = Object.values(constraints)[0] ?? '';

if (message.includes('|')) {
  const [key, argsJson] = message.split('|');
  try {
    const args = JSON.parse(argsJson) as Record<string, unknown>;
    return { field: error.property, message: i18n?.t(key, { args }) ?? message };
  } catch {
    return { field: error.property, message: i18n?.t(key) ?? message };
  }
}
return { field: error.property, message: i18n?.t(message) ?? message };
```

Pour chaque erreur :
- On prend le **nom du champ** (`email`, `password`, etc.)
- On **traduit** le message d'erreur via i18n (avec support du format `clé|{"arg": "value"}` pour les arguments)
- On crée un objet `{ field, message }`

#### Étape 4 : Envoi de la réponse

Le filtre envoie une réponse **400 Bad Request** avec un JSON au format suivant :

- **statusCode** : 400
- **message** : `'Validation failed'`
- **error** : `'Bad Request'`
- **violations** : tableau de `{ field, message }` (une entrée par champ en erreur)

Ce format est le même que celui utilisé ailleurs pour les erreurs de validation (voir [responses.md](./06-responses.md)), ce qui permet au frontend de mapper les erreurs aux champs du formulaire de façon cohérente.

### Support des traductions (optionnel mais conseillé)

Le filtre supporte les traductions i18n, mais ce n'est **pas obligatoire**. Tu peux utiliser des messages en dur, mais les traductions sont **fortement recommandées** pour une application multilingue et pour le scaling.

**1. Sans traduction (déconseillé)**
```typescript
"L'email est requis"
```
→ Message direct : "L'email est requis"

**2. Avec traduction simple (recommandé)**
```typescript
"user.email.required"
```
→ Le filtre traduit via i18n : "L'email est requis"

**3. Avec traduction + arguments**
```typescript
"user.password.minLength|{\"min\":8}"
```
→ Le filtre sépare la clé et les arguments
→ Traduit : "Le mot de passe doit contenir au moins 8 caractères"

**Exemple de fichier de traduction** (`i18n/fr/user.json`) :
```json
{
  "email": {
    "required": "L'email est requis"
  },
  "password": {
    "minLength": "Le mot de passe doit contenir au moins {$constraint0} caractères"
  }
}
```

### Configuration globale

Dans `main.ts`, on enregistre le filtre pour toute l'application :

```typescript
app.useGlobalFilters(new ValidationExceptionFilter());
```

Ainsi, **toutes** les erreurs de validation sont automatiquement gérées.

## Exemple de réponse

Quand le filtre intercepte une erreur, il retourne une réponse standardisée :

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "error": "Bad Request",
  "violations": [
    {
      "field": "email",
      "message": "L'email doit être valide"
    },
    {
      "field": "password",
      "message": "Le mot de passe doit contenir au moins 8 caractères"
    }
  ]
}
```

> 💡 **Pour plus de détails sur les réponses**, voir **[responses.md](./06-responses.md)**.


## Comment créer une validation ?

### Étape 1 : Définir un DTO avec les règles

```typescript
import { IsEmail, IsNotEmpty, MinLength } from 'class-validator';

export class LoginDto {
  @IsEmail()                    // Doit être un email valide
  @IsNotEmpty()                 // Ne peut pas être vide
  email: string;

  @MinLength(8)                 // Minimum 8 caractères
  @IsNotEmpty()
  password: string;
}
```

### Étape 2 : Utiliser le DTO dans un controller

```typescript
@Post('login')
async login(@Body() loginDto: LoginDto) {
  // Si on arrive ici, c'est que loginDto est valide !
  return this.authService.login(loginDto);
}
```

### Étape 3 : C'est tout !

Le reste est automatique :

1. **NestJS** valide automatiquement le DTO
2. Si c'est invalide → **ValidationExceptionFilter** attrape l'erreur
3. Le filtre **traduit** les messages
4. Le filtre **retourne** une réponse 400 formatée

```
Client envoie { email: "invalid", password: "123" }
                    ↓
NestJS valide avec class-validator
                    ↓
❌ Erreurs détectées
                    ↓
ValidationExceptionFilter
                    ↓
Réponse 400 avec violations détaillées
                    ↓
Frontend affiche les erreurs à l'utilisateur
```

## Validation personnalisée (avancé)

Parfois, tu as besoin de validations qui ne peuvent pas être faites avec des décorateurs.

### Exemple : Vérifier que deux champs correspondent

```typescript
@Post('register')
async register(@Body() dto: RegisterDto) {
  // Validation spécifique : password === confirmPassword
  if (dto.password !== dto.confirmPassword) {
    throw new BadRequestException(
      ResponseUtil.validationError([
        { 
          field: 'password', 
          message: this.i18n.t('user.password.notMatch') 
        },
        { 
          field: 'confirmPassword', 
          message: this.i18n.t('user.confirmPassword.notMatch') 
        }
      ])
    );
  }
  
  // Continuer si valide
  return this.userService.register(dto);
}
```

### Exemple : Vérifier l'unicité en base de données

```typescript
@Post('register')
async register(@Body() dto: RegisterDto) {
  // Vérifier si l'email existe déjà
  const existingUser = await this.userService.findByEmail(dto.email);
  
  if (existingUser) {
    throw new BadRequestException(
      ResponseUtil.validationError([
        { 
          field: 'email', 
          message: this.i18n.t('user.email.alreadyExists', {
            args: { email: dto.email }
          })
        }
      ])
    );
  }
  
  // Créer l'utilisateur
  return this.userService.create(dto);
}
```

> 💡 **Pour plus de détails sur les réponses**, voir **[responses.md](./06-responses.md)**.

## Traductions des messages

Les messages d'erreur sont stockés dans des fichiers JSON par langue et les clés sont **définies directement dans les DTOs**.

### Fichiers de traduction

**Fichier** : `src/i18n/fr/user.json`

```json
{
  "email": {
    "required": "L'email est requis",
    "invalid": "L'email n'est pas valide",
    "alreadyExists": "L'email {email} est déjà utilisé"
  },
  "password": {
    "required": "Le mot de passe est requis",
    "minLength": "Le mot de passe doit contenir au moins {$constraint0} caractères",
    "notMatch": "Les mots de passe ne correspondent pas"
  }
}
```

### Utilisation dans les DTOs

**Les clés de traduction se définissent dans les DTOs** :

```typescript
import { IsEmail, IsNotEmpty, MinLength } from 'class-validator';
import { i18nValidationMessage } from 'nestjs-i18n';

export class RegisterDto {
  @IsEmail({}, { message: 'user.email.invalid' })
  @IsNotEmpty({ message: 'user.email.required' })
  email: string;

  @MinLength(8, { 
    message: i18nValidationMessage('user.password.minLength')  // ← Avec i18nValidationMessage
  })
  password: string;
}
```

**Le filtre traduit automatiquement** :
1. Le DTO détecte l'erreur : `@MinLength(8)` échoue
2. Le message est : `"user.password.minLength"` (la clé)
3. Le `ValidationExceptionFilter` traduit via i18n : `"Le mot de passe doit contenir au moins 8 caractères"`

### Différence entre `message: 'clé'` et `i18nValidationMessage('clé')`

`i18nValidationMessage()` permet de **récupérer automatiquement les paramètres du décorateur** dans la traduction.

#### Exemple : @MinLength

**❌ Sans `i18nValidationMessage`** :
```typescript
@MinLength(8, { message: 'user.password.minLength' })
```
```json
{
  "password": {
    "minLength": "Le mot de passe doit contenir au moins 8 caractères"
  }
}
```
**Problème** : Si tu changes `@MinLength(8)` en `@MinLength(10)`, tu dois **aussi modifier manuellement** la traduction.

**✅ Avec `i18nValidationMessage`** :
```typescript
@MinLength(8, { message: i18nValidationMessage('user.password.minLength') })
```
```json
{
  "password": {
    "minLength": "Le mot de passe doit contenir au moins {$constraint0} caractères"
  }
}
```
**Avantage** : Le `8` de `@MinLength(8)` est injecté automatiquement dans `{$constraint0}`. Si tu changes la valeur, la traduction s'adapte automatiquement !

#### Exemple : @IsEnum

**✅ Avec `i18nValidationMessage` et arguments personnalisés** :
```typescript
@IsEnum(Gender, {
  message: i18nValidationMessage('user.register.validation.gender.isEnum', {
    enumValues: Object.values(Gender).join(', ')
  })
})
gender: Gender;
```
```json
{
  "gender": {
    "isEnum": "Le genre doit être une valeur valide. Valeurs possibles : {enumValues}"
  }
}
```
→ Affiche : `"Le genre doit être une valeur valide. Valeurs possibles : MALE, FEMALE"`

### Règle simple

| Décorateur | As-tu besoin des paramètres dans la traduction ? | Utilise `i18nValidationMessage` ? |
|------------|--------------------------------------------------|-----------------------------------|
| `@IsString()` | Non (pas de paramètre) | ❌ Non : `message: 'clé'` |
| `@IsEmail()` | Non (pas de paramètre) | ❌ Non : `message: 'clé'` |
| `@IsNotEmpty()` | Non (pas de paramètre) | ❌ Non : `message: 'clé'` |
| `@MinLength(8)` | **Oui** (afficher "8") | ✅ Oui : `message: i18nValidationMessage('clé')` |
| `@MaxLength(50)` | **Oui** (afficher "50") | ✅ Oui : `message: i18nValidationMessage('clé')` |
| `@Min(18)` | **Oui** (afficher "18") | ✅ Oui : `message: i18nValidationMessage('clé')` |
| `@IsEnum(Gender)` | **Oui** (lister les valeurs) | ✅ Oui : `message: i18nValidationMessage('clé', {...})` |

**En résumé** : Utilise `i18nValidationMessage()` **uniquement si tu veux utiliser les valeurs du décorateur dans ton message de traduction** (pour rendre le message dynamique).

### Résumé

| Contexte | Où mettre les clés | Comment ça marche |
|----------|-------------------|-------------------|
| **DTO (class-validator)** | Dans les décorateurs `{ message: 'clé' }` | Le filtre traduit automatiquement |
| **Validation personnalisée** | Avec `this.i18n.t('clé')` | Tu traduis manuellement avant de lancer l'exception |

## Voir aussi

- **[responses.md](./06-responses.md)** - Les différents types de réponses HTTP
- **[config.md](./01-config.md)** - Configuration de l'API

## Ressources externes

- **NestJS Exception Filters** : https://docs.nestjs.com/exception-filters
- **Class Validator** : https://github.com/typestack/class-validator
- **nestjs-i18n** : https://nestjs-i18n.com/

---

[← Retour au sommaire](./SUMMARY.md)
