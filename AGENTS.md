# AGENTS.md — Cinema Data Core

> Guide pour les agents IA travaillant sur ce projet.  
> Dernière mise à jour : 2026-02-19

---

## 1. Vue d'ensemble du projet

**Cinema Data Core** (nom de travail : Projectcineavis) est une base cinématographique ouverte et une plateforme communautaire qui permet d'explorer, évaluer et discuter les œuvres cinématographiques dans leurs moindres détails.

### Double nature du produit

| **Cinema Data Core** | **Cinema Social** |
|----------------------|-------------------|
| Infrastructure de données | Plateforme communautaire |
| Faits bruts, traçables | Évaluations, discussions, découvertes |
| API B2B à terme | Engagement B2C dès le lancement |
| Source de vérité | Moteur de vie et d'acquisition |

### Problème résolu

Les bases cinématographiques existantes (IMDb, AlloCiné, TMDB) souffrent de limitations : licences propriétaires, manque de traçabilité, pas d'historique, pas de communauté vivante. Ce projet vise à créer une alternative **ouverte**, **canonique** (source unique de vérité), **data-first** et **communautaire**.

---

## 2. Stack technologique

| Couche | Technologie | Justification |
|--------|-------------|---------------|
| Frontend | Next.js 14 (App Router) | SSR, SEO, Developer Experience |
| Backend API | Go (Echo) | Performance, concurrence, binaire statique |
| Database | PostgreSQL 15 | Relations + JSONB flexible |
| Cache | Redis | Sessions, rate limit, cache API |
| Search | Meilisearch | Full-text, faceting, léger |
| Queue | PostgreSQL + Go workers | Pas de complexité Kafka/RabbitMQ |
| Auth | OAuth 2.0 (Google, Apple, X) | Pas de gestion mots de passe |
| IA | OpenAI API | Génération descriptions |
| Hosting | VPS 8 Go RAM | Souveraineté, coût maîtrisé |

---

## 3. Architecture système

### 3.1 Vue globale

