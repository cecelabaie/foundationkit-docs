# Gestion de la session

[← Retour au sommaire](./SUMMARY.md)

Ce document explique le système complet de gestion de session côté API, incluant l'authentification JWT, les tokens d'accès et de rafraîchissement, et la sécurité des sessions.

## Vue d'ensemble

Le système de session utilise une architecture JWT (JSON Web Token) avec deux types de tokens :

- 🔑 **Access Token** : Token JWT courte durée (15 minutes) stocké en cookie HTTP-only
- 🔄 **Refresh Token** : Token aléatoire longue durée (24h) stocké en base de données et en cookie HTTP-only
- 🛡️ **Revoked Tokens** : Liste des tokens révoqués pour invalider les sessions
- 🔒 **needToReconnect** : Flag pour forcer la reconnexion (ex: après reset de mot de passe)

### Avantages de cette architecture

- ✅ **Sécurité** : Cookies HTTP-only protègent contre XSS, tokens hashés en base
- ✅ **Performance** : Access token en JWT (pas de requête DB), refresh token en base pour contrôle total
- ✅ **Révocation** : Possibilité d'invalider des sessions immédiatement
- ✅ **Multi-appareils** : Un utilisateur peut se connecter sur plusieurs appareils simultanément
- ✅ **Traçabilité** : IP et user-agent stockés pour chaque session

## Architecture du système

```
┌─────────────────────────────────────────────────────────────────────┐
│                          SYSTÈME DE SESSION                         │
└─────────────────────────────────────────────────────────────────────┘

1. LOGIN (/auth/login)
   ├─ Vérification email/password
   ├─ Génération Access Token (JWT 15min)
   ├─ Génération Refresh Token (aléatoire 24h, stocké en DB)
   └─ Envoi des tokens en cookies HTTP-only

2. REQUÊTE AUTHENTIFIÉE
   ├─ AuthGuard extrait l'access token du cookie
   ├─ Validation du JWT (signature, expiration)
   ├─ Vérification que le token n'est pas révoqué
   ├─ Vérification que user.needToReconnect = false
   └─ Autorisation ✅

3. REFRESH SESSION (/auth/refresh-session)
   ├─ Lecture de l'ancien access token + refresh token depuis cookies
   ├─ Révocation de l'ancien access token (si encore valide)
   ├─ Vérification du refresh token en DB (non révoqué, non expiré)
   ├─ Révocation de l'ancien refresh token
   ├─ Génération d'un nouveau Access Token + Refresh Token
   └─ Envoi des nouveaux tokens en cookies

4. LOGOUT (/auth/logout)
   ├─ Révocation de l'access token (ajout à RevokedToken)
   ├─ Révocation du refresh token (revokedAt en DB)
   └─ Suppression des cookies

5. NEED TO RECONNECT
   ├─ Reset de mot de passe → needToReconnect = true
   ├─ À la prochaine validation d'access token → révocation + erreur 401
   ├─ Lors du prochain refresh de session → vérification du flag + erreur 401
   └─ L'utilisateur est forcé de se reconnecter avec email/password
```

## Le flag needToReconnect

Le flag `needToReconnect` permet de forcer un utilisateur à se reconnecter sur **tous ses appareils**.

### Quand est-il activé ?

**1. Reset de mot de passe (mot de passe oublié)**

**Fichier** : `src/domains/reset-password/controller/reset-password.controller.ts`

**Pourquoi ?** Si un attaquant a volé une session avant le reset, on veut l'invalider.

**Que se passe-t-il ensuite ?**

Lorsque `needToReconnect = true` :

1. **À la prochaine requête authentifiée** (avec access token) :
   - Le `AuthGuard` valide l'access token
   - Le système détecte `user.needToReconnect = true`
   - L'access token est **immédiatement révoqué** et ajouté à la liste noire
   - Erreur 401 : "Reconnexion requise"

2. **Lors du prochain refresh de session** :
   - Le frontend tente de rafraîchir l'access token avec le refresh token
   - Le système vérifie le refresh token en base
   - Le système détecte `user.needToReconnect = true`
   - Le refresh est **refusé**
   - Erreur 401 : "Reconnexion requise"

3. **L'utilisateur doit se reconnecter** :
   - Le frontend reçoit l'erreur 401
   - Redirection vers la page de login
   - L'utilisateur saisit email + mot de passe
   - Lors du login réussi → `needToReconnect` est remis à `false`
   - Nouvelles sessions créées sur tous les appareils

→ **Toutes les sessions actives sont invalidées**, l'utilisateur doit se reconnecter partout.

### Comment est vérifié needToReconnect ?

**1. Lors de la validation d'un access token** :

