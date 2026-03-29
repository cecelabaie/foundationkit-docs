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

- **SWAGGER_JSON** : URL ou chemin vers le fichier OpenAPI. Chargé depuis le premier fichier trouvé dans l'ordre : `.env.production`, `.env.local`, `.env`
- **mode: 'tags-split'** : Génère des fichiers séparés par tags OpenAPI
- **client: 'react-query'** : Génère des hooks React Query
- **mutator personnalisé** : Utilise un client Axios personnalisé
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

Le projet utilise un client Axios personnalisé défini dans `src/transformers/customMutator.ts` :

```typescript
import { SERVER_URL } from '@/constants/constants';
import Axios, { AxiosRequestConfig } from 'axios';

export const AXIOS_INSTANCE = Axios.create({
  baseURL: SERVER_URL,
  withCredentials: true, // Active l'envoi des cookies
});

export default async function customMutator<T>(config: AxiosRequestConfig): Promise<T> {
  const source = Axios.CancelToken.source();

  const promise = AXIOS_INSTANCE({
    ...config,
    cancelToken: source.token,
  }).then(({ data }) => data);

  // Support pour l'annulation des requêtes
  promise.cancel = () => {
    source.cancel('Query was cancelled by React Query');
  };

  return promise;
}
```

Ce mutator personnalisé :
- Configure l'URL de base pour toutes les requêtes
- Active l'envoi des cookies pour l'authentification
- Supporte l'annulation des requêtes avec React Query
- Extrait automatiquement les données de la réponse

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
   - format: email → `zod.string().email()`
   - minLength: 1 → `zod.string().min(1)`

#### Schémas générés

Pour chaque endpoint de l'API, Orval génère des schémas Zod correspondants dans le dossier `src/api/generated/zod/` :

```typescript
// src/api/generated/zod/auth/auth.ts
export const authControllerLoginBody = zod.object({
  email: zod.string().email(),  // Contrainte email du Swagger
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
import {
  InternalServerErrorExceptionResponseDTO,
  TooManyRequestResponseDTO,
  UnauthorizedResponseDTO,
  ValidationExceptionResponseDTO,
} from '@/api/generated/schemas';
import { authControllerLoginBody } from '@/api/generated/zod/auth/auth';
import { Form } from '@/components/form/form';
import { RhfTextInput } from '@/components/form/inputs';
import { parseAxiosError } from '@/utils/errors';
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod';

export default function LoginForm() {
  // Hook React Query généré automatiquement par Orval
  const { mutate: mutate_login } = useAuthControllerLogin();
  
  // Schéma Zod généré automatiquement par Orval à partir des décorateurs du backend
  const zloginSchema = authControllerLoginBody;
  
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
  
  const { handleSubmit, setError } = methods;
  
  // Soumission du formulaire avec handleSubmit
  const onSubmit = handleSubmit((input) => {
    // Utilisation du hook généré pour appeler l'API avec mutate
    mutate_login(
      { data: input },
      {
        onSuccess: () => {
          // Traitement en cas de succès...
        },
        onError: (error) => {
          // Gestion des erreurs typées avec l'utilitaire parseAxiosError
          const errData = parseAxiosError<LoginErrorResponse>(error, 'Erreur');
          // Traitement des erreurs selon le statusCode...
          if (errData.statusCode === 400) {
            // Mapper les erreurs de validation aux champs
          }
          // ...
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
# Sur Windows
npm run gen:win

# Sur Linux
npm run gen:linux
```

Ces commandes :
1. Vident le contenu du dossier `src/api/generated` (sans supprimer le dossier)
2. Exécutent Orval pour régénérer le code
3. Appliquent ESLint pour corriger les problèmes de formatage
4. Recréent le fichier `.gitignore` dans `src/api/generated`