```
┌─────────────────────────────────────────────────────────────┐
│                         CLIENT                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Next.js   │  │   Mobile    │  │   API B2B (futur)   │  │
│  │   (Vercel   │  │   (futur)   │  │                     │  │
│  │   ou VPS)   │  │             │  │                     │  │
│  └──────┬──────┘  └─────────────┘  └─────────────────────┘  │
└─────────┼───────────────────────────────────────────────────┘
          │ HTTPS
          ▼
┌─────────────────────────────────────────────────────────────┐
│                      VPS 8 Go RAM                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              NGINX (reverse proxy)                  │   │
│  │    SSL, rate limit, static files, load balance      │   │
│  └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│  ┌───────────────────────┼───────────────────────┐         │
│  │                       ▼                       │         │
│  │  ┌─────────────────────────────────────────┐  │         │
│  │  │         API GATEWAY (Go/Echo)           │  │         │
│  │  │  • JWT validation                       │  │         │
│  │  │  • Rate limiting (Redis)                │  │         │
│  │  │  • Request ID / tracing                 │  │         │
│  │  │  • Routing /core/* vs /social/*         │  │         │
│  │  │  • Circuit breaker (failover)           │  │         │
│  │  └─────────────────────────────────────────┘  │         │
│  │                       │                       │         │
│  │       ┌───────────────┼───────────────┐       │         │
│  │       ▼               ▼               ▼       │         │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐   │         │
│  │  │  CORE   │    │ SOCIAL  │    │  ADMIN  │   │         │
│  │  │ Service │    │ Service │    │ Service │   │         │
│  │  │  :8081  │    │  :8082  │    │  :8083  │   │         │
│  │  └────┬────┘    └────┬────┘    └─────────┘   │         │
│  │       │              │                       │         │
│  │       └──────────────┼───────────────────────┘         │
│  │                      │                                  │
│  │  ┌───────────────────┼───────────────────┐             │
│  │  │                   ▼                   │             │
│  │  │  ┌─────────┐  ┌─────────┐  ┌────────┐ │             │
│  │  │  │PostgreSQL│  │  Redis  │  │Meilisearch│ │             │
│  │  │  │  :5432  │  │ :6379   │  │ :7700  │ │             │
│  │  │  └─────────┘  └─────────┘  └────────┘ │             │
│  │  └───────────────────────────────────────┘             │
│  │                                                         │
│  │  ┌─────────────────────────────────────────┐             │
│  │  │      WORKERS (Go, background)          │             │
│  │  │  • Ingestion Wikidata (cron)            │             │
│  │  │  • Génération IA (queue)                │             │
│  │  │  • Notifications (webhook)              │             │
│  │  │  • Retry failed jobs                    │             │
│  │  └─────────────────────────────────────────┘             │
│  └─────────────────────────────────────────────────────────┘
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Services

- **Core Service** (:8081) : Gestion des données cinématographiques (œuvres, personnes, relations)
- **Social Service** (:8082) : Fonctionnalités communautaires (notations, posts, feeds)
- **Admin Service** (:8083) : Outils d'administration et modération

---

## 4. Structure de la base de données

### 4.1 Core Database (données canoniques)

Tables principales :
- `works` : Œuvres (films, séries, courts, documentaires, animations)
- `work_sources` : Traçabilité des sources (Wikidata)
- `work_facts` : Faits extraits normalisés
- `work_generations` : Historique des générations IA
- `work_images` : Liens vers images (posters, etc.)
- `persons` : Personnes (acteurs, réalisateurs, techniciens)
- `crew_links` : Liens œuvre-personne avec rôles

### 4.2 Social Database (fonctionnalités communautaires)

Tables principales :
- `users` : Utilisateurs
- `user_oauth` : Liaisons OAuth (Google, Apple, X)
- `ratings` : Notations multi-critères (6 critères 1-10)
- `posts` : Posts dans les feeds
- `post_votes` : Votes up/down
- `moderation_queue` : File de modération

---

## 5. Processus de développement

### 5.1 Organisation des fichiers

```
projectcineavis/
├── docs/                       # Documentation fonctionnelle
│   ├── 01-vision-generale.md   # Vision et principes
│   ├── 02-fonctionnel-detaille.md  # Specs fonctionnelles
│   ├── 03-technique.md         # Architecture technique
│   ├── 04-roadmap.md           # Planning 6 mois
│   └── *.svg                   # Diagrammes (markmap)
├── specs/                      # Spécifications techniques
│   ├── api-specs.md            # Specs API (à compléter)
│   └── user-flows.md           # Parcours utilisateurs (à compléter)
├── README.md                   # Point d'entrée
└── AGENTS.md                   # Ce fichier
```

**Note importante** : Le projet est actuellement en phase de planning. Aucun code n'a encore été écrit. Les répertoires suivants seront créés prochainement :
- `cinema-core/` : Backend Go
- `cinema-web/` : Frontend Next.js
- `docker-compose.yml` : Environnement de développement

### 5.2 Phases de développement (Roadmap)

| Phase | Période | Focus | Livrable principal |
|-------|---------|-------|-------------------|
| **Phase 1** | Mois 1-2 | Fondations Data | Pipeline ingestion + génération IA |
| **Phase 2** | Mois 3-4 | Socle Technique | API + site minimal fonctionnel |
| **Phase 3** | Mois 5-6 | Social & Lancement | Notations, feeds, beta publique |

### 5.3 Principes de développement

1. **Étape par étape** — on valide le data avant le social
2. **Supervision humaine** — l'IA génère, l'humain valide
3. **Qualité > Quantité** — mieux vaut 10k fiches propres que 100k bancales
4. **Open source d'esprit** — données libres, transparence
5. **Vitesse contrôlée** — Cursor accélère, mais on garde la main

---

## 6. Conventions de code

### 6.1 Langue

- **Documentation** : Français (tous les documents dans `docs/` sont en français)
- **Code** : Anglais (comme standard international)
- **Commentaires** : Anglais recommandé, français accepté pour clarifications métier
- **Commits** : Anglais recommandé, messages clairs et descriptifs

### 6.2 Go (Backend)

- Structure standard Go : `cmd/`, `internal/`, `pkg/`
- Pattern repository pour l'accès données
- Interface pour les services (testabilité)
- Logs structurés JSON
- Gestion explicite des erreurs

### 6.3 TypeScript/Next.js (Frontend)

- App Router de Next.js 14
- Tailwind CSS pour le styling
- Components fonctionnels avec hooks
- Types stricts TypeScript

---

## 7. API Endpoints

### 7.1 API Publique (v1)

```
GET    /v1/works?type=film&page=1
GET    /v1/works/:id
GET    /v1/works/:id/cast
GET    /v1/works/:id/crew
GET    /v1/works/search?q=dune

