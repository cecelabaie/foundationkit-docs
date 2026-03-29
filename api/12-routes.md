# Routes

[← Retour au sommaire](./SUMMARY.md)

Ce document présente toutes les routes disponibles dans l'API, organisées par catégorie selon leur niveau d'authentification requis. Le comportement détaillé de chaque route est décrit dans les [diagrammes](./03-diagrams.md).

## Routes publiques

Ces routes sont accessibles sans authentification.

### **POST** `/auth/login`

**Fonction** : Authentification d'un utilisateur existant

**Description** : Permet à un utilisateur de se connecter en fournissant son email et son mot de passe. En cas de succès, l'API génère un access token et un refresh token et les stocke dans des cookies sécurisés. Plusieurs sessions (multi-appareils) sont autorisées : chaque connexion crée une nouvelle session sans invalider les autres. Voir [Gestion de la session](./07-session.md).

**Utilisation** : Point d'entrée principal pour l'authentification des utilisateurs.

### **POST** `/auth/logout`

**Fonction** : Déconnexion de l'utilisateur

**Description** : Déconnecte l'utilisateur en révoquant les tokens d'accès et de rafraîchissement de la session courante, puis supprime les cookies. Les autres sessions (autres appareils) restent actives. Voir [Gestion de la session](./07-session.md).

**Utilisation** : Permet aux utilisateurs de fermer leur session de manière sécurisée.

### **POST** `/auth/refresh-session`

**Fonction** : Renouvellement des tokens de session

**Description** : Renouvelle l'access token en utilisant le refresh token valide. Révoque l'ancien access token et génère de nouveaux tokens d'accès et de rafraîchissement. Vérifie également si l'utilisateur nécessite une reconnexion forcée.

**Utilisation** : Maintient la session active sans demander à l'utilisateur de se reconnecter.

### **POST** `/user/register`

**Fonction** : Inscription d'un nouvel utilisateur

**Description** : Crée un nouveau compte utilisateur avec les informations personnelles fournies (nom, prénom, email, date de naissance, genre, mot de passe). Valide l'unicité de l'email et du nom d'utilisateur, et hash le mot de passe avant stockage.

**Utilisation** : Première étape pour les nouveaux utilisateurs souhaitant accéder à l'application.

### **POST** `/user/verify-account`

**Fonction** : Activation du compte utilisateur

**Description** : Active le compte d'un utilisateur nouvellement inscrit en vérifiant le token reçu par email. Si le token est valide, le compte est marqué comme vérifié (`isVerified = true`) et le token est invalidé. Un compte non vérifié ne peut pas se connecter.

**Utilisation** : Étape obligatoire après l'inscription pour accéder à l'application.

### **POST** `/reset-password/forgot`

**Fonction** : Demande de réinitialisation de mot de passe

**Description** : Initie le processus de réinitialisation de mot de passe en envoyant un email contenant un token sécurisé à l'utilisateur. Révoque automatiquement tout token de réinitialisation précédent pour des raisons de sécurité.

**Utilisation** : Permet aux utilisateurs d'accéder à leur compte en cas d'oubli de mot de passe.

### **POST** `/reset-password/validate`

**Fonction** : Validation d'un token de réinitialisation

**Description** : Vérifie la validité d'un token de réinitialisation de mot de passe avant de permettre la mise à jour. S'assure que le token n'a pas expiré et n'a pas déjà été utilisé.

**Utilisation** : Étape intermédiaire pour valider le token reçu par email avant la réinitialisation.

### **POST** `/reset-password/update`

**Fonction** : Mise à jour du mot de passe

**Description** : Met à jour le mot de passe de l'utilisateur en utilisant un token de réinitialisation valide. Valide la correspondance entre le mot de passe et sa confirmation, puis marque l'utilisateur comme nécessitant une reconnexion.

**Utilisation** : Finalise le processus de réinitialisation de mot de passe.

## Routes authentifiées

Ces routes nécessitent une authentification valide via cookie `access_token`.

### **POST** `/auth/validate-session`

**Fonction** : Validation de la session active

**Description** : Vérifie si la session de l'utilisateur est toujours valide en validant l'access token. Si la session est valide, l'utilisateur n'a pas besoin de se reconnecter. Utilisé par le frontend pour vérifier l'état d'authentification sans effectuer d'opérations coûteuses.

**Utilisation** : Vérification périodique de l'état d'authentification côté client.

> ⚠️ **Note** : Cette route ne possède pas de `@Throttle()` contrairement aux autres routes du projet. À ajouter si un rate limiting est souhaité.

### **GET** `/user/profile`

**Fonction** : Récupération du profil utilisateur

**Description** : Retourne les informations complètes du profil de l'utilisateur authentifié, incluant ses données personnelles et son image de profil si disponible.

**Utilisation** : Affichage des informations utilisateur dans l'interface.

### **POST** `/user/update`

**Fonction** : Mise à jour du profil utilisateur

**Description** : Permet à l'utilisateur de modifier ses informations personnelles (nom, prénom, genre, etc.) et optionnellement de mettre à jour son image de profil. Valide les données et traite l'upload de fichier si nécessaire.

**Utilisation** : Gestion des informations personnelles par l'utilisateur.

### **POST** `/user/update-password`

**Fonction** : Changement de mot de passe

**Description** : Permet à un utilisateur authentifié de changer son mot de passe en fournissant l'ancien mot de passe et le nouveau. Valide l'ancien mot de passe et met à jour avec le nouveau mot de passe hashé.

**Utilisation** : Changement de mot de passe pour des raisons de sécurité.

### **DELETE** `/user/delete-account`

**Fonction** : Suppression du compte utilisateur

**Description** : Supprime définitivement le compte de l'utilisateur authentifié. Les tokens (access, refresh, reset-password) ainsi que les données associées (médias, profil) sont supprimés en cascade par Prisma via `onDelete: Cascade`.

**Utilisation** : Permet à l'utilisateur de supprimer son compte de manière irréversible.

### **GET** `/media/private/{mediaId}`

**Fonction** : Accès aux médias privés

**Description** : Endpoint prévu pour l'accès aux fichiers médias privés (images, documents) après vérification des permissions. La logique de vérification des droits d'accès (propriétaire, rôle, etc.) et l'envoi du fichier doivent être implémentées dans le contrôleur. **Par défaut**, tant qu'aucune résolution n'est ajoutée, l'API retourne **401 Unauthorized** et ne sert pas le fichier. Voir [Media](./09-media.md).

**Utilisation** : Accès sécurisé aux fichiers privés une fois la logique d'autorisation en place.


## Sécurité

- **Rate Limiting** : Toutes les routes sont protégées par un système de limitation de requêtes
- **Validation** : Toutes les données d'entrée sont validées et sanitizées
- **Authentification** : Les routes protégées vérifient la validité des tokens JWT
- **Cookies sécurisés** : Les tokens sont stockés dans des cookies HTTP-only et sécurisés
- **Internationalisation** : Tous les messages d'erreur sont traduits selon la langue de l'utilisateur

---

[← Retour au sommaire](./SUMMARY.md)
