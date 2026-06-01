# CLI FoundationKit

[← Retour au sommaire](./SUMMARY.md)

Le CLI `fdn` est l'outil officiel pour gérer un projet FoundationKit.

## Installation

```bash
npm install -g @foundationkit/cli
```

> Prérequis : un compte et une licence active sur [foundationkit.dev](https://foundationkit.dev).
> Pour la connexion et les commandes globales (`fdn login`, `fdn logout`, `fdn create`…), voir [Documentation API : CLI](../api/00-cli.md).

---

## Commandes Frontend

### `fdn init front`
Installe les dépendances, synchronise le client API et compile le frontend.

```bash
fdn init front
fdn init front --yes   # mode non-interactif (pas de confirmation)
```

---

### `fdn build front`
Build le frontend Next.js pour la production.

```bash
fdn build front
```

---

### `fdn sync`
Régénère le client API TypeScript (types, hooks React Query, schémas Zod) depuis le swagger de l'API.

```bash
fdn sync
```

> À utiliser chaque fois que l'API change (nouveaux endpoints, modification de DTOs).
> Voir [Génération](./09-generation.md) pour le détail de ce qui est généré.

---

[← Retour au sommaire](./SUMMARY.md)
