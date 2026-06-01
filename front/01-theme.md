# Light and Dark mode

Système de thème dark/light avec `next-themes`, variables CSS oklch et persistance localStorage.

Le projet utilise **Shadcn/ui** pour les composants et **Tailwind CSS v4** pour le styling.

## Structure des fichiers

**Point d'entrée :** `src/assets/styles/globals.css`

```css
@import "tailwindcss";
@import "tw-animate-css";
@import "shadcn/tailwind.css";
@import './theme.css';
@import './sonner.css';
```

---

## Theme.css

**Fichier :** `src/assets/styles/theme.css`

Fichier central du thème. Contient trois blocs :

### 1. Dark mode variant

```css
@custom-variant dark (&:is(.dark *));
```

Active le variant Tailwind `dark:` qui cible les éléments dans un parent `.dark`.

### 2. Tailwind v4 `@theme inline`

Mappe les variables CSS aux classes utilitaires Tailwind :

```css
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-border: var(--border);
  --color-success: var(--success);
  --color-warning: var(--warning);
  --color-illustration: var(--illustration);
  /* ... autres variables */
}
```

⚠️ Le préfixe `--color-` est obligatoire pour que Tailwind génère les classes utilitaires (`bg-primary`, `text-muted`, etc.).

### 3. Palette et tokens sémantiques

La palette de base utilise la **notation oklch** avec une teinte slate (hue 220 : bleu-gris froid) :

```css
:root {
  /* Palette primitive */
  --slate-50:  oklch(0.984 0.003 220);
  --slate-100: oklch(0.968 0.006 220);
  /* ... */
  --slate-950: oklch(0.098 0.007 220);

  /* Tokens sémantiques : mode clair */
  --background: var(--slate-50);
  --foreground: var(--slate-950);
  --primary: var(--slate-950);
  --primary-foreground: var(--slate-50);
  --muted: var(--slate-200);
  --muted-foreground: var(--slate-500);
  --border: var(--slate-200);
  --success: oklch(0.46 0.17 150);
  --warning: oklch(0.50 0.19 65);
  --illustration: #a8d0ea;
  --radius: 0.625rem;
}

.dark {
  /* Tokens sémantiques : mode sombre */
  --background: var(--slate-950);
  --foreground: var(--slate-50);
  --primary: var(--slate-50);
  --primary-foreground: var(--slate-950);
  --muted: var(--slate-800);
  --illustration: var(--slate-400);
  /* ... */
}
```

Le mode clair est défini sur `:root`, le mode sombre sur `.dark` (classe appliquée sur `<html>` par `next-themes`).

---

## Ajouter une couleur custom

**1. Déclarer dans `theme.css` (`:root` et `.dark`)**

```css
:root {
  --ma-couleur: #ff0000;
}

.dark {
  --ma-couleur: #00ff00;
}
```

**2. Ajouter dans le bloc `@theme inline`**

```css
@theme inline {
  --color-ma-couleur: var(--ma-couleur);
}
```

**3. Utiliser en Tailwind**

```tsx
<div className="bg-ma-couleur text-ma-couleur">Contenu</div>
```

---

## Sonner.css

**Fichier :** `src/assets/styles/sonner.css`

Styles des notifications toast (Sonner) : bouton de fermeture et barre de progression animée (4s, se vide de gauche à droite, pause au hover).

---

## Next-themes

Package `next-themes` configuré dans `src/providers/providers.tsx`.

### Configuration

```tsx
<ThemeProvider attribute="class" defaultTheme="system" enableSystem>
```

- `attribute="class"` : ajoute/retire la classe `dark` sur `<html>`, ce qui active `.dark { }` dans `theme.css`
- `defaultTheme="system"` : thème initial si aucune valeur en localStorage
- `enableSystem` : détection de la préférence OS activée (mais le script `theme-sanitize` s'exécute avant et force `'dark'` si la valeur localStorage est invalide, ce qui court-circuite la détection système en pratique)
- `storageKey` : non défini explicitement, utilise la valeur par défaut `"theme"` de `next-themes` (clé aussi lue par le script `theme-sanitize`)

### Utilisation

```tsx
import { useTheme } from 'next-themes';

const { theme, setTheme } = useTheme();

setTheme(theme === 'dark' ? 'light' : 'dark');
```

### Script dans le head

**Fichier :** `layout.tsx`

Script `beforeInteractive` qui sanitize le localStorage au chargement. Si la valeur est corrompue ou invalide, force **`'dark'`** (thème sombre par défaut), en gardant `localStorage` et la classe sur `<html>` alignés. Évite le FOUC (flash de thème).

---

## Classes disponibles

```tsx
<div className="bg-background text-foreground" />
<div className="bg-primary text-primary-foreground" />
<div className="bg-muted text-muted-foreground" />
<div className="border-border" />
<div className="bg-success bg-warning bg-destructive" />
```

---

## Barre de chargement de page

**Fichier :** `src/providers/providers.tsx`

`nextjs-toploader` affiche une barre de progression en haut de l'écran lors des navigations. Utilise la variable `--secondary` du thème actif.

---

## Bouton toggle thème

**Fichier :** `src/components/ui/theme-toggle.tsx`

Utilisé dans le header desktop et mobile.
