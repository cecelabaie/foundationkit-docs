# Guards

[← Retour au sommaire](./SUMMARY.md)

Ce document explique le système de guards (gardes) dans l'API, notamment le `AuthGuard` qui protège les routes et vérifie l'authentification.

## Concept de base

Les guards (gardes) sont des composants NestJS qui déterminent si une requête peut être traitée ou non. Ils s'exécutent **avant** les intercepteurs et les pipes, et **après** les middlewares.

### À quoi ça sert ?

- 🔒 **Protéger les routes** : Vérifier l'authentification avant d'exécuter le contrôleur
- ✅ **Autorisation** : Contrôler l'accès en fonction des rôles ou permissions
- 🚫 **Bloquer les requêtes non autorisées** : Retourner une erreur 401 si l'utilisateur n'est pas authentifié

## Flux de traitement d'une requête

```
Client envoie une requête
         ↓
Middlewares (CORS, cookies, etc.)
         ↓
Guards ← NOUS SOMMES ICI
         ↓
Intercepteurs
         ↓
Pipes (validation)
         ↓
Contrôleur (handler)
         ↓
Réponse au client
```

## AuthGuard

Le `AuthGuard` est le guard principal de l'application. Il protège **toutes les routes par défaut**.

**Fichier** : `src/domains/auth/guard/auth.guard.ts`

### Configuration globale

Dans `auth.module.ts`, les guards sont enregistrés **globalement** :

```typescript
@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: AuthGuard,
    },
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
    // ...
  ],
})
export class AuthModule {}
```

**Conséquence** : Toutes les routes nécessitent une authentification **SAUF** celles marquées avec `@Public()`. Le `RolesGuard` s'applique en plus pour les contrôles d'autorisation par rôle.

### Comment il fonctionne

Le guard implémente l'interface `CanActivate` avec la méthode `canActivate()` qui retourne :
- `true` → La requête peut continuer
- `false` ou `throw UnauthorizedException` → La requête est bloquée (401)

```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private readonly reflector: Reflector,
    private readonly authService: AuthService,
    private readonly revokedTokenService: RevokedTokenService,
    private readonly accessTokenService: AccessTokenService,
    private readonly i18n: I18nService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // ... logique de vérification
  }
}
```

### Flux de vérification

```
┌─────────────────────────────────────────────────────────────┐
│                    AuthGuard.canActivate()                  │
└─────────────────────────────────────────────────────────────┘

1. Vérifier si la route est publique (@Public())
   ├─ OUI → return true ✅
   └─ NON → Continuer

2. Extraire l'access token des cookies
   ├─ Token présent → Continuer
   └─ Token absent → throw UnauthorizedException ❌

3. Valider l'access token (JWT)
   ├─ Vérification signature JWT
   ├─ Vérification expiration
   ├─ Vérification user.needToReconnect
   ├─ Valide → Continuer
   └─ Invalide → throw UnauthorizedException ❌

4. Vérifier la liste noire (RevokedToken)
   ├─ Token révoqué → throw UnauthorizedException ❌
   └─ Token OK → return true ✅

5. En cas d'erreur
   └─ Supprimer le cookie access_token
```

### Étape 1 : Vérification route publique

```typescript
async canActivate(context: ExecutionContext): Promise<boolean> {
  // Lire les métadonnées de la route
  const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
    context.getHandler(),  // Métadonnées de la méthode
    context.getClass(),    // Métadonnées du contrôleur
  ]);
  
  if (isPublic) {
    return true;  // ← Route publique, pas de vérification
  }
  
  // ... continuer la vérification
}
```

**Comment marquer une route publique ?**

Les routes publiques utilisent le décorateur `@Public()` intégré dans les décorateurs personnalisés :

```typescript
// Dans auth-login.decorator.ts
export function LoginEndpoint(path: string) {
  return applyDecorators(
    Public(),  // ← Rend la route publique
    Post(path),
    // ...
  );
}
```

**Routes publiques** (pas de vérification) :
- `/auth/login`
- `/auth/logout`
- `/auth/refresh-session`
- `/user/register`
- `/user/verify-account`
- `/reset-password/forgot`
- `/reset-password/validate`
- `/reset-password/update`

**Routes protégées** (vérification obligatoire) :
- `/auth/validate-session`
- `/user/profile`
- `/user/update`
- `/user/update-password`
- `/user/delete-account`
- `/media/private/{mediaId}`

### Étape 2 : Extraction du token

```typescript
const request = context.switchToHttp().getRequest<Request>();
const token = this.extractTokenFromCookie(request);

if (!token) {
  throw new UnauthorizedException(
    this.i18n.t('auth.session.token-invalid-or-expired'),
  );
}
```

**Méthode d'extraction** :

```typescript
private extractTokenFromCookie(request: Request): string | undefined {
  try {
    const token = (request.cookies as Record<'access_token', string>)
      .access_token;
    return token;
  } catch (err) {
    return undefined;
  }
}
```