GET    /v1/persons/:id
GET    /v1/persons/:id/filmography

POST   /v1/auth/oauth/:provider
POST   /v1/auth/callback

GET    /v1/works/:id/my-rating      # auth requis
POST   /v1/works/:id/rate           # auth requis

GET    /v1/feeds/:type/:id
POST   /v1/posts                    # auth requis
POST   /v1/posts/:id/vote           # auth requis
```

### 7.2 API Interne (service-to-service)

```
GET    /internal/works/:id
GET    /internal/works/batch
GET    /internal/persons/:id
```

---

## 8. Stratégie de test

### 8.1 Niveaux de test

- **Unit tests** : Logique métier, services
- **Integration tests** : Repositories, API endpoints
- **E2E tests** : Parcours utilisateurs critiques (Playwright/Cypress)

### 8.2 Données de test

- 100 œuvres pour les tests initiaux
- 1000 œuvres pour valider le pipeline
- 10k œuvres pour la phase 1 complète

### 8.3 Critères de validation

- Couverture > 70% pour le code critique
- Tous les endpoints API testés
- Temps de réponse p95 < 200ms

---

## 9. Sécurité

| Couche | Mesure |
|--------|--------|
| Transport | HTTPS TLS 1.3, HSTS |
| Auth | OAuth 2.0 + JWT (15min access, 7j refresh) |
| API | Rate limit 100/min anon, 1000/min auth |
| Données | Tokens chiffrés en DB (AES-256) |
| Headers | CORS strict, CSP, X-Frame-Options |
| Secrets | `.env`, jamais commité |

---

## 10. Monitoring et observabilité

### 10.1 Logs structurés (JSON)

```json
{
  "timestamp": "2026-02-19T12:00:00Z",
  "level": "error",
  "service": "core-service",
  "request_id": "uuid",
  "message": "wikidata fetch failed",
  "work_id": "uuid",
  "error": "timeout",
  "attempt": 2,
  "max_attempts": 3
}
```

### 10.2 Health check

```
GET /health
{
  "status": "healthy",
  "services": {
    "postgres": "up",
    "redis": "up",
    "meilisearch": "up"
  },
  "version": "1.0.0"
}
```

---

## 11. Ressources et documentation

### Documentation existante

- [Vision Générale](docs/01-vision-generale.md) — Vision, problème, solution, différenciateurs
- [Fonctionnel Détaillé](docs/02-fonctionnel-detaille.md) — Parcours utilisateurs, écrans, système de notation
- [Technique](docs/03-technique.md) — Stack, architecture, schéma DB, APIs
- [Roadmap](docs/04-roadmap.md) — Planning détaillé sur 6 mois

### Glossaire

| Terme | Définition |
|-------|------------|
| **CC0** | Licence Creative Commons Zero — domaine public |
| **Canonique** | Source de vérité unique, non dupliquée |
| **Core** | Partie data/infrastructure du produit |
| **Social** | Partie communautaire/engagement |
| **Feed** | Fil d'actualité thématique ou global |
| **Notation multi-critères** | Évaluation sur 6 dimensions indépendantes (1-10) |
| **Revendication** | Processus pour obtenir droits d'édition sur une fiche |

---

## 12. Notes pour les agents IA

### Quand vous travaillez sur ce projet :

1. **Lisez d'abord la documentation** dans `docs/` pour comprendre le contexte
2. **Respectez la langue** : Français pour docs, Anglais pour code
3. **Suivez la roadmap** : Ne sautez pas d'étapes, validez chaque phase
4. **Qualité avant quantité** : Mieux vaut moins mais propre
5. **Supervisez l'IA** : Toute génération IA doit être validée

### Anti-patterns à éviter :

- Ne pas coder de fonctionnalités de la Phase 3 alors qu'on est en Phase 1
- Ne pas ignorer la traçabilité des données (toujours lier à la source)
- Ne pas hardcoder de clés API ou secrets
- Ne pas négliger les retries et la gestion d'erreurs

### Points de vigilance :

- Wikidata peut être indisponible → implémenter retries avec backoff
- OpenAI API peut être lente → utiliser des queues asynchrones
- Meilisearch peut tomber → fallback vers PostgreSQL LIKE
- Redis peut tomber → auth sans rate limit, pas de cache

---

*Ce document est vivant. Mettez-le à jour quand l'architecture ou les conventions évoluent.*
