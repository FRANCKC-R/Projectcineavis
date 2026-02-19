# Documentation Technique

Ce document décrit l'architecture technique et les choix technologiques.

# FICHE PROJET — Technique (v2)

**Projet :** Cinema Data Core  
**Date :** 2026-02-19  
**Version :** 2.0  
**Statut :** À valider

---

## 1. Stack technologique

| Couche | Technologie | Pourquoi |
|--------|-------------|----------|
| Frontend | Next.js 14 (App Router) | SSR, SEO, DX |
| Backend API | Go (Echo) | Performance, concurrence, binaire statique |
| Database | PostgreSQL 15 | Relations + JSONB flexible |
| Cache | Redis | Sessions, rate limit, cache API |
| Search | Meilisearch | Full-text, faceting, léger |
| Queue | PostgreSQL + Go workers | Pas de complexité Kafka/RabbitMQ |
| Auth | OAuth 2.0 (Google, Apple, X) | Pas de gestion mots de passe |
| IA | OpenAI API | Génération descriptions |
| Hosting | VPS 8 Go RAM | Souveraineté, coût maîtrisé |

---

## 2. Architecture système

### 2.1 Vue globale
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
│                                                             │
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
│  │                                                          │
│  │  ┌─────────────────────────────────────────┐             │
│  │  │      WORKERS (Go, background)          │             │
│  │  │  • Ingestion Wikidata (cron)            │             │
│  │  │  • Génération IA (queue)                │             │
│  │  │  • Notifications (webhook)              │             │
│  │  │  • Retry failed jobs                    │             │
│  │  └─────────────────────────────────────────┘             │
│  └─────────────────────────────────────────────────────────┘
│                                                             │
└─────────────────────────────────────────────────────────────┘
plain


### 2.2 Résilience — Gestion des pannes

| Composant | Si ça tombe | Comportement |
|-----------|-------------|--------------|
| **Wikidata API** | Retry 3x, backoff exponentiel | Skip batch, log, retry next cron |
| **OpenAI API** | Queue message, retry toutes les 5min | Fallback : marquer "description pending" |
| **Meilisearch** | Circuit breaker ouvert | API continue, search dégradé (PostgreSQL LIKE) |
| **Redis** | Circuit breaker ouvert | Auth sans rate limit, pas de cache |
| **PostgreSQL** | — | Health check fail, alerte, maintenance |

---

## 3. Schéma de données

### 3.1 Core Database

