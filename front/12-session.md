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
4. **Flag de session** - Indicateur simple dans le localStorage qui signale l'état potentiel de connexion

Pour plus de détails sur l'implémentation backend de la session, consultez [api/07-session.md](../api/07-session.md).

## AuthContext

Le `AuthContext` est le cœur du système d'authentification. Il fournit :

- L'état de l'utilisateur courant (`user`)
- Les méthodes d'authentification et de vérification
- Les états de chargement

### Méthodes principales

| Méthode | Description |
|---------|-------------|
| `checkToken` | Vérifie la validité du token actuel via l'API et, en cas d'échec, appelle `tryRefreshToken` |
| `fetchAndSetUser` | Récupère les données de l'utilisateur depuis l'API et met à jour le contexte |
| `tryRefreshToken` | Tente de rafraîchir la session en utilisant le refresh token stocké dans les cookies |
| `disconnect` | Déconnecte l'utilisateur, supprime le flag de session et redirige vers la page d'accueil |

### Cycle de vie de la session

1. **Initialisation** : Au chargement de l'application, `AuthContext` vérifie si un flag de session existe dans le localStorage
2. **Vérification** : Si le flag existe, appel à l'API `/auth/validate-session` pour vérifier la validité du token
3. **Rafraîchissement** : Si le token est invalide mais qu'un refresh token existe, tentative de rafraîchissement
4. **Chargement des données** : Si l'authentification réussit, récupération des données utilisateur

### Flag de session

Le système utilise un simple drapeau dans le localStorage pour indiquer l'état potentiel de connexion :

```typescript
export const FLAG_SESSION = {
  KEY: 'flag_session',
  VALID: 'up',
};
```

Ce flag sert uniquement lors du rafraîchissement de la page pour déterminer si une tentative de validation de session doit être effectuée. Avec ce flag, l'application évite un appel API inutile pour les visiteurs non connectés. **Important** : ce flag n'est pas une preuve d'authentification, mais simplement un indicateur d'état potentiel qui optimise les performances.

## Guards de protection

L'application utilise trois guards pour protéger les différentes parties de l'application :

### AuthGuard

Protège les routes nécessitant une authentification (`/app/(logged)/...`).

- Vérifie que l'utilisateur est connecté
- Redirige vers la page de login si non connecté
- Supporte la vérification des rôles (`requiredRoles`)
- Conserve l'URL originale dans le paramètre `returnTo` pour rediriger après connexion

```tsx
<AuthGuard requiredRoles={['user']}>
  {children}
</AuthGuard>
```

### UnloggedGuard

Protège les routes accessibles uniquement aux utilisateurs non connectés (`/app/(unlogged)/...`).

- Redirige vers la page d'accueil si l'utilisateur est déjà connecté

```tsx
<UnloggedGuard>
  {children}
</UnloggedGuard>
```

### ForgotPasswordGuard

Protège spécifiquement la route de réinitialisation de mot de passe (`/forgot-password/new-password`).

- Vérifie la validité du token dans l'URL
- Redirige vers la page de demande de réinitialisation si le token est invalide
- Affiche des messages d'erreur appropriés selon le type d'erreur

```tsx
<ForgotPasswordGuard>
  {children}
</ForgotPasswordGuard>
```

### Protection des vues

Grâce au système de guards, le contenu protégé n'est jamais visible tant que la vérification d'authentification n'a pas réussi. Pendant la vérification, un composant de chargement (`PageLoader`) est affiché à la place du contenu.

```tsx
// Extrait de AuthGuard
if (isLoading || !user || !hasAccess) {
  return <PageLoader />;
}

return children;
```

> **Important** : Cette protection est implémentée au niveau du client (JavaScript). Bien que les guards empêchent l'affichage des composants protégés dans l'interface utilisateur, le code source des pages reste accessible dans le build et peut être inspecté dans les outils de développement du navigateur. Toutes données sensibles doivent venir de l'API et ne doivent pas être côté client.

## Intégration avec React Query

Le `QueryClient` est configuré pour gérer automatiquement les erreurs d'authentification (401).
Pour plus de détails sur la configuration de React Query, consultez [data.md](./11-data.md).