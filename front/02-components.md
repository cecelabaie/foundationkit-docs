# Tous les composants

Le projet utilise **Shadcn/ui** comme base pour les composants UI. Tous les composants sont dans `src/components/`.

## Storybook

Le projet dispose d'un **Storybook** qui présente tous les composants et leurs variantes de façon interactive.

```bash
npm run storybook
```

Ouvre sur [http://localhost:6006](http://localhost:6006). Toutes les variantes, props et états de chaque composant y sont documentés visuellement. C'est la référence de fait pour explorer les composants disponibles avant d'en utiliser un.

---

## Structure

```
components/
├── ui/                    # Composants Shadcn de base + composants custom
├── background/            # BackgroundGrid (Magic UI)
├── form/inputs/           # Wrappers React Hook Form
├── modals/                # Modales réutilisables (EditModal, DeleteModal)
└── loaders/               # Loaders (auth, page)
```

---

## Composants UI

**Dossier :** `src/components/ui/`

Composants de base Shadcn, tous personnalisés avec les variables du thème, plus quelques composants custom.

### Liste des composants UI

- **accordion** - Accordéons dépliables
- **alert** - Alertes/notifications inline
- **alert-dialog** - Dialogues d'alerte (confirmations destructives)
- **badge** - Badges/étiquettes
- **button** - Boutons avec variants (default, destructive, ghost, outline, link, icon)
- **calendar** - Calendrier (date-picker)
- **card** - Cartes de contenu
- **checkbox** - Cases à cocher
- **command** - Command palette (recherche)
- **dialog** - Composant Dialog Shadcn de base (voir `modals/` pour les modals réutilisables)
- **drawer** - Tiroirs latéraux (mobile)
- **dropdown-menu** - Menus déroulants
- **footer** - Pied de page générique configurable (brand, tagline, sections, liens)
- **form** - Wrapper formulaire (React Hook Form)
- **input** - Input texte de base
- **input-group** - Groupe d'inputs avec prefix/suffix
- **input-number** - Input numérique
- **label** - Labels de formulaire
- **language-selector** - Sélecteur de langue FR/EN
- **link** - Lien typé avec variants
- **navigation-menu** - Menu de navigation
- **popover** - Popovers
- **radio-group** - Boutons radio
- **select** - Select dropdown
- **separator** - Séparateurs
- **sonner** - Notifications toast
- **stepper** - Indicateur d'étapes
- **switch** - Interrupteurs toggle
- **tabs** - Onglets
- **text** - Composant typographique avec variants (h1→h4, lead, muted, small…)
- **textarea** - Zone de texte multi-lignes
- **theme-toggle** - Bouton toggle light/dark
- **user-menu** - Menu utilisateur (popover avec liens profil/déconnexion)

### Utilisation

```tsx
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader } from '@/components/ui/card';
import { Text } from '@/components/ui/text';

<Button variant="default" size="lg">
  Cliquer
</Button>

<Card>
  <CardHeader>Titre</CardHeader>
  <CardContent>Contenu</CardContent>
</Card>

<Text variant="h2">Titre principal</Text>
<Text variant="muted">Texte secondaire</Text>
```

**Variants du Button :**
- `default` - Bouton principal
- `destructive` - Action destructive (rouge)
- `ghost` - Transparent
- `outline` - Bordure uniquement
- `link` - Style lien
- `icon` - Icône seule

---

## BackgroundGrid (Magic UI)

**Fichier :** `src/components/background/background-grid.tsx`

Grille SVG animée issu de [Magic UI](https://magicui.design), utilisée en fond de page. Masquée sur les bords via deux gradients horizontaux/verticaux.

```tsx
import { BackgroundGrid } from '@/components/background/background-grid';

<BackgroundGrid />
```

Pour appliquer un masque custom (ex: trou radial centré), wrappez le composant dans un `div` avec `maskImage` dans la vue : ne modifiez pas le composant partagé.

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

1. **rhf-text-input** - Texte libre
2. **rhf-email-input** - Email (validation format)
3. **rhf-password-input** - Mot de passe avec toggle visibility
4. **rhf-text-area-input** - Zone de texte multi-lignes
5. **rhf-number-input** - Nombres
6. **rhf-select-input** - Select avec options
7. **rhf-combobox-input** - Select avec recherche
8. **rhf-radio-input** - Boutons radio
9. **rhf-checkbox-input** - Case à cocher
10. **rhf-switch-input** - Switch toggle
11. **rhf-date-input** - Date avec calendrier
12. **rhf-calendar-input** - Calendrier complet
13. **rhf-file-input** - Upload de fichier
14. **rhf-input-color-picker** - Sélecteur de couleur
15. **rhf-multi-select-input** - Sélection multiple avec recherche
16. **rhf-dynamic-select-input** - Select avec création à la volée via modale

### Utilisation avec React Hook Form

```tsx
import { useForm } from 'react-hook-form';
import { Form } from '@/components/form/form';
import { RhfTextInput, RhfSelectInput, RhfDateInput } from '@/components/form/inputs';

const methods = useForm({
  defaultValues: { email: '', gender: '', birthDate: '' }
});

<Form methods={methods} onSubmit={methods.handleSubmit(onSubmit)}>

  <RhfTextInput
    control={methods.control}
    name="email"
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
- `className` - Classes CSS supplémentaires (s'applique au conteneur `FormItem`)

### Validation automatique

Les erreurs de validation s'affichent automatiquement sous le champ grâce au composant `FormMessage` intégré.

---

## Composants avancés

### RhfMultiSelectInput

Select qui stocke un **tableau de strings** (`string[]`). Propose une recherche et un bouton "Tous" pour tout sélectionner/désélectionner.

```tsx
<RhfMultiSelectInput
  control={methods.control}
  name="tagIds"
  label="Tags"
  placeholder="Rechercher..."
  required
  options={tags.map((tag) => ({ value: tag.id, label: tag.name }))}
/>
```

---

### RhfDynamicSelectInput

Select qui stocke un **id (string)** et permet de **créer une nouvelle entrée en base de données à la volée** via une modale, sans quitter le formulaire en cours.

La prop `translationKey: DynamicSelectKey` détermine les labels du popover et le formulaire de création affiché dans la modale.

```tsx
<RhfDynamicSelectInput
  control={methods.control}
  name="projectId"
  label="Projet"
  required
  options={projectOptions}
  translationKey={DynamicSelectKey.Project}
  isDataLoading={isLoading}
/>
```

#### Ajouter un nouveau type d'entité

Pour brancher `RhfDynamicSelectInput` sur une nouvelle entité (ex : `Client`), 4 étapes dans `rhf-dynamic-select-input.tsx` :

1. Ajouter la clé dans l'enum `DynamicSelectKey`
2. Ajouter le `case` dans le switch des labels
3. Créer le formulaire de création (`onClose`, `onCreated` props)
4. Brancher le formulaire dans le bloc conditionnel de la `Dialog`

---

## Modales

**Dossier :** `src/components/modals/`

### EditModal

Modale d'édition avec `Form` intégré, affichage `errors.root`, boutons Annuler / Enregistrer.

```tsx
import { EditModal } from '@/components/modals/edit-modal';

<EditModal
  open={open}
  onOpenChange={(next) => { if (!next) onClose(); }}
  title="Modifier l'élément"
  methods={methods}
  onSubmit={handleSubmit(onSubmit)}
  onCancel={onClose}
  loading={isPending}
>
  <RhfTextInput control={methods.control} name="name" label="Nom" required />
</EditModal>
```

### DeleteModal

Modale de confirmation basée sur `AlertDialog` (ne se ferme pas en cliquant à côté).

```tsx
import { DeleteModal } from '@/components/modals/delete-modal';

<DeleteModal
  open={open}
  onClose={onClose}
  title="Suppression du projet"
  description="Êtes-vous sûr de vouloir supprimer ce projet ?"
  error={error}
  onConfirm={handleConfirm}
  isConfirming={isPending}
/>
```

---

## Loaders

**Dossier :** `src/components/loaders/`

### PageLoader

Spinner SVG animé centré sur la zone disponible.

```tsx
import PageLoader from '@/components/loaders/page-loader';

<PageLoader />
```

### AuthContextLoader

Wrapper plein écran autour de `PageLoader`. Utilisé par les guards pendant la vérification de session.

```tsx
import AuthContextLoader from '@/components/loaders/auth-context-loader';

<AuthContextLoader />
```

---

## Notifications (Toasts)

Les notifications toast utilisent **Sonner**. Importer `toast` directement depuis `'sonner'` :

```tsx
import { toast } from 'sonner';

toast.success('Action réussie !');
toast.error('Une erreur est survenue');
```

Le composant `<Toaster />` est déjà monté dans `Providers` avec `withHeader={true}` par défaut, ce qui positionne les toasts sous le header (`calc(56px + 8px)`).

Sur les pages sans header (ex: page de login, modal plein écran), passer `withHeader={false}` pour positionner les toasts en haut de l'écran (`8px`) :

```tsx
import { Toaster } from '@/components/ui/sonner';

<Toaster withHeader={false} />
```

Voir `src/assets/styles/sonner.css` pour le style du bouton de fermeture et de la barre de progression.
