# Formulaires

Ce document explique comment utiliser les formulaires dans l'application, avec React Hook Form, Zod et les composants personnalisés.

## Structure et composants

### React Hook Form

Le projet utilise [React Hook Form](https://react-hook-form.com/) avec Zod pour la validation.

### Composants de formulaire

Le projet fournit des composants réutilisables pour les formulaires dans le dossier `components/form/inputs`. Ces composants sont des wrappers autour des composants UI de base, adaptés pour fonctionner avec React Hook Form.

#### Liste des composants disponibles

1. `RhfTextInput` - Champs texte
2. `RhfEmailInput` - Email (avec icône `@` intégrée)
3. `RhfPasswordInput` - Mot de passe avec toggle visibilité
4. `RhfTextAreaInput` - Zone de texte multi-lignes
5. `RhfNumberInput` - Nombres
6. `RhfSelectInput` - Select avec options
7. `RhfComboboxInput` - Select avec recherche
8. `RhfRadioInput` - Boutons radio
9. `RhfCheckboxInput` - Case à cocher
10. `RhfSwitchInput` - Switch toggle
11. `RhfDateInput` - Date avec calendrier
12. `RhfCalendarInput` - Calendrier complet
13. `RhfFileInput` - Upload de fichier
14. `RhfInputColorPicker` - Sélecteur de couleur
15. `RhfMultiSelectInput` - Sélection multiple avec recherche
16. `RhfDynamicSelectInput` - Select avec création à la volée via modale

## Utilisation

### Configuration de base

```tsx
// Importer les composants nécessaires
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Form } from '@/components/form/form';
import { Button } from '@/components/ui/button';
import { RhfEmailInput, RhfPasswordInput } from '@/components/form/inputs';
import { useZodI18n } from '@/hooks/useZodI18n';
import { handleServerErrors } from '@/utils/errors';
import { useFeatureControllerAction } from '@/api/generated/{domain}/{domain}';

// Définir un schéma de validation avec Zod
const loginSchema = z.object({
  email: z.string().email('Email invalide'),
  password: z.string().min(8, 'Mot de passe trop court')
});

// Ou utiliser un schéma généré par Orval
// import { AuthControllerLoginBody } from '@/api/generated/zod/auth/auth';
// const loginSchema = AuthControllerLoginBody;

// Type inféré du schéma
type LoginFormInputs = z.infer<typeof loginSchema>;

// Dans votre composant
function LoginForm() {
  const { mutate, isPending } = useFeatureControllerAction();

  const methods = useForm<LoginFormInputs>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: '',
      password: ''
    }
  });
  
  // Hook pour forcer la re-validation quand la langue change
  useZodI18n(methods);
  
  // Destructurer les méthodes et états utiles
  const {
    handleSubmit,
    setError,
    formState: { errors, isSubmitting }
  } = methods;
  
  const onSubmit = handleSubmit((data) => {
    mutate(
      { data },
      {
        onSuccess: () => {
          // Traitement succès...
        },
        onError: (error) => {
          handleServerErrors(error, setError);
        },
      }
    );
  });

  return (
    <Form methods={methods} onSubmit={onSubmit}>
      <RhfEmailInput
        control={methods.control}
        name="email"
        label="Email"
        placeholder="Votre email"
        autoComplete="email"
        required={!loginSchema.shape.email.safeParse('').success}
      />

      <RhfPasswordInput
        control={methods.control}
        name="password"
        label="Mot de passe"
        placeholder="Votre mot de passe"
        autoComplete="current-password"
        required={!loginSchema.shape.password.safeParse('').success}
      />

      <Button
        type="submit"
        loading={isSubmitting || isPending}
      >
        Se connecter
      </Button>
    </Form>
  );
}
```

## Validation et gestion des erreurs

### Validation avec Zod

La validation des formulaires est gérée par [Zod](https://github.com/colinhacks/zod) via le resolver de React Hook Form. Les schémas Zod sont générés automatiquement à partir des définitions d'API (voir [generation.md](./09-generation.md)).

Vous pouvez soit :
- Créer vos propres schémas Zod pour des validations personnalisées
- Utiliser les schémas générés par Orval qui correspondent exactement aux attentes de l'API

Consultez [generation.md](./09-generation.md).

### Affichage des erreurs

Chaque composant de formulaire affiche automatiquement les messages d'erreur sous le champ concerné. Ces messages sont traduits automatiquement grâce à l'intégration de Zod avec i18next.

### Hook useZodI18n

Le hook `useZodI18n` permet de forcer la re-validation du formulaire lorsque la langue change, assurant ainsi que les messages d'erreur sont toujours affichés dans la bonne langue.

### Alerte d'erreur globale

`handleServerErrors` place automatiquement les erreurs non-champ sur `errors.root`. Il suffit d'afficher ce champ dans le JSX :

```tsx
import { Alert, AlertTitle } from '@/components/ui/alert';
import { AlertCircleIcon } from 'lucide-react';

{errors.root && (
  <Alert variant="destructive">
    <AlertCircleIcon />
    <AlertTitle>{errors.root.message}</AlertTitle>
  </Alert>
)}
```

## Gestion des erreurs serveur

### Pattern standard : handleServerErrors

`handleServerErrors` est la fonction à utiliser dans `onError` pour tous les formulaires. Elle dispatche automatiquement selon le code HTTP :

- **400** : erreurs de champ (affichées sous chaque input via `mapFieldsErrors`)
- **401** : silencieux par défaut car le QueryClient intercepte le 401, tente le refresh et rejoue la mutation automatiquement. L'utilisateur ne doit pas voir d'erreur dans ce cas.
- **Autres** : erreur sur `root` (affichée dans l'`Alert` du formulaire)

```tsx
import { handleServerErrors } from '@/utils/errors';

onError: (error) => {
  handleServerErrors(error, setError);
}
```

**Exception : routes dans `ROUTES_WITHOUT_RETRY`**

Pour les routes exclues du retry (ex: `/auth/login`), le QueryClient ne tente pas de refresh sur un 401 car un 401 sur ces routes signifie une vraie erreur métier (mauvais identifiants, token invalide...) et non une session expirée. Dans ce cas, il faut afficher l'erreur 401 explicitement :

```tsx
onError: (error) => {
  handleServerErrors(error, setError, {
    includeUnauthorizedAlert: true,
    onUnauthorized: () => router.push(APP_PATHS.LOGIN), // optionnel : action à exécuter sur 401
  });
}
```

Pour plus de détails sur `handleServerErrors`, `displayError` et `mapFieldsErrors`, consultez [utils.md](./15-utils.md).