→ Le token est lu depuis le **cookie `access_token`** (pas depuis les headers)

### Étape 3 : Validation du token

```typescript
try {
  // Valide le JWT et vérifie user.needToReconnect
  await this.accessTokenService.validateAccessToken(token);
  
  // Vérifie la liste noire
  const isRevoked = await this.revokedTokenService.findByToken(token);
  if (isRevoked) {
    throw new UnauthorizedException(
      this.i18n.t('auth.service.token-revoked'),
    );
  }
} catch (err) {
  // En cas d'erreur, supprimer le cookie
  const response = context.switchToHttp().getResponse<Response>();
  this.authService.setAccessTokenCookie(response, '', -1);
  
  throw new UnauthorizedException(
    this.i18n.t('auth.session.token-invalid-or-expired'),
  );
}

return true;  // ← Authentification réussie
```

**Que fait `validateAccessToken()` ?**

1. Vérifie la **signature** du JWT avec `JWT_SECRET`
2. Vérifie que le JWT n'est **pas expiré**
3. Vérifie que l'**utilisateur existe** en base
4. Vérifie que `user.needToReconnect = false`
5. Si toutes les vérifications passent → retourne `{ user, payload }` et le guard attache `user` à `request.user`

> 💡 Pour plus de détails, voir **[session.md](./07-session.md)** - Le flag needToReconnect

### Étape 4 : Suppression du cookie en cas d'erreur

Si la validation échoue, le guard **supprime le cookie** pour forcer le frontend à redemander une authentification :

```typescript
const response = context.switchToHttp().getResponse<Response>();
this.authService.setAccessTokenCookie(response, '', -1);
```

**Pourquoi ?**
- ✅ Évite que le frontend réessaie avec un token invalide
- ✅ Force le refresh de session automatique (via intercepteur)
- ✅ Nettoie les cookies expirés

## Le décorateur @Public()

Le décorateur `@Public()` permet de **contourner** le `AuthGuard` pour rendre une route accessible sans authentification.

**Fichier** : `src/domains/common/decorators/public.decorator.ts`

```typescript
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';

export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

### Comment l'utiliser ?

**Méthode directe** (rarement utilisée) :

```typescript
@Public()  // ← Route publique
@Post('login')
async login() {
  // ...
}
```

**Méthode recommandée** (via décorateur personnalisé) :

```typescript
// Dans un décorateur personnalisé
export function LoginEndpoint(path: string) {
  return applyDecorators(
    Public(),  // ← Intégré dans le décorateur
    Post(path),
    // ...
  );
}
```

```typescript
// Dans le contrôleur
@LoginEndpoint('login')  // ← Public implicitement
async login() {
  // ...
}
```

**Avantages** :
- ✅ Centralise la configuration (Public + Swagger + Throttle)
- ✅ Cohérence dans toute l'application
- ✅ Facilite la maintenance

## RolesGuard

En plus de l'`AuthGuard`, l'application dispose d'un `RolesGuard` qui permet de restreindre l'accès à certains endpoints selon le rôle de l'utilisateur.

**Fichier** : `src/domains/common/guards/roles.guard.ts`

### Configuration

Le guard est enregistré **globalement** dans `AuthModule`, après l'`AuthGuard`. Il s'exécute donc sur toutes les routes.

```typescript
providers: [
  { provide: APP_GUARD, useClass: AuthGuard },
  { provide: APP_GUARD, useClass: RolesGuard },
]
```

### Le décorateur @Roles()

```typescript
// src/domains/common/decorators/roles.decorator.ts
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);
```

**Utilisation sur un endpoint** :

```typescript
@Roles(Role.admin)
@Get('admin-only')
async adminEndpoint() { ... }
```

### Comportement

```
1. Pas de @Roles() sur la route → autorisé pour tous les utilisateurs authentifiés
2. @Roles(Role.admin) → vérifie que user.role contient 'admin'
   ├─ Contient le rôle → return true ✅
   └─ Ne contient pas le rôle → throw ForbiddenException 403 ❌
3. Utilisateur absent de la requête → throw UnauthorizedException 401 ❌
```

### Rôles disponibles

```typescript
enum Role {
  admin
  user
}
```

- **user** : rôle attribué automatiquement à l'inscription
- **admin** : accès étendu (à attribuer manuellement en base)

> 💡 Le champ `role` du modèle `User` est un tableau (`Role[]`). Un utilisateur peut avoir plusieurs rôles simultanément.

---

## Ressources

- **NestJS Guards** : https://docs.nestjs.com/guards
- **ExecutionContext** : https://docs.nestjs.com/fundamentals/execution-context
- **Reflector** : https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata

## Voir aussi

- **[session.md](./07-session.md)** - Système de session et authentification JWT
- **[config.md](./01-config.md)** - Configuration JWT et sécurité
- **[responses.md](./06-responses.md)** - Format des réponses d'erreur

---

[← Retour au sommaire](./SUMMARY.md)
