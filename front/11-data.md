# Récupération et envoi de données

Ce document explique comment gérer les appels API et la récupération de données dans l'application, en utilisant React Query et les hooks générés par Orval.

## Hooks générés par Orval

### Utilisation

Les hooks d'API sont générés automatiquement par Orval à partir des définitions OpenAPI. Ces hooks permettent d'interagir facilement avec l'API backend :

```tsx
// Import du hook généré pour la connexion
import { useAuthControllerLogin } from '@/api/generated/auth/auth';
import { AuthLoginBodyDTO, AuthLoginResponseDTO } from '@/api/generated/schemas';
import { parseAxiosError } from '@/utils/errors';

function LoginComponent() {
  // Récupération du hook de mutation avec son état
  const { 
    mutate: login,
    isLoading,
    isError,
    error 
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
          // Gestion des erreurs avec parseAxiosError
          const errData = parseAxiosError<UnauthorizedResponseDTO>(error, 'Erreur de connexion');
          console.error('Erreur de connexion', errData);
        }
      }
    );
  };
  
  return (
    <button 
      onClick={() => handleLogin({ email: 'user@example.com', password: 'secret' })}
      disabled={isLoading}
    >
      {isLoading ? 'Connexion...' : 'Se connecter'}
    </button>
  );
}
```

**Note** : 
- **Pour les formulaires React Hook Form** : Dans ce projet, on utilise `mutate` avec `handleSubmit` et les callbacks `onSuccess`/`onError`.
- **Pour les cas simples (bouton onClick sans formulaire)** : On peut utiliser `mutateAsync` avec try/catch si on a besoin d'attendre le résultat de la mutation (ex: dans un contexte, un guard, etc.).

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

Le `QueryClient` gère automatiquement le rafraîchissement de session et le retry. Il est configuré avec trois niveaux :

#### QueryCache — erreurs sur les `useQuery`

Sur toute erreur 401 :
1. Si la route est dans `ROUTES_WITHOUT_RETRY` → abandon immédiat (évite une boucle infinie sur `/auth/refresh-session` lui-même)
2. Sinon → appel `POST /auth/refresh-session`
   - Succès (201) : refetch automatique de la query concernée
   - Échec : redirection vers `/` (déconnexion implicite)

#### MutationCache — erreurs sur les `useMutation`

Même logique que le QueryCache, mais au lieu d'un refetch, la mutation est **ré-exécutée** avec les mêmes variables (`mutation.execute(variables)`).

#### defaultOptions — politique de retry

| | Queries | Mutations |
|---|---|---|
| Max tentatives | 1 | 1 |
| Skip si 401 | oui | oui |
| Skip si 429 | oui | oui |
| Skip si 400 | non | **oui** |
| Skip si route sans retry | oui | oui |

Les mutations ne retentent jamais sur une 400 (erreur de validation) car une deuxième tentative donnerait le même résultat.

Les routes exclues du retry sont définies dans `src/constants/constants.ts` :

```tsx
export const ROUTES_WITHOUT_RETRY = [
  API_PATHS.AUTH.VALIDATE_SESSION,
  API_PATHS.AUTH.REFRESH_SESSION,
  API_PATHS.AUTH.LOGIN,
  API_PATHS.AUTH.LOGOUT,
  API_PATHS.USER.REGISTER,
  // ...
];
```

Pour plus d'informations sur la configuration globale, consultez [config.md](./16-config.md).