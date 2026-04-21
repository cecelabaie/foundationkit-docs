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
14. **rhf-multi-select-input** - Sélection multiple avec recherche
15. **rhf-dynamic-select-input** - Select avec création à la volée via modale

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
- `className` - Classes CSS supplémentaires. S'applique au conteneur du champ (`FormItem`), à l'input et au `FormMessage`. Permet notamment de définir la taille du champ :
  ```tsx
  className="min-w-[180px]! max-w-fit"
  ```

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

## Composants avancés

### RhfMultiSelectInput

Select qui stocke un **tableau de strings** (`string[]`). Propose une recherche et un bouton "Tous" pour tout sélectionner/désélectionner.

**Label affiché dans le bouton trigger :**
- Aucune sélection → `"Aucun"`
- Toutes les options sélectionnées → `"Tous"`
- 1 seule option → son label
- N options → `"N sélectionnés"`

```tsx
import { RhfMultiSelectInput } from '@/components/form/inputs';

const schema = z.object({
  tagIds: z.array(z.string()).min(1, 'Sélectionner au moins un tag'),
});

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

#### Fonctionnement

L'utilisateur ouvre le select et peut, comme pour `RhfComboboxInput`, **rechercher** ou **sélectionner** une option existante.

En bas du popover, un bouton **"Ajouter..."** ouvre une modale de création. La modale permet :
- d'ajouter une entrée en base de données
- de rafraîchir les options
- de fermer la modale
- d'auto-sélectionner l'entrée ajoutée dans le champ

#### La prop `translationKey`

`translationKey: DynamicSelectKey` détermine :
- les labels du popover (bouton "Ajouter", message "Aucun résultat", titre de la modale)
- le formulaire de création affiché dans la modale

```tsx
import { DynamicSelectKey, RhfDynamicSelectInput } from '@/components/form/inputs';

// Pour cet exemple, les options viennent d'un hook React Query
const { data, isLoading } = useProjectsControllerGetProjects();
const projectOptions = (data?.data ?? []).map((p) => ({ value: p.id, label: p.name }));

<RhfDynamicSelectInput
  control={methods.control}
  name="projectId"
  label={t('my-form.project-label', { defaultValue: 'Projet' })}
  placeholder={t('my-form.project-placeholder', { defaultValue: 'Sélectionner un projet' })}
  required
  options={projectOptions}
  translationKey={DynamicSelectKey.Project}
  isDataLoading={isLoading}
/>
```

**Props spécifiques :**

| Prop | Type | Description |
|---|---|---|
| `options` | `{ value: string; label: string }[]` | Dériver des entités API : `entities.map(e => ({ value: e.id, label: e.name }))` |
| `translationKey` | `DynamicSelectKey` | Détermine les labels et le formulaire de création |
| `isDataLoading` | `boolean` | Affiche un spinner sur le bouton tant que les options n'ont pas chargé |

#### Ajouter un nouveau type d'entité

Pour brancher `RhfDynamicSelectInput` sur une nouvelle entité (ex : `Client`), 4 étapes dans le composant `rhf-dynamic-select-input.tsx` :

**1. Ajouter la clé dans l'enum**

```ts
export enum DynamicSelectKey {
  ProjectType = 'project-type',
  Project = 'project',
  Client = 'client', // ← nouveau
}
```

**2. Ajouter le case dans le switch (labels)**

Le switch détermine quels labels afficher selon le type d'entité sélectionné.

```ts
switch (translationKey) {
  case DynamicSelectKey.ProjectType:
    noResultLabel = t('dynamic-select.project-type.no-result', { defaultValue: 'Aucun résultat' });
    createButtonLabel = t('dynamic-select.project-type.create-button', { defaultValue: 'Ajouter un type' });
    modalTitle = t('dynamic-select.project-type.create-modal-title', { defaultValue: 'Ajout de type' });
    break;
  case DynamicSelectKey.Project:
    noResultLabel = t('dynamic-select.project.no-result', { defaultValue: 'Aucun résultat' });
    createButtonLabel = t('dynamic-select.project.create-button', { defaultValue: 'Ajouter un projet' });
    modalTitle = t('dynamic-select.project.create-modal-title', { defaultValue: 'Ajout de projet' });
    break;
  case DynamicSelectKey.Client: // ← nouveau
    noResultLabel = t('dynamic-select.client.no-result', { defaultValue: 'Aucun résultat' });
    createButtonLabel = t('dynamic-select.client.create-button', { defaultValue: 'Ajouter un client' });
    modalTitle = t('dynamic-select.client.create-modal-title', { defaultValue: 'Ajout de client' });
    break;
}
```

**3. Créer le formulaire de création**

Créer un composant dans `src/sections/<feature>/form/`. Il doit exposer deux props :
- `onClose: () => void` — appelé pour fermer la modale
- `onCreated?: (id: string) => void` — appelé avec l'id de la nouvelle entité, ce qui déclenche l'auto-sélection dans le champ

Voici un exemple complet :

```tsx
interface ClientCreationFormProps {
  onClose: () => void;
  onCreated?: (id: string) => void;
}

export function ClientCreationForm({ onClose, onCreated }: ClientCreationFormProps) {
  const { mutate: createClient, isPending } = useClientsControllerCreate();

  const methods = useForm<CreateFormInputs>({
    resolver: zodResolver(createSchema),
    defaultValues: { name: '' },
  });
  useZodI18n(methods);

  const handleCreate = methods.handleSubmit((data) => {
    createClient({ data }, {
      onSuccess: async () => {
        // Force-refresh du cache pour avoir les données à jour
        const freshData = await queryClient.fetchQuery({
          queryKey: getClientsControllerGetClientsQueryKey(),
          queryFn: () => clientsControllerGetClients(),
          staleTime: 0,
        });
        // Trouver la nouvelle entité dans les données fraîches
        const newClient = freshData?.data?.findLast((c) => c.name === data.name);
        // onCreated transmet l'id → le select auto-sélectionne l'entrée créée
        if (newClient) onCreated?.(newClient.id);
        onClose();
      },
      onError: (error) => {
        // ... gestion des erreurs (voir section Validation)
      },
    });
  });

  return (
    <Form methods={methods} onSubmit={handleCreate}>
      <RhfTextInput control={methods.control} name="name" label="Nom" required />
      <DialogFooter>
        <Button type="button" variant="outline" onClick={onClose}>Annuler</Button>
        <LoadingButton loading={isPending} variant="success" type="submit">Enregistrer</LoadingButton>
      </DialogFooter>
    </Form>
  );
}
```

**4. Brancher le formulaire dans la Dialog**

Dans `rhf-dynamic-select-input.tsx`, ajouter le nouveau cas dans le bloc conditionnel de la `Dialog` :

```tsx
{translationKey === DynamicSelectKey.Project ? (
  <ProjectsCreationForm
    onClose={() => setModalOpen(false)}
    onCreated={(id) => onChange(id)}
  />
) : translationKey === DynamicSelectKey.ProjectType ? (
  <TaskTypeCreationForm
    onClose={() => setModalOpen(false)}
    onCreated={(id) => onChange(id)}
  />
) : translationKey === DynamicSelectKey.Client ? ( // ← nouveau
  <ClientCreationForm
    onClose={() => setModalOpen(false)}
    onCreated={(id) => onChange(id)}
  />
) : null}
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