```typescript
async validateAccessToken(token: string) {
  const payload = await this.jwtService.verifyAsync(token);
  const user = await this.userService.findById(payload.sub);
  
  // Si needToReconnect = true, on révoque le token immédiatement
  if (user?.needToReconnect) {
    // expirationDate en ms (payload.exp est en secondes)
    await this.revokedTokenService.create(token, payload.exp * 1000, payload.sub);
    throw new UnauthorizedException('Reconnexion requise');
  }

  return { user, payload };
}
```

**2. Lors du refresh de session** :

```typescript
async refreshSession() {
  // ...
  const { refreshTokenNotRevoked, user } =
    await this.refreshTokenService.findNotRevokedByToken(oldRefreshToken);

  // Si needToReconnect = true, on refuse le refresh
  if (user.needToReconnect) {
    throw new UnauthorizedException('Reconnexion requise');
  }
  // ...
}
```

**3. Lors du login** :

```typescript
async signIn(email: string, password: string) {
  const user = await this.usersService.findByLoginCredentials(email, password);
  
  // Créer les tokens...

  // Réinitialiser le flag après login réussi
  if (user.needToReconnect) {
    await this.usersService.setNeedToReconnect(user.id, false);
  }

  return { accessToken, refreshToken };
}
```

## Connexion multi-appareils

### Comment ça fonctionne ?

✅ **Un utilisateur peut se connecter sur plusieurs appareils simultanément.**

Chaque login crée :
- Un nouvel **access token** (unique)
- Un nouveau **refresh token** en base de données

Les sessions sont **indépendantes** : se déconnecter d'un appareil ne déconnecte pas les autres.

### Traçabilité des sessions

Chaque refresh token stocke :

```typescript
{
  ip: '192.168.1.1',               // IP de l'utilisateur
  userAgent: 'Mozilla/5.0...',     // Navigateur/OS
}
```

### Comment lister les sessions actives ? (pas encore implémenté)

```typescript
async getActiveSessions(userId: string) {
  const sessions = await this.prisma.refreshToken.findMany({
    where: {
      userId,
      revokedAt: null,
      expiresAt: { gt: new Date() },
    },
    orderBy: { createdAt: 'desc' },
  });

  return sessions.map(s => ({
    id: s.id,
    ip: s.ip,
    userAgent: s.userAgent,
    createdAt: s.createdAt,
    expiresAt: s.expiresAt,
  }));
}
```

### Comment déconnecter un appareil spécifique ? (pas encore implémenté)

```typescript
async revokeSession(userId: string, sessionId: string) {
  const session = await this.prisma.refreshToken.findFirst({
    where: { id: sessionId, userId },
  });

  if (session) {
    await this.refreshTokenService.revoke(session);
  }
}
```

## Sécurité

### 1. Protection contre XSS (Cross-Site Scripting)

✅ **Cookies HTTP-only** : Les tokens ne sont pas accessibles en JavaScript

```typescript
res.cookie('access_token', token, {
  httpOnly: true,  // ← Le JS ne peut pas lire le cookie
  // ...
});
```

→ Même si un attaquant injecte du code JS, il ne peut pas voler les tokens.

### 2. Protection contre CSRF (Cross-Site Request Forgery)

✅ **SameSite=Strict** : Les cookies ne sont envoyés que sur le même domaine

```typescript
res.cookie('access_token', token, {
  sameSite: 'strict',  // ← Cookie envoyé uniquement depuis APP_URL
  // ...
});
```

→ Un site malveillant ne peut pas faire de requêtes authentifiées à votre API.

### 3. Protection contre le vol de base de données

✅ **Tokens hashés en SHA-256** : Si la DB est compromise, les tokens ne sont pas lisibles

**Fichier** : `src/domains/common/services/utils.service.ts`

```typescript
hashToken(token: string) {
  return createHash('sha256').update(token).digest('hex');
}
```

| Token | Stockage en base | Envoyé au client |
|---|---|---|
| Refresh token | SHA-256 du token aléatoire | Token aléatoire brut (cookie) |
| Reset password token | SHA-256 du token aléatoire | Token aléatoire brut (email) |
| Access token révoqué | SHA-256 du JWT | — |

> 💡 L'access token révoqué est hashé en SHA-256 avant stockage. Lors de la vérification, le token reçu est hashé puis comparé via `findUnique` sur le hash en base.

→ Un attaquant qui vole la DB ne peut pas utiliser les refresh tokens, les reset password tokens ni les access tokens révoqués.

### 4. HTTPS en production

✅ **Secure flag** : Les cookies ne sont transmis qu'en HTTPS

```typescript
res.cookie('access_token', token, {
  secure: NODE_ENV === 'production',  // ← HTTPS uniquement en prod
  // ...
});
```

