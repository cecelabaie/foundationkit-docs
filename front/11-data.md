# Récupération et envoi de données

Ce document explique comment gérer les appels API et la récupération de données dans l'application, en utilisant React Query et les hooks générés par Orval.

## Hooks générés par Orval

### Utilisation

Les hooks d'API sont générés automatiquement par Orval à partir des définitions OpenAPI. Ces hooks permettent d'interagir facilement avec l'API backend :

```tsx
// Import du hook généré pour la connexion
import { useAuthControllerLogin } from '@/api/generated/auth/auth';
import { AuthLoginBodyDTO } from '@/api/generated/schemas';
import { displayError } from '@/utils/errors';

function LoginComponent() {
  // Récupération du hook de mutation avec son état
  const {
    mutate: login,
    isPending,
  } = useAuthControllerLogin();

  const handleLogin = (credentials: AuthLoginBodyDTO) => {
    // Utilisation de mutate avec callbacks pour une meilleure gestion des erreurs
    login(
      { data: credentials },
      {
        onSuccess: (response) => {
          // Traitement en cas de succès
          console.log('Connexion réussie', response);
        },
        onError: (error) => {
          displayError(error);
        }
      }
    );
  };
  
  return (
    <button 
      onClick={() => handleLogin({ email: 'user@example.com', password: 'secret' })}
      disabled={isPending}
    >
      {isPending ? 'Connexion...' : 'Se connecter'}
    </button>
  );
}
```

**Note** :
- **Pour les formulaires React Hook Form** : Dans ce projet, on utilise `mutate` avec `handleSubmit` et les callbacks `onSuccess`/`onError`.
- **Pour les guards et useEffect** : Toujours utiliser `mutateAsync` avec `try/catch` dans une fonction `async`. Les callbacks `onSuccess`/`onError` passés à `mutate` sont liés à l'observer du composant — React StrictMode monte/démonte/remonte les composants en développement, ce qui drop silencieusement ces callbacks si la réponse API arrive après le démontage. `mutateAsync` retourne une Promise qui survit au cycle de vie du composant.
- **Pour les cas simples (bouton onClick sans formulaire)** : On peut utiliser `mutateAsync` avec try/catch si on a besoin d'attendre le résultat de la mutation.

Pour l'utilisation avec les formulaires, consultez [formulaires.md](./10-formulaires.md).

## Gestion des erreurs

### parseAxiosError

La fonction utilitaire `parseAxiosError` permet de traiter les erreurs des requêtes API de manière typée :

```tsx
import { parseAxiosError } from '@/utils/errors';

try {
  // Appel API
} catch (error) {
  const errData = parseAxiosError<
    ValidationExceptionResponseDTO | UnauthorizedResponseDTO
  >(error, 'Message d\'erreur par défaut');
  
  // Traitement selon le type d'erreur
  if (errData.statusCode === 400) {
    // Erreur de validation
  }
}
```

Pour plus de détails sur la gestion des erreurs et les autres utilitaires disponibles, consultez [utils.md](./15-utils.md).

## Configuration de React Query

### QueryClient personnalisé

**Fichier :** `src/config/queryClient.ts`

Le `QueryClient` configure la politique de retry des requêtes. La gestion des erreurs 401 est entièrement déléguée à l'interceptor Axios dans `src/config/httpClient.ts`.

#### Gestion des 401 : interceptor Axios dans `httpClient.ts`

La gestion des erreurs 401 se fait via un interceptor `AXIOS_INSTANCE.interceptors.response` défini dans `src/config/httpClient.ts`. Sur toute erreur 401 :

1. Si la route est dans `ROUTES_WITHOUT_RETRY` → abandon immédiat (évite une boucle infinie sur `/auth/refresh-session` lui-même)
2. Si la requête a déjà été rejouée (`_retry`) → abandon immédiat
3. Sinon → appel `refreshToken()`
   - Succès : la requête originale est rejouée automatiquement par Axios
   - Échec : appel du callback `onAuthFailed`, qui efface l'utilisateur en cache (enregistré depuis `AuthContext`)

Le refresh est **dédupliqué** via une `refreshPromise` singleton : si plusieurs requêtes échouent en 401 simultanément, un seul appel `POST /auth/refresh-session` est effectué.

Le callback est enregistré depuis `AuthContext` au montage :

```tsx
registerAuthFailedCallback(() => {
  queryClient.setQueryData(getUserControllerGetProfileQueryKey(), null);
});
```

#### defaultOptions : politique de retry

La politique de retry utilise une **allowlist** : seules les erreurs transitoires sont retentées. Les erreurs déterministes (4xx) ne sont jamais retentées car une deuxième tentative donnerait le même résultat.

```tsx
const RETRYABLE_STATUS = new Set([408, 500, 502, 503, 504]);

const shouldRetry = (failureCount: number, error: unknown): boolean => {
  if (failureCount >= 1) return false;
  const status = getErrorStatus(error);
  return status === undefined || RETRYABLE_STATUS.has(status);
};
```

| | Queries | Mutations |
|---|---|---|
| Max tentatives | 1 | 1 |
| Retry si 4xx (400, 401, 429...) | non | non |
| Retry si 5xx / 408 | oui | oui |
| Retry si statut inconnu | oui | oui |

Les routes exclues de la logique 401 (dans `httpClient.ts`) sont définies dans `src/constants/constants.ts` :

```tsx
export const ROUTES_WITHOUT_RETRY = [
  API_PATHS.AUTH.REFRESH_SESSION,
  API_PATHS.AUTH.LOGIN,
  API_PATHS.RESET_PASSWORD.VALIDATE,
  API_PATHS.RESET_PASSWORD.UPDATE,
  API_PATHS.USER.VERIFY_ACCOUNT,
];
```

Pour plus d'informations sur la configuration globale, consultez [config.md](./16-config.md).