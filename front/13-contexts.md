# Contexts

Ce document présente les différents contextes React utilisés dans l'application pour gérer l'état global et partager des fonctionnalités entre les composants.

## Vue d'ensemble

L'application utilise plusieurs contextes React pour gérer différents aspects de l'état global :

1. **AuthContext** - Gestion de l'authentification et de la session utilisateur
2. **ThemeSampleContext** - Personnalisation du thème en temps réel
3. **ReactQueryDevtoolsClient** - Outils de développement pour React Query (mode développement uniquement)

Ces contextes sont centralisés dans le composant `Providers` (`src/providers/providers.tsx`) qui encapsule l'application.

## AuthContext

Le `AuthContext` fournit les fonctionnalités d'authentification et de gestion de session à l'ensemble de l'application. Il expose l'état de l'utilisateur connecté et les méthodes nécessaires pour gérer la session.

```tsx
import { useAuth } from '@/contexts/AuthContext';

function MyComponent() {
  const { user, isLoading } = useAuth();
  
  if (isLoading) return <Loader />;
  
  return user ? <p>Connecté en tant que {user.firstName}</p> : <p>Non connecté</p>;
}
```

Pour une documentation complète sur le système d'authentification et la gestion de session, consultez [session.md](./12-session.md).

## ThemeSampleContext

### Objectif

Le `ThemeSampleContext` permet la personnalisation en temps réel du thème. Ce contexte peut être retiré si la fonctionnalité de theme builder n'est plus nécessaire.

## ReactQueryDevtoolsClient

### Objectif

Le `ReactQueryDevtoolsClient` est un composant qui intègre les outils de développement de React Query dans l'application. Il n'est actif qu'en mode développement.

### Fonctionnalités

- **Visualisation des requêtes** : Affichage de toutes les requêtes actives, en attente et en erreur
- **Inspection des données** : Examen des données retournées par les requêtes
- **Débogage** : Outils pour comprendre et résoudre les problèmes liés aux requêtes API

### Activation

Les devtools sont accessibles via un petit panneau flottant en bas à droite de l'écran en mode développement. Ils sont automatiquement désactivés en production.