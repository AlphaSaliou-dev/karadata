# Proposition BNB Chain Builder Grant

## KaraData Decentralized Data Marketplace (GenCoin / GNC)

---

## 1. Résumé du projet (Executive Summary)

Une marketplace décentralisée de données conçue pour combler un vide majeur : la rareté de données de qualité issues de marchés et de communautés sous-représentés (Afrique de l'Ouest en premier lieu et dans les secteurs comme la santé et l'économie). La plateforme connecte des fournisseurs de données vérifiées à des acheteurs (chercheurs, entreprises, développeurs IA), avec paiement et récompense qualité en **GenCoin (GNC)**, un token BEP-20 natif sur BNB Smart Chain.

**Ce qui différencie ce projet** : contrairement aux marketplaces de données génériques, l'accent est mis sur des données qui ne sont *pas* librement disponibles sur Internet (données locales, sectorielles, vérifiées par un pipeline d'analyse qualité automatisé).

---

## 2. Le problème

- Les jeux de données de qualité sur des marchés émergents (Afrique de l'Ouest notamment) sont rares, dispersés, et rarement monétisables pour ceux qui les détiennent.
- Les marketplaces de données existantes sont saturées de données déjà scrapées/publiques, ce qui limite leur valeur réelle pour l'IA et la recherche.
- Les fournisseurs de données locales n'ont aujourd'hui aucun moyen simple et équitable d'être rémunérés pour des données vérifiées.

## 3. La solution

Une plateforme en trois couches optimisées :

```
Next.js (UX, wallet, catalogue) → Node.js (API, orchestration, paiements) → Python (analyse qualité IA)
```

- **Sourcing terrain via agents de partenariats data** : recrutement d'agents de développement commercial chargés de négocier des accords de partage de données, avec consentement explicite, auprès d'entreprises locales et internationales — le principal levier de différenciation face aux marketplaces génériques.
- **Vérification qualité automatisée** : chaque dataset soumis passe par un pipeline d'analyse Python (score de qualité, détection de duplication avec des données publiques déjà connues).
- **Paiement et récompense en GNC** : les acheteurs paient en GenCoin ; les vendeurs dont les données obtiennent un score qualité élevé reçoivent un bonus supplémentaire depuis une réserve dédiée.
- **Architecture asynchrone scalable** : traitement des fichiers lourds via file d'attente (Redis/BullMQ), sans bloquer l'expérience utilisateur.

## 4. Pourquoi BNB Chain

- Frais de transaction bas et rapidité, essentiels pour des micro-paiements de données à grande échelle.
- Écosystème BEP-20 mature, compatible avec PancakeSwap pour la liquidité de GNC.
- Alignement avec la stratégie d'accès aux marchés émergents (Afrique, Asie) que BNB Chain cherche à servir.

## 5. État actuel du projet

- ✅ Architecture technique complète documentée (stack Next.js / Node.js / Python, queue asynchrone, stockage objet)
- ✅ Smart contract GenCoin (GNC) rédigé en Solidity, testé sur Remix (BEP-20, supply 200M, mécanisme de taxe 2%)
- ✅ Modèle économique et tokenomics définis (répartition de la supply, flux de paiement/récompense)
- ✅ Roadmap produit en 3 phases (MVP sans crypto → renforcement → intégration Web3 complète)
- 🔲 Développement du MVP (en cours de démarrage)
- 🔲 Audit de sécurité du smart contract
- 🔲 Déploiement mainnet et création de liquidité

## 6. Ce que le grant permettrait de financer

| Poste | Objectif |
|---|---|
| Agents de partenariats data (sourcing terrain) | Recruter des agents de développement commercial pour négocier des accords de partage de données consentis avec des entreprises locales et internationales, le cœur de la stratégie de différenciation (données exclusives, non disponibles sur Internet) |
| Audit de sécurité du smart contract GenCoin | Garantir la sécurité avant tout déploiement mainnet et listing PancakeSwap |
| Développement du MVP (marketplace + pipeline qualité) | Valider la demande réelle sur les niches santé/économie |
| Infrastructure initiale | Couvrir les besoins de scaling au-delà des tiers gratuits une fois la traction validée |
| Intégration Web3 (Phase 3 de la roadmap) | Connexion wallet, smart contract d'escrow marketplace, mécanisme de récompense qualité on-chain |

### 6.1 Budget estimatif 

Hypothèse : 3 agents à temps partiel, zone prioritaire Guinée puis élargissement Afrique de l'Ouest (Sénégal, Côte d'Ivoire), sur une période de 5 mois. Rémunération agents basée sur un taux local de 100-200 $/mois.

| Poste | Estimation | Détail |
|---|---|---|
| Agents de partenariats data (3 agents × 5 mois, 100-200 $/mois) | 1 500 – 3 000 $ | Rémunération de base + commission sur les accords conclus (aligne incitation et coût) |
| Audit de sécurité smart contract | 3 000 – 6 000 $ | Tarif de marché 2026 pour un audit de token BEP-20/ERC-20 simple (source : rapports d'auditeurs indépendants) |
| Développement MVP (temps du fondateur + éventuel freelance ponctuel) | 2 000 – 4 000 $ | Complément si besoin d'aide freelance sur des tâches spécifiques (design, tests) |
| Infrastructure (au-delà des tiers gratuits, sur 6-12 mois) | 500 – 1 000 $ | Stockage objet, base de données si le volume dépasse les tiers gratuits |
| Marge de sécurité / imprévus | ~1 000 $ | Marge standard couvrant les imprévus du projet |
| **Total estimé** | **~8 000 – 15 000 $** | Montant total demandé, ajustable selon les modalités du programme |

## 7. Équipe

- **Alpha Baldé** — Développeur backend (Node.js, MongoDB, intégration IA), porteur du projet, basé à Conakry, Guinée.
- **Contact** : alphasalioubalde231@gmail.com | Telegram : Undertaker
- **Entité juridique** : MarketConnect (SARLU, Guinée) — statuts en cours de finalisation auprès du RCCM. Le projet sera porté comme nouvelle activité/filiale de MarketConnect.

## 8. Prochaines étapes après financement

1. Audit du smart contract GenCoin
2. Développement et lancement du MVP (paiement classique, sans crypto, pour valider la demande)
3. Intégration progressive de GenCoin (Phase 3 de la roadmap produit)
4. Déploiement de la liquidité PancakeSwap avec verrouillage LP

---

*Document de candidature préparé pour le BNB Chain Builder Grant, basé sur le document d'architecture technique complet du projet (architecture-ddm.md).*
