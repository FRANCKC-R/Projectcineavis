# FICHE PROJET — Vision Générale

**Nom de travail :** Cinema Data Core  
**Date de création :** 2026-02-19  
**Version :** 1.0

---

## 1. Vision (phrase d'accroche)

&gt; *"Une base cinématographique ouverte et une communauté qui explore, évalue et discute chaque œuvre dans ses moindres détails."*

---

## 2. Problème résolu

### Le constat
Les bases cinématographiques existantes (IMDb, AlloCiné, TMDB) souffrent de limitations majeures :

- **Propriétaires ou licences contraignantes** — impossible de réutiliser librement les données
- **Mélange confus** entre contenu éditorial, opinions et données factuelles
- **Pas de traçabilité** — on ne sait pas d'où vient l'information
- **Peu adaptées à l'exploitation data** — API limitées, pas d'historique, pas de qualité garantie
- **Pas de communauté vivante** — consultation passive, pas d'échange structuré

### Le besoin
Il n'existe pas aujourd'hui de base cinématographique :
- **Ouverte** (données libres de droits)
- **Canonique** (une seule vérité, traçable)
- **Data-first** (conçue comme une infrastructure)
- **Communautaire** (avec évaluation et discussion)

---

## 3. Solution proposée

### Double nature du produit

| **Cinema Data Core** | **Cinema Social** |
|----------------------|-------------------|
| Infrastructure de données | Plateforme communautaire |
| Faits bruts, traçables | Évaluations, discussions, découvertes |
| API B2B à terme | Engagement B2C dès le lancement |
| Source de vérité | Moteur de vie et d'acquisition |

### Principes fondamentaux

1. **Séparation faits / interprétation**
   - Faits : titres, dates, cast, crew — sourcés et traçables
   - Interprétation : descriptions, analyses, avis — générés ou communautaires

2. **Source unique de vérité**
   - Wikidata (CC0) comme source primaire
   - Génération IA pour enrichir sans copier

3. **Multi-critères d'évaluation**
   - Notation 1-10 (pas 1-5)
   - Plusieurs dimensions : scénario, réalisation, interprétation, image, son, technique

4. **Valorisation des métiers techniques**
   - Chaque professionnel référencé
   - Pages dédiées (portfolios à terme)

---

## 4. Cibles

### Utilisateurs principaux (B2C)

| Segment | Besoin | Comment on répond |
|---------|--------|-------------------|
| Cinéphiles curieux | Découvrir, creuser, discuter | Feed thématique, évaluations détaillées |
| Professionnels du cinéma | Visibilité, networking | Pages profil, portfolios (v2) |
| Studios / distributeurs | Présence, correction données | Revendication de fiches, corrections |

### Clients potentiels (B2B)

| Segment | Usage |
|---------|-------|
| Apps de rencontre | Matching par goûts cinématographiques |
| Plateformes de streaming | Enrichissement métadonnées |
| Médias / journalistes | Données fiables, API rapide |
| Chercheurs / data scientists | Dataset propre et ouvert |

---

## 5. Périmètre — In / Out

### IN (on fait)

- [x] Films, séries, courts-métrages, documentaires, animations
- [x] Cast & crew complet (acteurs, réalisateurs, techniciens)
- [x] Données factuelles sourcées (Wikidata)
- [x] Descriptions générées par IA (originales, pas copiées)
- [x] Système de notation multi-critères (1-10)
- [x] Feeds thématiques (œuvres, événements)
- [x] Auth social (Google, Apple, X)
- [x] API publique (v2)

### OUT (on ne fait pas — pour l'instant)

- [ ] Streaming / VOD intégré
- [ ] Critiques professionnelles (presse)
- [ ] Box-office temps réel
- [ ] Monétisation dès le lancement
- [ ] Portfolios professionnels complets (v2)

---

## 6. Différenciateurs clés

| # | Différenciateur | Pourquoi c'est unique |
|---|-----------------|----------------------|
| 1 | **Data canonique traçable** | Chaque fait a une source, une date, une confiance |
| 2 | **Descriptions IA originales** | Pas de copie d'IMDb, contenu propre |
| 3 | **Évaluation multi-critères** | Noter un film c'est noter 5-6 dimensions, pas une moyenne |
| 4 | **Valorisation technique** | Le monteur, le DOP, le chef déco ont leur place |
| 5 | **Communauté structurée** | Feeds thématiques, pas un chaos de commentaires |

---

## 7. Objectifs — Roadmap 6 mois

### Mois 1-2 : Fondations Data
- [ ] Ingestion Wikidata (films, personnes, relations)
- [ ] Schéma DB canonique
- [ ] Génération IA descriptions (pipeline + validation)
- [ ] Scoring qualité interne

### Mois 3-4 : Socle Technique
- [ ] API interne (Go)
- [ ] Auth social (Google, Apple, X)
- [ ] Site minimal (recherche, fiches œuvres/personnes)
- [ ] Cache Redis, index Meilisearch

### Mois 5-6 : Social & Lancement
- [ ] Système de notation 1-10 multi-critères
- [ ] Feeds thématiques (chronologique + tendance)
- [ ] Profils utilisateurs basiques
- [ ] Beta fermée (100-500 utilisateurs)

### Post-6 mois (hors scope immédiat)
- [ ] API B2B publique
- [ ] Portfolios professionnels
- [ ] Monétisation (studios, API payante)
- [ ] Algorithmie avancée (recommandations personnalisées)

---

## 8. Risques principaux

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|------------|
| Qualité IA insuffisante | Moyenne | Haut | Validation humaine échantillonnée, itérations prompts |
| Peu de traction utilisateurs | Moyenne | Haut | Beta testée, acquisition ciblée cinéphiles |
| Données Wikidata incomplètes | Haute | Moyen | Focus films récents/populaires d'abord |
| Concurrence IMDb/TMDB | Faible | Moyen | Différenciation data-first + communautaire |
| Scaling technique imprévu | Moyenne | Moyen | Architecture simple, monitoring dès le départ |

---

## 9. Principes de développement

1. **Étape par étape** — on valide le data avant le social
2. **Supervision humaine** — l'IA génère, toi tu valides
3. **Qualité &gt; Quantité** — mieux vaut 10k fiches propres que 100k bancales
4. **Open source d'esprit** — données libres, transparence
5. **Vitesse contrôlée** — Cursor accélère, mais tu gardes la main

---

## 10. Glossaire

| Terme | Définition |
|-------|------------|
| **CC0** | Licence Creative Commons Zero — domaine public |
| **Canonique** | Source de vérité unique, non dupliquée |
| **Consensus** | Validation d'un fait par croisement de sources |
| **Core** | Partie data/infrastructure du produit |
| **Social** | Partie communautaire/engagement |
| **Wikidata** | Base de connaissances libre structurée |

---
