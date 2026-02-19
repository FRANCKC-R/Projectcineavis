# Roadmap

Ce document détaille la feuille de route du projet.
# FICHE PROJET — Roadmap & Planning

**Projet :** Cinema Data Core  
**Date :** 2026-02-20  
**Version :** 1.0  
**Durée totale :** 6 mois  
**Statut :** À valider

---

## 1. Vue d'ensemble

| Phase | Période | Focus | Livrable principal |
|-------|---------|-------|-------------------|
| **Phase 1** | Mois 1-2 | Fondations Data | Pipeline ingestion + génération IA |
| **Phase 2** | Mois 3-4 | Socle Technique | API + site minimal fonctionnel |
| **Phase 3** | Mois 5-6 | Social & Lancement | Notations, feeds, beta publique |

---

## 2. Phase 1 — Fondations Data (Mois 1-2)

### Objectif
Avoir un pipeline robuste d'ingestion Wikidata et de génération IA fonctionnel.

### Semaines 1-2 : Setup & Architecture

| Jour | Tâche | Livrable |
|------|-------|----------|
| 1-2 | Initialiser repo Git, structure Go | `cinema-core/` avec cmd/, internal/ |
| 3-4 | Docker Compose (Postgres, Redis, Meilisearch) | `docker-compose.yml` fonctionnel |
| 5-6 | Migrations DB (schema core) | Tables `works`, `persons`, `work_sources` |
| 7-8 | Client Wikidata basique | Fetch par ID, parsing JSON |
| 9-10 | Normalisation données | Mapping Wikidata → struct Go |
| 11-12 | Tests ingestion | 100 œuvres test en base |
| 13-14 | Documentation | README setup, env example |

**Livrable S2 :** Ingestion manuelle fonctionnelle, 100 œuvres en base.

### Semaines 3-4 : Pipeline & Workers

| Jour | Tâche | Livrable |
|------|-------|----------|
| 15-16 | Worker ingestion (cron) | Job toutes les 6h |
| 17-18 | Queue PostgreSQL | Table `job_queue`, status tracking |
| 19-20 | Retry logic | Backoff exponentiel, dead letter |
| 21-22 | Quality scoring | Algo scoring basé sur complétude |
| 23-24 | Monitoring basique | Logs structurés, health check |
| 25-26 | Tests pipeline | 1000 œuvres ingérées auto |
| 27-28 | Optimisation | Batch inserts, conn pooling |

**Livrable S4 :** Pipeline auto, 1000 œuvres, retry/failover OK.

### Semaines 5-6 : Génération IA

| Jour | Tâche | Livrable |
|------|-------|----------|
| 29-30 | Client OpenAI | Client Go, prompts tests |
| 31-32 | Prompt engineering | Template description film |
| 33-34 | Worker génération | Queue + génération async |
| 35-36 | Quality check IA | Scoring auto des descriptions |
| 37-38 | Review workflow | Flag pour validation humaine |
| 39-40 | Tests génération | 500 descriptions générées |
| 41-42 | Itération prompts | Amélioration qualité |

**Livrable S6 :** 500 descriptions IA validées, pipeline génération stable.

### Semaines 7-8 : Polissage Data

| Jour | Tâche | Livrable |
|------|-------|----------|
| 43-44 | Images (liens) | Table `work_images`, fetch URLs |
| 45-46 | Personnes & crew | Ingestion cast + crew |
| 47-48 | Relations œuvres | Suite, remake, série/saisons |
| 49-50 | Meilisearch index | Indexation search |
| 51-52 | Tests charge | 10k œuvres, perf OK |
| 53-54 | Backup setup | Cron pg_dump, rsync |
| 55-56 | Docs API interne | Spec endpoints core |

**Livrable Phase 1 :** 10k œuvres complètes (data + description IA), search fonctionnel.

---

## 3. Phase 2 — Socle Technique (Mois 3-4)

### Objectif
API fonctionnelle, auth sociale, site minimal consultable.

### Semaines 9-10 : API Core

