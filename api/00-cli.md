# CLI FoundationKit

[← Retour au sommaire](./SUMMARY.md)

Le CLI `fdn` est l'outil officiel pour gérer un projet FoundationKit.

## Installation

```bash
npm install -g @foundationkit/cli
```

> Prérequis : un compte et une licence active sur [foundationkit.dev](https://foundationkit.dev).

---

## Commandes globales

### `fdn login`
Connecte votre compte FoundationKit. Requis avant toute autre commande.

```bash
fdn login
```

---

### `fdn logout`
Déconnecte le compte courant.

```bash
fdn logout
```

---

### `fdn license`
Affiche les informations de la licence active (statut, domaine, expiration).

```bash
fdn license
```

---

### `fdn create`
Télécharge et initialise un nouveau projet FoundationKit dans le dossier courant.

```bash
fdn create
fdn create --name mon-projet
```

---

## Commandes API

### `fdn init api`
Installe les dépendances, initialise la base de données (migrations Prisma) et compile l'API.

```bash
fdn init api
fdn init api --yes   # mode non-interactif (pas de confirmation)
```

---

### `fdn build api`
Build l'API NestJS pour la production.

```bash
fdn build api
```

---

## Commandes Frontend

> Voir [Documentation Frontend : CLI](../front/00-cli.md) pour les commandes `fdn init front`, `fdn build front` et `fdn sync`.

---

[← Retour au sommaire](./SUMMARY.md)
