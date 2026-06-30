# Génération de code API

Le projet utilise [Orval](https://orval.dev/) pour générer automatiquement :

1. Des types TypeScript à partir du schéma OpenAPI de l'API
2. Des hooks React Query pour interagir avec l'API
3. Des schémas de validation Zod pour les requêtes

Cette approche crée un système de validation cohérent de bout en bout :
- Les contraintes de validation définies dans l'API (via des décorateurs comme `@IsEmail()`, `@MinLength()`) sont automatiquement converties en schémas Zod dans le frontend
- Les types et validations sont toujours synchronisés entre le backend et le frontend
- Les formulaires frontend utilisent exactement les mêmes règles de validation que l'API
- Les messages d'erreur sont cohérents et localisés

> **Important** : Le code généré dans `src/api/generated` ne doit jamais être modifié manuellement. Tous les changements seraient écrasés lors de la prochaine génération. Cependant, vous pouvez corriger les erreurs de linting avec la commande `npm run lint:generated:fix`.

## Configuration Orval

La configuration se trouve dans `orval.config.ts` à la racine du projet :

### Points clés de la configuration

- **SWAGGER_TOKEN** : Token d'accès à la documentation Swagger (protégée côté API). Requis : une erreur est levée au démarrage si absent.
- **SWAGGER_JSON** : URL vers le fichier OpenAPI, incluant le token : `{API_URL}/api-json?token={SWAGGER_TOKEN}`. Chargé depuis le premier fichier trouvé dans l'ordre : `.env.production`, `.env.local`, `.env`
- **mode: 'tags-split'** : Génère des fichiers séparés par tags OpenAPI
- **client: 'react-query'** : Génère des hooks React Query
- **mutator** : Utilise le client HTTP `src/config/httpClient.ts` (Axios + interceptor 401 + refresh token)
- **apiZodSchemas** : Configuration pour générer les schémas Zod

## Structure des fichiers générés

### Dossier `src/api/generated`

```
src/api/generated/
├── auth/                  # Hooks React Query pour l'authentification
│   └── auth.ts
├── user/                  # Hooks React Query pour les utilisateurs
│   └── user.ts
├── reset-password/        # Hooks React Query pour la réinitialisation de mot de passe
│   └── reset-password.ts
├── media/                 # Hooks React Query pour les médias
│   └── media.ts
├── schemas/               # Types TypeScript pour toutes les entités
│   ├── authLoginBodyDTO.ts
│   ├── userProfileDTO.ts
│   └── ...
└── zod/                   # Schémas de validation Zod
    ├── auth/
    ├── user/
    └── ...
```

## Client HTTP personnalisé

Le projet utilise un client HTTP personnalisé défini dans `src/config/httpClient.ts`. Il est utilisé par Orval comme mutator pour toutes les requêtes générées.

Responsabilités :
- Configure `AXIOS_INSTANCE` avec l'URL de base (`SERVER_URL`) et l'envoi des cookies (`withCredentials`)
- Interceptor 401 : tente un refresh de session, rejoue la requête en cas de succès, appelle `onAuthFailed` en cas d'échec — le refresh est dédupliqué via une `refreshPromise` singleton
- Expose `registerAuthFailedCallback` pour que `AuthContext` puisse réagir à un échec d'authentification
- Supporte l'annulation des requêtes React Query via `CancelToken`

Pour les détails sur la gestion des 401 et le refresh, voir [data.md](./11-data.md).

## Types générés

### Types TypeScript

Pour chaque requête et réponse de l'API, les types TypeScript sont générés dans le dossier `schemas` :

```typescript
// src/api/generated/schemas/authLoginBodyDTO.ts
export interface AuthLoginBodyDTO {
  email: string;
  /** @minLength 1 */
  password: string;
}
```

Ces types représentent :
- Les corps de requêtes (body) envoyés à l'API
- Les réponses reçues de l'API (Succès, erreurs)
- Les structures de données communes

### Schémas Zod

#### Origine des schémas Zod

Les schémas Zod sont générés à partir du fichier OpenAPI (Swagger) de l'API backend. Le processus fonctionne comme suit :

1. NestJS génère automatiquement une documentation OpenAPI (Swagger) à partir des DTOS et des décorateurs

2. Orval lit cette documentation OpenAPI et convertit ces contraintes en validations Zod équivalentes
   - format: email → `zod.email()`
   - minLength: 1 → `zod.string().min(1)`

#### Schémas générés

Pour chaque endpoint de l'API, Orval génère des schémas Zod correspondants dans le dossier `src/api/generated/zod/` :

```typescript
// src/api/generated/zod/auth/auth.ts
export const AuthControllerLoginBody = zod.object({
  email: zod.email(),           // Contrainte email du Swagger
  password: zod.string().min(1), // Contrainte minLength du Swagger
});
```

Ces schémas Zod sont utilisés pour :
- Valider les données de formulaire côté client
- Typer fortement les données avec TypeScript via `z.infer<typeof schema>`
- Générer des messages d'erreur localisés

## Hooks React Query

Pour chaque endpoint de l'API, un hook React Query est généré :

```typescript
// src/api/generated/auth/auth.ts (simplifié)
export const useAuthControllerLogin = (
  options?: UseMutationOptions<
    AuthLoginResponseDTO,
    Error | ValidationExceptionResponseDTO | UnauthorizedResponseDTO | TooManyRequestResponseDTO,
    AuthControllerLoginVariables
  >
) => {
  return useMutation<
    AuthLoginResponseDTO,
    Error | ValidationExceptionResponseDTO | UnauthorizedResponseDTO | TooManyRequestResponseDTO,
    AuthControllerLoginVariables
  >((data) => authControllerLoginMutator({ url: `/auth/login`, method: 'post', data: data.data }), options);
};
```

Ces hooks offrent :
- Gestion automatique du chargement, des erreurs et du cache
- Typage fort pour les paramètres et les réponses
- Support pour les options de React Query (invalidation, retry, etc.)

### Upload de fichiers et multipart/form-data

⚠️ **Important** : Si une route de l'API contient un upload de fichier et est correctement configurée côté API avec les décorateurs Swagger appropriés (`@ApiConsumes('multipart/form-data')` et `@ApiBody`), Orval générera automatiquement le hook correspondant en version `multipart/form-data`. L'utilisation reste identique.

## Utilisation dans les composants

### Exemple avec un formulaire de connexion

Voici comment les éléments générés sont utilisés dans un composant réel :

```typescript
// src/sections/login/login-form.tsx
import { useAuthControllerLogin } from '@/api/generated/auth/auth';
import { AuthControllerLoginBody } from '@/api/generated/zod/auth/auth';
import { Form } from '@/components/form/form';
import { RhfEmailInput, RhfPasswordInput } from '@/components/form/inputs';
import { useZodI18n } from '@/hooks/useZodI18n';
import { handleServerErrors } from '@/utils/errors';
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod';

export default function LoginForm() {
  // Hook React Query généré automatiquement par Orval
  const { mutate: mutateLogin } = useAuthControllerLogin();

  // Schéma Zod généré automatiquement par Orval à partir des décorateurs du backend
  const zloginSchema = AuthControllerLoginBody;

  // Type inféré du schéma Zod
  type LoginFormInputs = z.infer<typeof zloginSchema>;

  // Configuration React Hook Form avec Zod
  const methods = useForm<LoginFormInputs>({
    resolver: zodResolver(zloginSchema),
    defaultValues: {
      email: '',
      password: '',
    },
  });

  // Obligatoire : re-valide les erreurs Zod au changement de langue
  useZodI18n(methods);

  const { handleSubmit, setError } = methods;

  // Soumission du formulaire avec handleSubmit
  const onSubmit = handleSubmit((input) => {
    // Utilisation du hook généré pour appeler l'API avec mutate
    mutateLogin(
      { data: input },
      {
        onSuccess: () => {
          // Traitement en cas de succès...
        },
        onError: (error) => {
          // LOGIN est dans ROUTES_WITHOUT_RETRY : 401 = mauvais identifiants
          handleServerErrors(error, setError, { includeUnauthorizedAlert: true });
        },
      }
    );
  });

  return (
    <Form methods={methods} onSubmit={onSubmit}>
      {/* Champs du formulaire... */}
    </Form>
  );
}
```


## Régénération du code

Pour régénérer le code après des changements dans l'API :

```bash
fdn sync
```

Cette commande :
1. Vide le contenu du dossier `src/api/generated` (sans supprimer le dossier)
2. Exécute Orval pour régénérer le code
3. Applique ESLint pour corriger les problèmes de formatage
4. Recrée le fichier `.gitignore` dans `src/api/generated`