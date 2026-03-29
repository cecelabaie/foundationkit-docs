# Hooks

Ce document présente les hooks personnalisés développés pour l'application et explique comment les utiliser.

## Vue d'ensemble

L'application utilise plusieurs hooks personnalisés :
1. **useAppRouter** - Navigation avec barre de progression
2. **useIsScreenBelowBreakpoint** - Détection de breakpoints pour rendu conditionnel
3. **useZodI18n** - Mise à jour des messages d'erreur lors du changement de langue

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

Contrairement à `hidden` de Tailwind qui masque via CSS, ce hook **évite le rendu dans le DOM** — utile pour ne pas charger des composants lourds sur mobile.

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