```sql
-- Œuvres canoniques
CREATE TABLE works (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    work_type VARCHAR(20) NOT NULL CHECK (work_type IN ('film', 'series', 'short', 'documentary', 'animation')),
    canonical_title VARCHAR(500) NOT NULL,
    original_title VARCHAR(500),
    release_date DATE,
    duration_minutes INT,
    plot_summary TEXT,
    plot_summary_generated BOOLEAN DEFAULT true,
    quality_score INT CHECK (quality_score BETWEEN 0 AND 100),
    status VARCHAR(20) DEFAULT 'processing' CHECK (status IN ('processing', 'pending_review', 'published', 'error')),
    error_message TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    published_at TIMESTAMPTZ
);

CREATE INDEX idx_works_status ON works(status);
CREATE INDEX idx_works_type_date ON works(work_type, release_date);

-- Traçabilité sources
CREATE TABLE work_sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    work_id UUID NOT NULL REFERENCES works(id) ON DELETE CASCADE,
    source VARCHAR(50) NOT NULL DEFAULT 'wikidata',
    external_id VARCHAR(100) NOT NULL,
    source_url TEXT,
    fetched_at TIMESTAMPTZ DEFAULT NOW(),
    raw_data JSONB,
    fetch_attempts INT DEFAULT 1,
    last_error TEXT,
    UNIQUE(work_id, source)
);

-- Facts extraits (source unique Wikidata)
CREATE TABLE work_facts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    work_id UUID NOT NULL REFERENCES works(id) ON DELETE CASCADE,
    field_name VARCHAR(100) NOT NULL,
    field_value JSONB NOT NULL,
    extracted_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(work_id, field_name)
);

-- Générations IA (historisées)
CREATE TABLE work_generations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    work_id UUID NOT NULL REFERENCES works(id) ON DELETE CASCADE,
    content_type VARCHAR(50) NOT NULL DEFAULT 'description',
    content TEXT NOT NULL,
    model VARCHAR(50),
    prompt_hash VARCHAR(64),
    generation_params JSONB,
    quality_score INT,
    status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'generated', 'reviewed', 'rejected')),
    reviewed_by UUID,
    attempts INT DEFAULT 0,
    last_error TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    generated_at TIMESTAMPTZ,
    reviewed_at TIMESTAMPTZ
);

CREATE INDEX idx_generations_status ON work_generations(status);

-- Images (liens)
CREATE TABLE work_images (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    work_id UUID NOT NULL REFERENCES works(id) ON DELETE CASCADE,
    image_type VARCHAR(50) NOT NULL,
    source VARCHAR(50) NOT NULL,
    source_url TEXT NOT NULL,
    license VARCHAR(100),
    width INT,
    height INT,
    is_primary BOOLEAN DEFAULT false,
    fetched_at TIMESTAMPTZ DEFAULT NOW(),
    fetch_status VARCHAR(20) DEFAULT 'ok' CHECK (fetch_status IN ('ok', 'broken', 'unreachable'))
);

-- Personnes
CREATE TABLE persons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    full_name VARCHAR(300) NOT NULL,
    birth_date DATE,
    death_date DATE,
    nationality VARCHAR(100),
    biography TEXT,
    biography_generated BOOLEAN DEFAULT false,
    primary_profession VARCHAR(100),
    quality_score INT CHECK (quality_score BETWEEN 0 AND 100),
    status VARCHAR(20) DEFAULT 'processing',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Liens œuvre-personne
CREATE TABLE crew_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    work_id UUID NOT NULL REFERENCES works(id) ON DELETE CASCADE,
    person_id UUID NOT NULL REFERENCES persons(id) ON DELETE CASCADE,
    role_category VARCHAR(50) NOT NULL CHECK (role_category IN ('cast', 'crew')),
    role_name VARCHAR(100) NOT NULL,
    role_details VARCHAR(200),
    billing_order INT,
    UNIQUE(work_id, person_id, role_name)
);

CREATE INDEX idx_crew_work ON crew_links(work_id);
CREATE INDEX idx_crew_person ON crew_links(person_id);
3.2 Social Database
sql

-- Utilisateurs
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE,
    username VARCHAR(50) UNIQUE NOT NULL,
    display_name VARCHAR(100),
    avatar_url TEXT,
    bio TEXT,
    is_verified BOOLEAN DEFAULT false,
    role VARCHAR(20) DEFAULT 'user' CHECK (role IN ('user', 'moderator', 'admin')),
    reputation_score INT DEFAULT 0,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    last_login_at TIMESTAMPTZ
);

-- OAuth
CREATE TABLE user_oauth (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    provider VARCHAR(20) NOT NULL,
    provider_user_id VARCHAR(100) NOT NULL,
    provider_email VARCHAR(255),
    access_token_encrypted TEXT,
    refresh_token_encrypted TEXT,
    expires_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(provider, provider_user_id)
);

-- Notations multi-critères
CREATE TABLE ratings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    work_id UUID NOT NULL,
    rating_scenario INT CHECK (rating_scenario BETWEEN 1 AND 10),
    rating_direction INT CHECK (rating_direction BETWEEN 1 AND 10),
    rating_acting INT CHECK (rating_acting BETWEEN 1 AND 10),
    rating_visuals INT CHECK (rating_visuals BETWEEN 1 AND 10),
    rating_sound INT CHECK (rating_sound BETWEEN 1 AND 10),
    rating_technical INT CHECK (rating_technical BETWEEN 1 AND 10),
    weighted_average DECIMAL(3,1) GENERATED ALWAYS AS (
        (COALESCE(rating_scenario, 0) * 0.20 +
         COALESCE(rating_direction, 0) * 0.20 +
         COALESCE(rating_acting, 0) * 0.20 +
         COALESCE(rating_visuals, 0) * 0.15 +
         COALESCE(rating_sound, 0) * 0.15 +
         COALESCE(rating_technical, 0) * 0.10) /
        NULLIF((rating_scenario IS NOT NULL)::int * 0.20 +
               (rating_direction IS NOT NULL)::int * 0.20 +
               (rating_acting IS NOT NULL)::int * 0.20 +
               (rating_visuals IS NOT NULL)::int * 0.15 +
               (rating_sound IS NOT NULL)::int * 0.15 +
               (rating_technical IS NOT NULL)::int * 0.10, 0)
    ) STORED,
    comment TEXT,
    contains_spoiler BOOLEAN DEFAULT false,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(user_id, work_id)
);

CREATE INDEX idx_ratings_work ON ratings(work_id);
CREATE INDEX idx_ratings_user ON ratings(user_id);

-- Posts
CREATE TABLE posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    author_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    feed_type VARCHAR(50) NOT NULL,
    feed_id UUID,
    post_type VARCHAR(50) NOT NULL,
    content TEXT NOT NULL,
    rating_id UUID REFERENCES ratings(id),
    work_id UUID,
    vote_score INT DEFAULT 0,
    reply_count INT DEFAULT 0,
    is_pinned BOOLEAN DEFAULT false,
    is_deleted BOOLEAN DEFAULT false,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_posts_feed ON posts(feed_type, feed_id, created_at DESC);
CREATE INDEX idx_posts_author ON posts(author_id, created_at DESC);

-- Votes
CREATE TABLE post_votes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    vote_value INT NOT NULL CHECK (vote_value IN (-1, 1)),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(post_id, user_id)
);

-- Modération
CREATE TABLE moderation_queue (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ticket_type VARCHAR(50) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'assigned', 'in_review', 'approved', 'rejected')),
    priority VARCHAR(20) DEFAULT 'normal' CHECK (priority IN ('low', 'normal', 'high', 'urgent')),
    submitted_by UUID NOT NULL REFERENCES users(id),
    assigned_to UUID REFERENCES users(id),
    target_type VARCHAR(50),
    target_id UUID,
    description TEXT NOT NULL,
    evidence JSONB,
    resolution_notes TEXT,
    resolved_by UUID REFERENCES users(id),
    resolved_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_moderation_status ON moderation_queue(status, priority, created_at);
4. Stratégie de cache
Table

Donnée	Cache	TTL	Invalidation
Fiche œuvre	Redis	1 heure	Publication nouvelle version
Résultats recherche	Meilisearch	—	Réindexation explicite
Session utilisateur	Redis	24 heures	Déconnexion
Rate limit	Redis	1 minute	—
Feed global	Redis	5 minutes	Nouveau post
Feed œuvre	Redis	2 minutes	Nouveau post sur œuvre
5. Pipeline de données (asynchrone)
plain

┌─────────────────┐
│  SCHEDULER      │ ← Cron toutes les 6 heures
│  (ingestion)    │
└────────┬────────┘
         ▼
┌─────────────────┐
│  FETCH WIKIDATA │ ← Retry 3x, backoff
│  (worker)       │
└────────┬────────┘
         ▼
┌─────────────────┐
│  NORMALIZE      │ ← Extraction facts
│  (worker)       │
└────────┬────────┘
         ▼
┌─────────────────┐
│  QUALITY SCORE  │ ← Si < 70 → flag humain
│  (worker)       │
└────────┬────────┘
         ▼
┌─────────────────┐
│  QUEUE GENERATE │ ← PostgreSQL queue table
│  (async)        │
└────────┬────────┘
         ▼
┌─────────────────┐
│  GENERATE IA    │ ← Retry si fail
│  (worker)       │
└────────┬────────┘
         ▼
┌─────────────────┐
│  QUALITY CHECK  │ ← Scoring interne
│  (worker)       │
└────────┬────────┘
         ▼
┌─────────────────┐
│  PUBLISH        │ ← Status = published
│  (worker)       │ ← Index Meilisearch
└─────────────────┘
6. Monitoring & Observabilité
6.1 Logs structurés (JSON)
JSON

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
6.2 Métriques clés
Table

Métrique	Où	Alert si
Request latency p99	API Gateway	> 500ms
Error rate	Tous services	> 1%
Queue depth	Generation queue	> 1000
Failed jobs	Workers	> 10/h
DB connections	PostgreSQL	> 80%
Disk usage	VPS	> 80%
6.3 Health checks
plain

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
7. Backup & Recovery
Table

Élément	Fréquence	Méthode	RTO	RPO
PostgreSQL	Quotidien 3h	pg_dump + rsync	4h	24h
Redis	—	Pas critique (cache)	—	—
Meilisearch	Hebdo	Rebuild from DB	2h	7j
Code	Push	Git	—	—
8. Sécurité
Table

Couche	Mesure
Transport	HTTPS TLS 1.3, HSTS
Auth	OAuth 2.0 + JWT (15min access, 7j refresh)
API	Rate limit 100/min anon, 1000/min auth
Données	Tokens chiffrés en DB (AES-256)
Headers	CORS strict, CSP, X-Frame-Options
Secrets	.env, jamais commité
9. APIs
9.1 Publique
plain

GET    /v1/works?type=film&page=1
GET    /v1/works/:id
GET    /v1/works/:id/cast
GET    /v1/works/:id/crew
GET    /v1/works/search?q=dune

GET    /v1/persons/:id
GET    /v1/persons/:id/filmography

POST   /v1/auth/oauth/:provider
POST   /v1/auth/callback

GET    /v1/works/:id/my-rating      # auth
POST   /v1/works/:id/rate           # auth

GET    /v1/feeds/:type/:id
POST   /v1/posts                    # auth
POST   /v1/posts/:id/vote           # auth
9.2 Interne (service-to-service)
plain

GET    /internal/works/:id
GET    /internal/works/batch
GET    /internal/persons/:id