| Jour | Tâche | Livrable |
|------|-------|----------|
| 57-58 | API Gateway | Routing, middleware base |
| 59-60 | Endpoints works | GET /works, /works/:id |
| 61-62 | Endpoints persons | GET /persons, /persons/:id |
| 63-64 | Search endpoint | GET /works/search (Meilisearch) |
| 65-66 | Pagination | Cursor pagination |
| 67-68 | Rate limiting | Redis-based |
| 69-70 | Tests API | Collection Postman/HTTP |

**Livrable S10 :** API publique v1, documentation.

### Semaines 11-12 : Auth & Social DB

| Jour | Tâche | Livrable |
|------|-------|----------|
| 71-72 | Migrations social | Tables `users`, `oauth`, `ratings` |
| 73-74 | OAuth Google | Login avec Google |
| 75-76 | OAuth Apple | Login avec Apple |
| 77-78 | OAuth X | Login avec X |
| 79-80 | JWT handling | Access + refresh tokens |
| 81-82 | Auth middleware | Protection endpoints |
| 83-84 | Tests auth | Flow complet testé |

**Livrable S12 :** Auth sociale fonctionnelle (3 providers).

### Semaines 13-14 : Social Service

| Jour | Tâche | Livrable |
|------|-------|----------|
| 85-86 | Endpoints ratings | POST /works/:id/rate |
| 87-88 | Endpoints posts | POST /posts, GET /feeds |
| 89-90 | Endpoints votes | POST /posts/:id/vote |
| 91-92 | Feed algorithm | Chronologique + tendance |
| 93-94 | Cache feeds | Redis cache |
| 95-96 | Tests social | Scénarios complets |
| 97-98 | API docs | Spec complète |

**Livrable S14 :** API social complète, cache fonctionnel.

### Semaines 15-16 : Frontend Minimal

| Jour | Tâche | Livrable |
|------|-------|----------|
| 99-100 | Setup Next.js 14 | App Router, Tailwind |
| 101-102 | Layout base | Header, navigation |
| 103-104 | Page Home | Feed global (lecture) |
| 105-106 | Page Work | Fiche œuvre complète |
| 107-108 | Page Person | Fiche personne |
| 109-110 | Auth frontend | Login/logout flows |
| 111-112 | Search UI | Barre recherche + résultats |
| 113-114 | Notation UI | Modal notation (read-only si pas auth) |
| 115-116 | Responsive | Mobile OK |
| 117-118 | SEO base | Meta tags, sitemap |
| 119-120 | Tests E2E | Cypress/Playwright |

**Livrable Phase 2 :** Site déployable, consultation + auth fonctionnels.

---

## 4. Phase 3 — Social & Lancement (Mois 5-6)

### Objectif
Notations, feeds interactifs, beta publique.

### Semaines 17-18 : Notations & Profils

| Jour | Tâche | Livrable |
|------|-------|----------|
| 121-122 | Modal notation complète | 6 critères, sliders |
| 123-124 | Soumission note | POST + update moyenne |
| 125-126 | Page profil utilisateur | Stats, historique |
| 127-128 | Édition profil | Bio, avatar |
| 129-130 | Mes notes | Liste notes données |
| 131-132 | Tests notation | Flow complet |

**Livrable S18 :** Notation 1-10 complète, profils utilisateurs.

### Semaines 19-20 : Feeds & Interactions

| Jour | Tâche | Livrable |
|------|-------|----------|
| 133-134 | Création posts | Input + soumission |
| 135-136 | Feed œuvre | Posts liés à œuvre |
| 137-138 | Feed global perso | Basé sur follows/activité |
| 139-140 | Votes up/down | UI + API |
| 141-142 | Réponses (1 niveau) | Threads basiques |
| 143-144 | Modération auto | Flag contenu toxique |
| 145-146 | Tests interactions | Scénarios complets |

**Livrable S20 :** Social complet : posts, votes, feeds.

### Semaines 21-22 : Admin & Modération

