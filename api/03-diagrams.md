# Diagrams

[← Retour au sommaire](./SUMMARY.md)

Ce document présente les diagrammes de flux (flowcharts) qui décrivent le comportement de chaque endpoint de l'API.

## Vue d'ensemble

Les diagrammes sont créés avec **PlantUML**, un langage permettant de générer des diagrammes à partir de texte. Chaque endpoint majeur de l'API possède son propre diagramme qui visualise :

- Les étapes du traitement de la requête
- Les validations effectuées
- Les conditions et branchements
- Les codes de statut HTTP retournés
- Les opérations en base de données

### Structure des fichiers

Chaque diagramme possède un fichier **`*.txt`** (code source PlantUML). Les images **`*.png`** peuvent être générées à partir de ces fichiers (éditeur en ligne, extension VS Code ou CLI PlantUML).

### Organisation

Les diagrammes sont organisés par domaine fonctionnel dans `api/diagrams/` :

```
api/diagrams/
├── auth/              # Authentification
├── reset-password/    # Réinitialisation mot de passe
└── user/              # Gestion utilisateur
```

## Maintien des diagrammes

### Process de mise à jour

1. Modifier le fichier `.txt`
2. Régénérer l'image `.png`
3. Vérifier que le diagramme correspond au code
4. Commiter les deux fichiers (`.txt` et `.png`)
5. L'avantage d'utiliser PlantUML est que l'intelligence artificielle peut facilement générer des diagrammes.

### Synchronisation avec le code

Les diagrammes doivent rester **synchronisés** avec le code source. Lors d'une review de PR :
- Vérifier que les diagrammes impactés sont mis à jour
- S'assurer que le flux documenté correspond au code

## Ressources

- **Documentation PlantUML** : https://plantuml.com/activity-diagram-beta
- **Éditeur en ligne** : http://www.plantuml.com/plantuml/uml/
- **Extension VS Code** : PlantUML (jebbs)
- **Syntaxe complète** : https://plantuml.com/fr/

## Liste complète des diagrammes

### Authentification
- ✅ `auth/login` - Connexion utilisateur
- ✅ `auth/logout` - Déconnexion utilisateur
- ✅ `auth/refresh-session` - Renouvellement des tokens
- ✅ `auth/validate-session` - Validation de session

### Réinitialisation mot de passe
- ✅ `reset-password/forgot` - Demande de réinitialisation
- ✅ `reset-password/validate` - Validation du token
- ✅ `reset-password/update` - Changement du mot de passe

### Utilisateur
- ✅ `user/register` - Inscription
- ✅ `user/verify-account` - Activation du compte
- ✅ `user/profile` - Récupération du profil
- ✅ `user/update` - Mise à jour du profil
- ✅ `user/update-password` - Changement de mot de passe connecté
- ✅ `user/delete-account` - Suppression du compte

---

[← Retour au sommaire](./SUMMARY.md)
