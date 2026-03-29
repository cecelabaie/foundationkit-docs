# Light and Dark mode

Système de thème dark/light avec `next-themes`, variables CSS et persistance localStorage.

Le projet utilise **Shadcn/ui** pour les composants et **Tailwind CSS v4** pour le styling.

## Structure des fichiers

**Point d'entrée :** `src/assets/styles/globals.css`

```css
@import './tailwind-color.css';
@import './theme/palette.css';
@import './theme/extends.css';
@import './theme/toast.css';
@import './background.css';
```

Ce fichier importe tous les CSS du thème dans le bon ordre.

---

## Palette

**Fichier:** `src/assets/styles/theme/palette.css`

Définit toutes les couleurs pour les modes light et dark.

### Variables personnalisées

```css
html.light {
  --primary-light: #ebf3ff;
  --secondary-light: #000000;
  --surface-light: #fbfaff;
  --neutral-light: #003d6b;
  --success-light: #22c55e;
  --info-light: #3b82f6;
  --accent-light: #0107c1;
  --warning-light: #facc15;
  --error-light: #a80505;
  --background-light: #f5f9ff;
  --muted-light: #707070;
}

html.dark {
  --primary-dark: #0d275d;
  --secondary-dark: #9fb9d0;
  --surface-dark: #020711;
  --neutral-dark: #ffffff;
  --success-dark: #addfad;
  --info-dark: #89e0eb;
  --accent-dark: #b387fa;
  --warning-dark: #fbc700;
  --error-dark: #dc4343;
  --background-dark: #010409;
  --muted-dark: #c2c2c2;
}
```

### Mapping vers Shadcn/Tailwind

```css
html.light {
  --primary: var(--primary-light);
  --foreground: var(--secondary-light);
  --background: var(--background-light);
  --card: var(--surface-light);
  --accent: var(--accent-light);
}

html.dark {
  --primary: var(--primary-dark);
  --foreground: var(--secondary-dark);
  /* ... etc */
}
```

### Theme inline pour ajouter des couleurs

**1. Déclarer dans `palette.css`**

```css
html.light {
  --ma-couleur-light: #ff0000;
  --ma-couleur: var(--ma-couleur-light);
}

html.dark {
  --ma-couleur-dark: #00ff00;
  --ma-couleur: var(--ma-couleur-dark);
}
```

**2. Déclarer pour Tailwind dans `tailwind-color.css`**

```css
@theme inline {
  --color-ma-couleur: var(--ma-couleur);
}
```

⚠️ Préfixe `--color-` obligatoire pour Tailwind.

**3. Utiliser**

```tsx
<div className="bg-ma-couleur text-ma-couleur">Contenu</div>
```

**Classes disponibles :**
- `bg-primary`, `text-primary`
- `bg-background`, `text-foreground`
- `border-accent`, `text-accent`
- `bg-success`, `bg-error`, `bg-warning`

---

## Extends.css

**Fichier:** `src/assets/styles/theme/extends.css`

Override général des styles globaux : inputs, boutons, liens, body.

---

## Next thème

Package `next-themes@^0.4.6` configuré dans `src/providers/providers.tsx`.

### Configuration

- `enableSystem={false}` — thème contrôlé manuellement, pas par les préférences OS
- `attribute="class"` — ajoute/retire la classe `light` ou `dark` sur `<html>`, ce qui active les variables CSS de `palette.css` (ex: `html.dark { }`)
- `storageKey="theme"` — clé localStorage utilisée (aussi lue par le script `theme-sanitize`)
- `themes={['light', 'dark']}` — seules ces deux valeurs sont acceptées

### Utilisation

```tsx
import { useTheme } from 'next-themes';

const { theme, setTheme } = useTheme();

setTheme(theme === 'dark' ? 'light' : 'dark'); // Toggle
```

Le hook `useTheme()` donne accès au thème actuel et permet de le changer. Le changement met à jour le `localStorage` et la classe sur `<html>`, ce qui déclenche les variables CSS de `palette.css`.

### Local Storage

Clé `theme`. La lib sauvegarde/restaure automatiquement.

### Script dans le head

**Fichier:** `layout.tsx`

Script `beforeInteractive` qui sanitize le localStorage au chargement. Si la valeur est corrompue ou invalide, force **`'dark'`** (thème sombre par défaut), en gardant `localStorage` et la classe sur `<html>` alignés. Évite le flash de thème (FOUC).

---

## Le bouton de changement de thème

**Fichiers :**
- `src/sections/header/header-desktop.tsx`
- `src/sections/header/header-mobile.tsx`

---

## Barre de chargement de page

**Fichier:** `src/providers/providers.tsx`

Utilise `nextjs-toploader` pour afficher une barre de progression en haut de l'écran lors des navigations.

```tsx
<NextTopLoader
  color={`var(--secondary)`}
  shadow={`0 0 10px var(--secondary),0 0 5px var(--secondary)`}
  height={3}
  crawlSpeed={200}
  speed={200}
  zIndex={1600}
/>
```

La barre utilise la variable `--secondary` du thème actif. Elle change donc de couleur automatiquement quand le thème change (light/dark).

---

## Notifications (Toasts)

**Fichier:** `src/assets/styles/theme/toast.css`

Applique les couleurs du thème aux barres de progression des notifications toast (react-toastify).

Les barres de progression prennent les couleurs selon le type de notification :
- Success → `--color-success`
- Error → `--color-error`
- Warning → `--color-warning`
- Info → `--color-info`

Les notifications s'adaptent automatiquement au thème actif.