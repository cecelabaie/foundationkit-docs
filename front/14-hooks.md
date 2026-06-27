# Hooks

Ce document présente les hooks personnalisés développés pour l'application et explique comment les utiliser.

## Vue d'ensemble

L'application utilise plusieurs hooks personnalisés :
1. **useAppRouter** - Navigation avec barre de progression
2. **useIsScreenBelowBreakpoint** - Détection de breakpoints pour rendu conditionnel
3. **useZodI18n** - Mise à jour des messages d'erreur lors du changement de langue
4. **useFileOpen** - Ouverture d'un fichier protégé dans un nouvel onglet
5. **useFileDownload** - Téléchargement d'un fichier protégé avec nom personnalisé

## useAppRouter

Wrapper autour de `useRouter` de Next.js qui déclenche automatiquement la barre de progression. À utiliser à la place de `useRouter` partout dans l'app.

### Utilisation

```tsx
import { useAppRouter } from '@/hooks/useAppRouter';

function MyComponent() {
  const router = useAppRouter();
  
  const handleNavigation = () => {
    router.push('/ma-page'); // La barre de progression s'affiche automatiquement
  };
  
  return <button onClick={handleNavigation}>Naviguer</button>;
}
```


## useIsScreenBelowBreakpoint

Contrairement à `hidden` de Tailwind qui masque via CSS, ce hook **évite le rendu dans le DOM** : utile pour ne pas charger des composants lourds sur mobile.

### Utilisation

```tsx
import { useIsScreenBelowBreakpoint } from '@/hooks/useIsScreenBelowBreakpoint';

function ResponsiveComponent() {
  // Par défaut, utilise le breakpoint 'md' (768px)
  const isMobile = useIsScreenBelowBreakpoint();

  // Ou spécifier un breakpoint personnalisé
  const isSmallScreen = useIsScreenBelowBreakpoint('sm'); // < 640px
  const isTablet = useIsScreenBelowBreakpoint('lg');      // < 1024px
  
  return (
    <div>
      {isSmallScreen ? (
        // Ce composant léger est uniquement rendu sur petit écran
        <SmallScreenComponent />
      ) : isMobile ? (
        // Ce composant est uniquement rendu sur mobile standard
        <MobileComponent />
      ) : isTablet ? (
        // Ce composant est uniquement rendu sur tablette
        <TabletComponent />
      ) : (
        // Ce composant lourd n'est jamais chargé sur les petits écrans
        <DesktopComponent />
      )}
    </div>
  );
}
```

### Breakpoints disponibles

| Clé | Valeur (px) | Description |
|-----|-------------|-------------|
| sm | 640 | Petit mobile |
| md | 768 | Mobile (défaut) |
| lg | 1024 | Tablette |
| xl | 1280 | Petit écran |
| 2xl | 1536 | Grand écran |

## useZodI18n

Force la re-validation des champs qui ont des erreurs lors d'un changement de langue. Sans ce hook, les messages de validation restent dans l'ancienne langue.

### Utilisation

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { useZodI18n } from '@/hooks/useZodI18n';
import { z } from 'zod';

function MyForm() {
  const schema = z.object({ /* ... */} );
  
  const methods = useForm({
    resolver: zodResolver(schema),
    defaultValues: { /* ... */ }
  });
  
  // Applique le hook pour la traduction des erreurs
  useZodI18n(methods);
  
  // Le reste du formulaire...
}
```

Pour plus d'informations sur le système de traduction et son intégration avec Zod, consultez [traductions.md](./06-traductions.md).

## useFileOpen

**Fichier :** `src/hooks/useFileFetch.ts`

Récupère un fichier protégé via l'API (avec les cookies de session) et l'ouvre dans un nouvel onglet. Utilise `AXIOS_INSTANCE`, ce qui signifie que la gestion du refresh token s'applique : si la requête retourne 401, le `QueryClient` tente automatiquement de rafraîchir la session puis rejoue la requête. `isLoading` reste vrai pendant tout ce temps.

### Utilisation

```tsx
import { useFileOpen } from '@/hooks/useFileFetch';

function MyComponent() {
  const { open, isLoading } = useFileOpen();

  return (
    <button onClick={() => open('/files/document.pdf')} disabled={isLoading}>
      Ouvrir le document
    </button>
  );
}
```

| Retour | Type | Description |
|--------|------|-------------|
| `open` | `(url: string) => void` | Déclenche le chargement et l'ouverture du fichier |
| `isLoading` | `boolean` | Vrai pendant le chargement (y compris pendant un refresh de session) |

## useFileDownload

**Fichier :** `src/hooks/useFileFetch.ts`

Récupère un fichier protégé via l'API et déclenche son téléchargement avec un nom de fichier personnalisé. Comme `useFileOpen`, bénéficie du refresh token automatique via `AXIOS_INSTANCE`.

### Utilisation

```tsx
import { useFileDownload } from '@/hooks/useFileFetch';

function MyComponent() {
  const { download, isLoading } = useFileDownload();

  return (
    <button onClick={() => download('/files/report.pdf', 'rapport-2024.pdf')} disabled={isLoading}>
      Télécharger le rapport
    </button>
  );
}
```

| Retour | Type | Description |
|--------|------|-------------|
| `download` | `(url: string, name: string) => void` | Déclenche le chargement et le téléchargement du fichier |
| `isLoading` | `boolean` | Vrai pendant le chargement (y compris pendant un refresh de session) |