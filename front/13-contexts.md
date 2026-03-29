# Contexts

Ce document présente les différents contextes React utilisés dans l'application pour gérer l'état global et partager des fonctionnalités entre les composants.

## Vue d'ensemble

L'application utilise plusieurs contextes React pour gérer différents aspects de l'état global :

1. **AuthContext** - Gestion de l'authentification et de la session utilisateur
2. **ToastContext** - Système de notifications et alertes
3. **ThemeSampleContext** - Personnalisation du thème dans la page d'exemple
4. **ReactQueryDevtoolsClient** - Outils de développement pour React Query (mode développement uniquement)

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

## ToastContext

### Utilisation

```tsx
import { useToast } from '@/contexts/ToastContext';

function MyComponent() {
  const { showSuccessToast, showErrorToast, showWarningToast, showInfoToast } = useToast();
  
  const handleSubmit = async () => {
    try {
      await saveData();
      showSuccessToast('Données enregistrées avec succès');
    } catch (error) {
      showErrorToast('Erreur lors de l'enregistrement');
    }
  };
  
  return (
    <button onClick={handleSubmit}>Enregistrer</button>
  );
}
```

## ThemeSampleContext

### Objectif

Le `ThemeSampleContext` est spécifiquement conçu pour la page d'exemple (`/sample`) et permet la personnalisation en temps réel du thème de l'application. Ce contexte n'est pas utilisé dans le reste de l'application.

### Fonctionnalités

- **Édition des couleurs** : Modification en temps réel des variables CSS du thème
- **Prévisualisation instantanée** : Application immédiate des changements de couleur

## ReactQueryDevtoolsClient

### Objectif

Le `ReactQueryDevtoolsClient` est un composant qui intègre les outils de développement de React Query dans l'application. Il n'est actif qu'en mode développement.

### Fonctionnalités

- **Visualisation des requêtes** : Affichage de toutes les requêtes actives, en attente et en erreur
- **Inspection des données** : Examen des données retournées par les requêtes
- **Débogage** : Outils pour comprendre et résoudre les problèmes liés aux requêtes API

### Activation

Les devtools sont accessibles via un petit panneau flottant en bas à droite de l'écran en mode développement. Ils sont automatiquement désactivés en production.