### 5. Révocation des tokens

Le système permet de révoquer les tokens avant leur expiration naturelle, ce qui permet d'invalider immédiatement une session.

#### Quand les tokens sont-ils révoqués ?

**Access Token (JWT)** :

1. **Lors du logout** :
   ```typescript
   // L'access token est ajouté à la liste noire
   await this.authService.revokeAccessToken(accessToken, userId, expiration);
   ```

2. **Lors du refresh de session** :
   ```typescript
   // L'ancien access token est révoqué (s'il est encore valide)
   try {
     const { payload } = await this.accessTokenService.validateAccessToken(oldAccessToken);
     await this.authService.revokeAccessToken(oldAccessToken, payload.sub, payload.exp);
   } catch (err) {
     // Si déjà expiré, c'est OK
   }
   ```

3. **Quand needToReconnect = true** :
   ```typescript
   // Lors de la validation, si user.needToReconnect = true
   // expiration : date d'expiration du JWT en millisecondes
   await this.revokedTokenService.create(token, expiration, userId);
   throw new UnauthorizedException('Reconnexion requise');
   ```

**Refresh Token** :

1. **Lors du logout** :
   ```typescript
   // Le refresh token est marqué comme révoqué en base
   await this.refreshTokenService.revoke(refreshToken);
   // → revokedAt = Date actuelle
   ```

2. **Lors du refresh de session** :
   ```typescript
   // L'ancien refresh token est révoqué (rotation des tokens)
   await this.refreshTokenService.revoke(oldRefreshToken);
   // Un nouveau refresh token est créé
   ```

#### Comment fonctionne la révocation ?

**Pour les Access Tokens** :

✅ **Liste noire (RevokedToken)** : Les JWT révoqués sont stockés en base de données

```typescript
// Le AuthGuard vérifie TOUJOURS la liste noire
const isRevoked = await this.revokedTokenService.findByToken(token);
if (isRevoked) {
  throw new UnauthorizedException('Token révoqué');
}
```

**Structure d'un token révoqué** :
```typescript
{
  token: "hash_sha256_du_jwt",      // JWT hashé en SHA-256 avant stockage
  userId: "user-id",
  revokedAt: "2024-01-15T10:30:00Z",
  expirationDate: "2024-01-15T10:45:00Z"  // Date d'expiration du JWT
}
```

→ Une fois expiré naturellement, le token peut être supprimé de la liste (nettoyage automatique à prévoir).

**Pour les Refresh Tokens** :

✅ **Flag revokedAt** : Le refresh token est marqué comme révoqué directement en base

```typescript
// Avant révocation
{
  hashedToken: "hash_sha256",
  revokedAt: null,  // ← null = token valide
  expiresAt: "2024-01-16T10:30:00Z"
}

// Après révocation
{
  hashedToken: "hash_sha256",
  revokedAt: "2024-01-15T10:30:00Z",  // ← Date de révocation
  expiresAt: "2024-01-16T10:30:00Z"
}
```

→ Lors du refresh, le système vérifie que `revokedAt === null`.

#### Comportement spécifique : reset-password/update

Lors de la finalisation d'un reset de mot de passe, le token est à la fois **révoqué** et **marqué comme utilisé** simultanément :

```typescript
await resetPasswordService.revoke(resetToken);  // revoked = true
await resetPasswordService.use(resetToken);      // used = true
```

> ℹ️ Ce double marquage est intentionnel : il garantit qu'un token ne peut être réutilisé ni via le flux normal (`used`) ni via une tentative de révocation contournée (`revoked`).

#### Pourquoi révoquer les tokens ?

✅ **Logout** : Invalider immédiatement la session  
✅ **Refresh** : Rotation des tokens pour limiter la fenêtre d'exploitation  
✅ **Reset de mot de passe** : Forcer la reconnexion sur tous les appareils  
✅ **Suspension de compte** : Déconnecter l'utilisateur immédiatement  
✅ **Vol de session détecté** : Révoquer les sessions compromises

## Ressources

- **NestJS Authentication** : https://docs.nestjs.com/security/authentication
- **NestJS Guards** : https://docs.nestjs.com/guards
- **Cookie Security** : https://owasp.org/www-community/controls/SecureCookieAttribute
- **Refresh Token Rotation** : https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation

## Voir aussi

- **[guards.md](./08-guards.md)** - Détails sur le AuthGuard et les guards personnalisés
- **[config.md](./01-config.md)** - Configuration JWT et variables d'environnement
- **[responses.md](./06-responses.md)** - Format des réponses d'erreur

---

[← Retour au sommaire](./SUMMARY.md)
