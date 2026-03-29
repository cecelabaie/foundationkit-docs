# Tous les composants

Le projet utilise **Shadcn/ui** comme base pour les composants UI. Tous les composants sont dans `src/components/`.

## Structure

```
components/
├── ui/                    # Composants Shadcn de base
├── form/inputs/           # Wrappers React Hook Form
├── translation/           # Sélecteur de langue
└── loaders/              # Loaders (auth, page)
```

---

## Composants UI (Shadcn)

**Dossier :** `src/components/ui/`

Composants de base Shadcn, tous personnalisés avec les variables du thème.

### Liste des composants UI

- **accordion** - Accordéons dépliables
- **alert** - Alertes/notifications inline
- **alert-dialog** - Dialogues d'alerte (confirmations destructives)
- **badge** - Badges/étiquettes
- **button** - Boutons avec variants (default, destructive, ghost, outline, link)
- **calendar** - Calendrier (date-picker)
- **card** - Cartes de contenu
- **checkbox** - Cases à cocher
- **command** - Command palette (recherche)
- **dialog** - Modales
- **drawer** - Tiroirs latéraux (mobile)
- **dropdown-menu** - Menus déroulants
- **form** - Wrapper formulaire (React Hook Form)
- **input** - Input texte de base
- **input-number** - Input numérique
- **label** - Labels de formulaire
- **loading-button** - Bouton avec état loading
- **navigation-menu** - Menu de navigation
- **popover** - Popovers
- **radio-group** - Boutons radio
- **select** - Select dropdown
- **separator** - Séparateurs
- **stepper** - Indicateur d'étapes
- **switch** - Interrupteurs toggle
- **tabs** - Onglets
- **textarea** - Zone de texte multi-lignes
- **timeline-stepper** - Stepper en format timeline

### Utilisation

```tsx
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader } from '@/components/ui/card';

<Button variant="default" size="lg">
  Cliquer
</Button>

<Card>
  <CardHeader>Titre</CardHeader>
  <CardContent>Contenu</CardContent>
</Card>
```

**Variants du Button :**
- `default` - Bouton principal
- `destructive` - Action destructive (rouge)
- `ghost` - Transparent
- `outline` - Bordure uniquement
- `link` - Style lien
- `icon` - Icône seule

---

## Inputs React Hook Form (RHF)

**Dossier :** `src/components/form/inputs`

Wrappers autour des composants UI pour React Hook Form. Gèrent automatiquement la validation et l'affichage des erreurs.

### Import

```tsx
import {
  RhfTextInput,
  RhfPasswordInput,
  RhfSelectInput,
  RhfDateInput,
  // ...
} from '@/components/form/inputs';
```

### Liste des inputs RHF

1. **rhf-text-input** - Texte, email
2. **rhf-password-input** - Mot de passe avec toggle visibility
3. **rhf-text-area-input** - Zone de texte multi-lignes
4. **rhf-number-input** - Nombres
5. **rhf-select-input** - Select avec options
6. **rhf-combobox-input** - Select avec recherche
7. **rhf-radio-input** - Boutons radio
8. **rhf-checkbox-input** - Case à cocher
9. **rhf-switch-input** - Switch toggle
10. **rhf-date-input** - Date avec calendrier
11. **rhf-calendar-input** - Calendrier complet
12. **rhf-file-input** - Upload de fichier
13. **rhf-input-color-picker** - Sélecteur de couleur

### Utilisation avec React Hook Form

```tsx
import { useForm } from 'react-hook-form';
import { Form } from '@/components/form/form';
import { RhfTextInput, RhfSelectInput, RhfDateInput } from '@/components/form/inputs';

const methods = useForm({
  defaultValues: {
    email: '',
    gender: '',
    birthDate: '',
  }
});

<Form methods={methods} onSubmit={methods.handleSubmit(onSubmit)}>
  
  <RhfTextInput
    control={methods.control}
    name="email"
    type="email"
    label="Email"
    placeholder="Votre email"
    required
  />
  
  <RhfSelectInput
    control={methods.control}
    name="gender"
    label="Genre"
    required
    options={[
      { value: 'male', label: 'Homme' },
      { value: 'female', label: 'Femme' },
    ]}
  />
  
  <RhfDateInput
    control={methods.control}
    name="birthDate"
    label="Date de naissance"
    required
  />
  
  <Button type="submit">Envoyer</Button>
</Form>
```

### Props communes

Tous les inputs RHF partagent ces props :

- `control` - Control React Hook Form (obligatoire)
- `name` - Nom du champ (obligatoire)
- `label` - Label affiché
- `placeholder` - Placeholder
- `required` - Affiche un astérisque
- `disabled` - Désactive le champ
- `className` - Classes CSS supplémentaires

### Validation automatique

Les erreurs de validation s'affichent automatiquement sous le champ grâce au composant `FormMessage` intégré.

```tsx
<RhfTextInput
  control={methods.control}
  name="email"
  label="Email"
  required
  // L'erreur s'affiche automatiquement en rouge sous l'input
/>
```

---

## Composants spéciaux

### Loading Button

**Fichier :** `src/components/ui/loading-button.tsx`

Bouton avec état de chargement.

```tsx
import { LoadingButton } from '@/components/ui/loading-button';

<LoadingButton loading={isPending} type="submit">
  Envoyer
</LoadingButton>
```

### Language Selector

**Fichier :** `src/components/translation/language-selector.tsx`

Sélecteur de langue (FR/EN) utilisé dans le header.

```tsx
import LanguageSelector from '@/components/translation/language-selector';

<LanguageSelector />
```

### Loaders

**Dossier :** `src/components/loaders/`

#### PageLoader

Spinner SVG animé avec label accessible (`sr-only`). Utilisé partout où l'application attend un état avant d'afficher du contenu.

```tsx
import PageLoader from '@/components/loaders/page-loader';

// Affiche un spinner centré sur toute la zone disponible
<PageLoader />
```

Couleurs du spinner : `text-primary` (piste) + `fill-secondary` (arc animé) — s'adapte automatiquement au thème.

#### AuthContextLoader

Wrapper plein écran autour de `PageLoader`. Utilisé spécifiquement par les guards pendant la vérification de la session, pour que le spinner occupe tout l'écran au lieu d'une zone partielle.

```tsx
import AuthContextLoader from '@/components/loaders/auth-context-loader';

// Plein écran (min-h-screen), utilisé dans AuthContext pendant checkToken
<AuthContextLoader />
```

---

## Toaster (Notifications)

Les notifications toast sont gérées via `ToastContext`. Voir la section Contexts pour l'utilisation.

```tsx
import { useToast } from '@/contexts/ToastContext';

const { showSuccessToast, showErrorToast } = useToast();

showSuccessToast('Action réussie !');
showErrorToast('Une erreur est survenue');
```

