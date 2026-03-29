# Formulaires

Ce document explique comment utiliser les formulaires dans l'application, avec React Hook Form, Zod et les composants personnalisés.

## Structure et composants

### React Hook Form

Le projet utilise [React Hook Form](https://react-hook-form.com/) avec Zod pour la validation.

### Composants de formulaire

Le projet fournit des composants réutilisables pour les formulaires dans le dossier `components/form/inputs`. Ces composants sont des wrappers autour des composants UI de base, adaptés pour fonctionner avec React Hook Form.

#### Liste des composants disponibles

1. `RhfTextInput` - Champs texte, email
2. `RhfPasswordInput` - Mot de passe avec toggle visibilité
3. `RhfTextAreaInput` - Zone de texte multi-lignes
4. `RhfNumberInput` - Nombres
5. `RhfSelectInput` - Select avec options
6. `RhfComboboxInput` - Select avec recherche
7. `RhfRadioInput` - Boutons radio
8. `RhfCheckboxInput` - Case à cocher
9. `RhfSwitchInput` - Switch toggle
10. `RhfDateInput` - Date avec calendrier
11. `RhfCalendarInput` - Calendrier complet
12. `RhfFileInput` - Upload de fichier
13. `RhfInputColorPicker` - Sélecteur de couleur

## Utilisation

### Configuration de base

```tsx
// Importer les composants nécessaires
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Form } from '@/components/form/form';
import { RhfTextInput, RhfPasswordInput } from '@/components/form/inputs';
import { LoadingButton } from '@/components/ui/loading-button';
import { useZodI18n } from '@/hooks/useZodI18n';

// Définir un schéma de validation avec Zod
const loginSchema = z.object({
  email: z.string().email('Email invalide'),
  password: z.string().min(8, 'Mot de passe trop court')
});

// Ou utiliser un schéma généré par Orval
// import { authControllerLoginBody } from '@/api/generated/zod/auth/auth';
// const loginSchema = authControllerLoginBody;

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
    formState: { isSubmitting }
  } = methods;
  
  const onSubmit = (data: LoginFormInputs) => {
    mutate({ data });
  };
  
  return (
    <Form methods={methods} onSubmit={handleSubmit(onSubmit)}>
      <RhfTextInput
        control={methods.control}
        name="email"
        type="email"
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
      
      <LoadingButton 
        type="submit"
        loading={isSubmitting || isPending}
      >
        Se connecter
      </LoadingButton>
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

Pour afficher une erreur globale (non liée à un champ spécifique), utilisez le pattern suivant :

```tsx
import { Alert, AlertTitle } from '@/components/ui/alert';
import { AlertCircleIcon } from 'lucide-react';

// Dans votre composant
const {
  handleSubmit,
  setError,
  formState: { errors }
} = methods;

// Dans votre gestionnaire d'erreur
setError('root', {
  type: 'server',
  message: "Message d'erreur global"
});

// Dans votre JSX
{errors.root && (
  <Alert variant="destructive">
    <AlertCircleIcon />
    <AlertTitle>
      {errors.root.message}
    </AlertTitle>
  </Alert>
)}
```

## Forcer des erreurs de validations

### makeValidationErrorResponse

La fonction `makeValidationErrorResponse` permet de créer manuellement des erreurs de validation côté client pour les champs, notamment pour traiter les réponses d'API qui ne correspondent pas au format attendu par les formulaires.
Pour plus d'informations consultez [utils.md](./15-utils.md).