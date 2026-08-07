# KaraData (Decentralized Data Marketplace)
## Document d'Architecture

> Document de référence technique et business pour KaraData, plateforme de marketplace décentralisée de données à vocation mondiale. Le lancement démarre en Afrique de l'Ouest comme marché de validation à fort potentiel de différenciation (données rares, sous-représentées), avec une architecture conçue pour s'étendre à d'autres régions du monde.

---

## Sommaire

1. [Vue d'ensemble de la stack](#1-vue-densemble-de-la-stack)
2. [Architecture technique détaillée](#2-architecture-technique-détaillée)
3. [Structure du repo](#3-structure-du-repo)
4. [Modèle économique & GenCoin](#4-modèle-économique--gencoin)
5. [Roadmap MVP](#5-roadmap-mvp)

---

## 1. Vue d'ensemble de la stack

```
[NEXT.JS] ➔ (UX / Connexion wallet / Pré-chiffrement léger)
     ↓
[NODE.JS / EXPRESS] ➔ (API, réception fichiers, orchestration, queue)
     ↓
[PYTHON] ➔ (Analyse IA, vérification qualité, calcul lourd)
```

| Couche | Rôle | Ne fait JAMAIS |
|---|---|---|
| Next.js | Affichage, wallet, pré-chiffrement petits fichiers | Traitement de données, calculs IA |
| Node.js | I/O massif, API, base utilisateurs/points, transactions | Calculs mathématiques/statistiques lourds |
| Python | Analyse IA, qualité des données, algorithmes | Servir le site web principal |

---

## 2. Architecture technique détaillée

### 2.1 Le broker de queue (pièce centrale)

| Option | Quand l'utiliser |
|---|---|
| **Redis + BullMQ** | MVP / démarrage simple, rapide à mettre en place |
| **RabbitMQ** | Montée en charge meilleure gestion des retries et dead-letter queues |
| **Kafka** | Seulement si volume massif (millions d'événements/sec) overkill au début |

**Recommandation** : démarrer avec Redis + BullMQ, migrer vers RabbitMQ si le volume l'exige.

### 2.2 Stockage des fichiers

- Node.js ne stocke **jamais** les fichiers lourds (ex. 500 Mo) sur son propre disque.
- Les fichiers sont **streamés directement** vers un stockage objet :
  - **S3** (ou compatible S3) pour une solution classique.
  - **IPFS / Filecoin** pour rester cohérent avec l'aspect Web3/décentralisé du projet.
- Node.js ne manipule que les **métadonnées** (URL ou CID) qu'il place dans la queue.

### 2.3 Communication Node.js ↔ Python

Deux approches possibles :

1. **Queue découplée (recommandé)** : Node.js pousse un job dans Redis/RabbitMQ → un worker Python (via **Celery** ou un poller custom) consomme le job. Avantage : Python peut scaler indépendamment de Node.js.
2. **Appel direct (gRPC / FastAPI interne)** : plus rapide, mais recouple les deux services à éviter si on veut scaler l'IA séparément (ce qui sera nécessaire, l'IA étant gourmande en ressources).

### 2.4 Flux complet du traitement d'un fichier

1. Le vendeur dépose son fichier via Next.js.
2. Node.js reçoit le fichier, le stream vers S3/IPFS, enregistre les métadonnées.
3. Node.js répond immédiatement : *"Fichier reçu, analyse en cours !"* l'interface se libère.
4. Node.js place un job dans la Task Queue (Redis/BullMQ).
5. Un worker Python prend le job, télécharge le fichier depuis le stockage objet, l'analyse (qualité, IA), calcule un verdict.
6. Python met à jour la base de données avec le résultat.
7. Le vendeur est notifié en temps réel (voir 2.5).

### 2.5 Notification temps réel

- Utiliser **Socket.io** (natif dans l'écosystème Node.js) pour notifier le vendeur dès que l'analyse Python est terminée.
- Alternative plus légère : polling côté Next.js sur un endpoint de statut.

### 2.6 Sécurité des workers Python

- Les fichiers uploadés viennent d'inconnus → **sandboxer** l'exécution des scripts Python (containers isolés type Docker/Firecracker).
- Imposer des **limites strictes CPU/mémoire/temps d'exécution** par job pour éviter qu'un fichier malveillant ne fasse tomber tout le pool de workers.
- Valider le type MIME réel du fichier (pas seulement l'extension) avant analyse.

---

## 3. Structure du repo

**Choix retenu : Monorepo** — tout le code dans un seul repo Git, avec des dossiers/services séparés. Avantages pour ce projet : partage facile des types/schémas entre Node.js et Next.js, un seul historique Git, déploiement coordonné plus simple au début (moins pertinent d'avoir la complexité de repos multiples tant que l'équipe est petite).

### 3.1 Arborescence proposée

```
data-marketplace/
├── apps/
│   ├── web/                    # Next.js — front-end (catalogue, wallet, dashboard)
│   │   ├── app/
│   │   ├── components/
│   │   └── package.json
│   │
│   └── api/                    # Node.js/Express — backend API
│       ├── src/
│       │   ├── routes/
│       │   ├── controllers/
│       │   ├── services/       # logique métier (upload, queue, transactions)
│       │   ├── models/         # schémas base de données
│       │   ├── queue/          # config BullMQ/Redis
│       │   └── sockets/        # Socket.io (notifications temps réel)
│       └── package.json
│
├── workers/
│   └── analyzer/                # Python — worker d'analyse IA/qualité
│       ├── tasks/                # tâches Celery
│       ├── models/               # modèles IA
│       ├── sandbox/               # config d'isolation (Docker limits)
│       └── requirements.txt
│
├── packages/                     # code partagé entre apps (monorepo)
│   ├── shared-types/              # types TypeScript partagés (Next.js ↔ Node.js)
│   └── contracts/                  # ABI / adresses des smart contracts (GenCoin)
│
├── contracts-solidity/            # smart contracts GenCoin (Solidity/Hardhat)
│   ├── contracts/
│   ├── test/
│   └── scripts/
│
├── infra/                         # docker-compose, configs Redis/RabbitMQ, déploiement
│   ├── docker-compose.yml
│   └── k8s/ (si besoin plus tard)
│
├── docs/
│   └── architecture-ddm.md        # ce document
│
├── package.json                   # workspace root (npm/pnpm workspaces ou Turborepo)
└── turbo.json                     # si Turborepo utilisé pour orchestrer les builds
```

### 3.2 Outils recommandés pour le monorepo

- **pnpm workspaces** ou **Turborepo** : pour gérer les dépendances partagées entre `web` et `api` sans dupliquer `node_modules`.
- Python (`workers/analyzer`) reste isolé avec son propre `requirements.txt` / environnement virtuel un monorepo ne veut pas dire mélanger les gestionnaires de paquets, juste centraliser le code source.
- `packages/shared-types` : essentiel pour garder Next.js et Node.js synchronisés sur la forme des données (ex. structure d'un fichier de données, statut d'analyse, etc.) sans dupliquer les interfaces.

---

## 4. Modèle économique & GenCoin

### 4.1 Le token : GenCoin (GNC)

| Paramètre | Valeur |
|---|---|
| Nom / Symbole | GenCoin / GNC |
| Réseau | BNB Smart Chain (BSC) |
| Standard | BEP-20 |
| Supply maximum | 200 000 000 GNC |
| Décimales | 18 |
| Taxe par transaction | 2% (paramétrable jusqu'à 5% max, via `setTaxPercent`) |
| Statut | Contrat rédigé et testé dans Remix (pas encore déployé en production) |

### 4.2 Répartition de la supply initiale

| Allocation | % | Montant | Destination |
|---|---|---|---|
| Fondateur | 40% | 80 000 000 GNC | Wallet du fondateur |
| Liquidité | 30% | 60 000 000 GNC | À injecter dans le pool PancakeSwap |
| Marketing | 15% | 30 000 000 GNC | Campagnes, partenariats |
| Réserve | 10% | 20 000 000 GNC | Réserve stratégique (peut alimenter les récompenses qualité) |
| Airdrop | 5% | 10 000 000 GNC | Distribution communautaire |

### 4.3 Rôle de GenCoin dans la marketplace

GenCoin a un **double rôle**, comme décidé :

1. **Moyen de paiement** — les acheteurs paient les vendeurs en GNC pour accéder aux jeux de données.
2. **Récompense qualité** — un bonus en GNC (issu de la réserve de 20M) est versé aux vendeurs dont les données obtiennent un bon score de qualité via l'analyse Python (voir section 2.4).

### 4.4 Flux de transaction proposé

```
Acheteur (GNC) ──▶ Smart Contract Marketplace (escrow)
                         │
                         ├──▶ Vendeur (paiement − 2% taxe token − frais plateforme)
                         │
                         └──▶ Si score qualité IA élevé :
                                   Bonus GNC (depuis la Réserve) ──▶ Vendeur
```

- Le **score qualité** est calculé par le worker Python (section 2) et transmis à Node.js, qui déclenche le paiement du bonus (soit via une transaction on-chain automatisée, soit via un mécanisme de claim par le vendeur).
- **Important** : comme la taxe de 2% s'applique à *chaque* transfert on-chain, le prix affiché côté Next.js doit être calculé pour que le vendeur reçoive bien le montant net attendu (ou alors la marketplace absorbe la taxe dans sa marge).

### 4.5 Points de vigilance avant déploiement en production

| Risque | Recommandation |
|---|---|
| Taxe appliquée aussi aux paiements internes | Intégrer le calcul de la taxe dans la logique de pricing côté Node.js/smart contract marketplace |
| `owner` = adresse unique (centralisation) | Migrer vers un **multisig** (ex. Gnosis Safe) avant tout listing public |
| Liquidité non verrouillée automatiquement | Ajouter manuellement la liquidité sur PancakeSwap puis **verrouiller le LP token** (ex. via PinkLock) |
| Pas d'interface `IERC20` standard héritée | Envisager une base **OpenZeppelin ERC20** pour un contrat audité et garanti compatible avec tous les wallets/exchanges |
| Pas d'audit de sécurité | Faire auditer le contrat (même un audit communautaire/automatisé) avant le déploiement mainnet |

---

## 5. Roadmap MVP

**Stratégie retenue : lancer sans crypto d'abord, intégrer GenCoin en v2.**

Justification : valider la demande réelle (vendeurs/acheteurs de données) avec un moyen de paiement simple et familier réduit les risques et la friction d'adoption pour le marché de lancement (Afrique de l'Ouest), où l'usage de wallets crypto reste limité. Cette approche progressive valider localement avant d'étendre s'applique à n'importe quel nouveau marché ciblé par la suite. Ça laisse aussi le temps de sécuriser GenCoin (audit, multisig, liquidité verrouillée. voir section 4.5) avant son intégration mondiale.

### Phase 1 — MVP (paiement classique)

- [ ] Next.js : catalogue de données (SSR) + dashboard vendeur (CSR), sans wallet crypto
- [ ] Node.js : upload de fichiers → stockage objet (S3) → queue (Redis/BullMQ)
- [ ] Python : worker d'analyse qualité basique (pas encore d'IA complexe)
- [ ] Paiement : intégration mobile money (Orange Money, MTN MoMo) + carte bancaire
- [ ] Notifications temps réel (Socket.io) pour le statut d'analyse
- [ ] Base de données utilisateurs/points (sans blockchain)

### Phase 2 — Renforcement produit

- [ ] Algorithmes IA plus poussés côté Python (scoring qualité avancé)
- [ ] Sandboxing complet des workers (sécurité)
- [ ] Système de réputation vendeur (basé sur les scores qualité historiques)
- [ ] Optimisation SEO du catalogue (Next.js SSR)

### Phase 3 — Intégration Web3 / GenCoin

- [ ] Audit de sécurité du smart contract GenCoin
- [ ] Déploiement mainnet BSC + création et verrouillage de la liquidité PancakeSwap (PinkLock)
- [ ] Migration `owner` vers un multisig (Gnosis Safe)
- [ ] Connexion wallet (MetaMask/WalletConnect) dans Next.js
- [ ] Smart contract marketplace (escrow) pour les paiements en GNC
- [ ] Mécanisme de bonus qualité en GNC (depuis la réserve)
- [ ] Option pour les utilisateurs : payer en GNC OU en paiement classique (transition douce)
