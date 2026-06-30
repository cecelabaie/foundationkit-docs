# Gestion de la session

Ce document explique comment la session utilisateur est gérée dans l'application front-end.

## Architecture stateless

L'application utilise une architecture de session **stateless** basée sur des cookies HTTP-only gérés par le backend. Cette approche offre plusieurs avantages :

- **Sécurité accrue** : Les tokens d'authentification ne sont jamais accessibles au JavaScript
- **Protection contre les attaques XSS** : Les cookies HTTP-only ne peuvent pas être lus par du code malveillant
- **Gestion centralisée** : Le backend gère l'émission, la validation, l'expiration et la révocation des tokens

Côté front-end, la session est orchestrée par plusieurs mécanismes complémentaires :

1. **AuthContext** - Contexte React central qui gère l'état de l'utilisateur et les opérations d'authentification
2. **Guards** - Composants de protection des routes selon l'état d'authentification
3. **QueryClient** - Configuration React Query qui gère le rafraîchissement automatique du token

Pour plus de détails sur l'implémentation backend de la session, consultez [api/07-session.md](../api/07-session.md).

## AuthContext

Le `AuthContext` est le cœur du système d'authentification. Il fournit :

- L'état de l'utilisateur courant (`user`)
- Les états de chargement

### Valeurs exposées

| Valeur | Type | Description |
|--------|------|-------------|
| `user` | `UserProfileDTO \| undefined` | Données de l'utilisateur connecté, `undefined` si non connecté |
| `isLoadingUser` | `boolean` | Vrai pendant le premier chargement du profil |
| `refetchMe` | `function` | Force un rechargement du profil utilisateur depuis l'API |

### Cycle de vie de la session

Au montage, `AuthContext` effectue un unique appel `GET /user/profile` via React Query pour déterminer si l'utilisateur est connecté. Aucune logique de flag localStorage ou de validate-session n'est nécessaire : la requête profil sert directement de vérification d'authentification.

- Si la requête réussit → `user` est alimenté, l'application s'affiche
- Si la requête retourne 401 → l'interceptor Axios dans `httpClient.ts` tente un refresh automatique (voir [data.md](./11-data.md))
  - Refresh réussi → la requête profil est relancée
  - Refresh échoué → `onAuthFailed` est appelé, ce qui place `null` en cache, `user` devient `undefined`

La query profil est configurée avec `retry: false` et `staleTime: Infinity` pour éviter tout appel parasite après un `setQueryData(null)`.

### Enregistrement du callback d'échec d'authentification

`AuthContext` enregistre un callback dans `httpClient` au montage via `registerAuthFailedCallback` (importé depuis `src/config/httpClient.ts`) :

```tsx
registerAuthFailedCallback(() => {
  queryClient.setQueryData(getUserControllerGetProfileQueryKey(), null);
});
```

Ce callback est appelé par l'interceptor Axios de `httpClient.ts` lorsqu'un refresh de token échoue. Il efface les données utilisateur du cache React Query, ce qui déconnecte l'utilisateur sans dépendance circulaire entre `AuthContext` et `httpClient`.

## Guards de protection

L'application utilise quatre guards pour protéger les différentes parties de l'application :

### AuthGuard

Protège les routes nécessitant une authentification (`src/app/(logged)/...`).

- Vérifie que l'utilisateur est connecté
- Redirige vers la page de login si non connecté (avec `?returnTo=` pour revenir après connexion)
- Supporte la vérification des rôles (`requiredRoles`)

```tsx
<AuthGuard requiredRoles={['user']}>
  {children}
</AuthGuard>
```

### UnloggedGuard

Protège les routes accessibles uniquement aux utilisateurs non connectés (`src/app/(unlogged)/...`).

- Redirige vers la page `returnTo` ou `/` si l'utilisateur est déjà connecté

```tsx
<UnloggedGuard>
  {children}
</UnloggedGuard>
```

### ForgotPasswordGuard

Protège la route de réinitialisation de mot de passe (`/forgot-password/new-password`).

- Vérifie la validité du token dans l'URL via `POST /reset-password/validate`
- Redirige vers `/forgot-password` si le token est invalide
- Affiche les erreurs via `displayError`

```tsx
<ForgotPasswordGuard>
  {children}
</ForgotPasswordGuard>
```

### VerifyAccountGuard

Protège la route de vérification de compte (`/verify-account`).

- Lit `userId` et `token` dans les query params
- Appelle `POST /user/verify-account` au montage
- Redirige systématiquement vers `/login` une fois la vérification terminée (succès ou erreur)
- N'affiche jamais de contenu enfant : retourne toujours un `PageLoader`

```tsx
<VerifyAccountGuard />
```

### Protection des vues

Pendant la vérification d'authentification, un composant `PageLoader` est affiché. Le contenu protégé n'est jamais rendu tant que la vérification n'a pas réussi.

> **Important** : Cette protection est implémentée au niveau du client (JavaScript). Le code source des pages reste accessible dans le build. Toutes données sensibles doivent venir de l'API et ne doivent pas être stockées côté client.

## Intégration avec React Query et httpClient

La gestion des erreurs 401 est assurée par l'interceptor Axios dans `src/config/httpClient.ts`. Le `QueryClient` (`src/config/queryClient.ts`) configure uniquement la politique de retry pour les erreurs transitoires (5xx, 408).
Pour plus de détails, consultez [data.md](./11-data.md).