| Jour | Tâche | Livrable |
|------|-------|----------|
| 147-148 | Dashboard admin | Stats, queue modération |
| 149-150 | Gestion tickets | Assignation, résolution |
| 151-152 | Outils modération | Delete posts, ban user |
| 153-154 | Revendication fiches | Formulaire + validation |
| 155-156 | Tests admin | Flows complets |

**Livrable S22 :** Outils admin et modération fonctionnels.

### Semaines 23-24 : Beta & Lancement

| Jour | Tâche | Livrable |
|------|-------|----------|
| 157-158 | Déploiement prod | VPS configuré, CI/CD |
| 159-160 | Monitoring prod | Alertes, dashboards |
| 161-162 | Beta fermée | 100-500 utilisateurs invités |
| 163-164 | Collecte feedback | Formulaires, interviews |
| 165-166 | Bug fixes | Priorisation rapide |
| 167-168 | Itération | Améliorations majeures |
| 169-170 | Préparation public | Landing, FAQ, réseaux |
| 171-172 | Lancement public | Annonce, acquisition |

**Livrable Phase 3 :** Beta publique lancée, 500+ utilisateurs.

---

## 5. Jalons clés (Milestones)

| Milestone | Date | Critère de succès |
|-----------|------|-------------------|
| **M1** : Data pipeline | Fin mois 2 | 10k œuvres, descriptions IA validées |
| **M2** : API stable | Fin mois 4 | 100% endpoints testés, &lt;200ms p95 |
| **M3** : Beta live | Fin mois 6 | 500 utilisateurs actifs, &lt;5% churn |

---

## 6. Ressources nécessaires

### Infrastructure (VPS 8 Go)

| Élément | Coût mensuel estimé |
|---------|---------------------|
| VPS Hetzner CX41 (8 Go) | ~12€ |
| Backup storage | ~5€ |
| Domaine + SSL | ~15€/an |
| OpenAI API (génération) | ~50-100€ (variable) |
| **Total mensuel** | **~70-120€** |

### Outils

| Outil | Usage | Coût |
|-------|-------|------|
| Cursor | Développement | 20$/mois |
| GitHub | Repo + CI | Gratuit (public) |
| Vercel (option) | Frontend hosting | Gratuit tier |
| Meilisearch Cloud (option) | Search hosted | Self-hosté gratuit |

### Temps (estimation)

| Activité | Heures/semaine | Qui |
|----------|----------------|-----|
| Développement core | 20-30h | Toi + Cursor |
| Review & validation | 5-10h | Toi |
| Prompt engineering | 3-5h | Toi + IA |
| Tests & debug | 5-10h | Toi |
| **Total hebdo** | **35-55h** | — |

---

## 7. Risques & Mitigations

| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|--------|------------|
| Qualité IA insuffisante | Moyenne | Haut | Prompts itératifs, validation humaine |
| Wikidata indisponible | Faible | Moyen | Retry, cache, dégradé gracieux |
| Lenteur développement | Moyenne | Haut | Scope ajustable, priorités claires |
| Peu de traction beta | Moyenne | Haut | Acquisition ciblée, cinéphiles forums |
| Coût OpenAI imprévu | Moyenne | Moyen | Rate limiting, batching, modèle moins cher |

---

## 8. Checklist de validation

### Avant de commencer chaque phase

- [ ] Phase précédente validée (démo fonctionnelle)
- [ ] Specs détaillées écrites (prompts pour Cursor)
- [ ] Environnement de dev prêt
- [ ] Backup configuré

### Avant le lancement

- [ ] 10k œuvres complètes
- [ ] 500+ descriptions IA validées
- [ ] 100 utilisateurs beta testés
- [ ] Monitoring en place
- [ ] Plan de rollback

---

## 9. Prochaines étapes immédiates

1. **Valider cette roadmap** — modifications ?
2. **Créer repo Git** — structure initiale
3. **Setup environnement** — Docker Compose
4. **Premier ticket** — ingestion Wikidata basique

---

**Questions pour toi :**

- Tu veux ajuster des délais ? (certaines semaines trop chargées ?)
- Tu veux ajouter des features dans une phase ?
- Tu préfères démarrer quand ? (date de début)