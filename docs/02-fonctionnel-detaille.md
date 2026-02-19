# Fonctionnel Détaillé

Ce document décrit les spécifications fonctionnelles détaillées.


ŒUVRE (film, série, court, doc, animation)
├── Fiche canonique (data)
├── Évaluations communautaires
├── Feed associé
└── Relations (suite, remake, même univers)
PERSONNE (acteur, réalisateur, technicien)
├── Fiche canonique
├── Filmographie
├── Rôles techniques détaillés
└── Page profil (v2 portfolio)
UTILISATEUR
├── Profil public/privé
├── Historique (vu, à voir, noté)
├── Préférences
└── Activité (posts, votes, commentaires)
FEED
├── Type : œuvre | événement | thème | persistant
├── Durée : temporaire | événementiel | permanent
└── Contenu : posts, avis, débats, actualités
plain


---

## 2. Parcours utilisateurs

### 2.1 Parcours 1 : Premier visiteur (non connecté)

| Étape | Action | Écran |
|-------|--------|-------|
| 1 | Arrive sur landing | Home |
| 2 | Recherche un film | Barre recherche → Résultats |
| 3 | Consulte fiche film | Fiche œuvre complète |
| 4 | Lit avis/feed | Feed œuvre (lecture seule) |
| 5 | Veut noter | CTA "Noter" → Auth requise |
| 6 | Crée compte | Modal auth (Google/Apple/X) |

### 2.2 Parcours 2 : Utilisateur connecté (notation)

| Étape | Action | Écran/Détail |
|-------|--------|--------------|
| 1 | Connecté, arrive sur home | Feed personnalisé (chrono) |
| 2 | Clique sur film dans feed | Fiche œuvre |
| 3 | Clique "Noter" | Modal notation multi-critères |
| 4 | Note 6 critères (1-10) | Sliders + commentaire optionnel |
| 5 | Valide | Note enregistrée, moyenne recalculée |
| 6 | Partage avis | Post automatique dans feed œuvre |

### 2.3 Parcours 3 : Recherche technique (professionnel)

| Étape | Action | Écran |
|-------|--------|-------|
| 1 | Recherche par métier | "monteur" + filtres |
| 2 | Liste monteurs | Résultats avec filmographie |
| 3 | Clique monteur | Fiche personne |
| 4 | Voir filmographie détaillée | Liste œuvres + rôle technique |
| 5 | Clique œuvre | Fiche œuvre avec crew complet |

### 2.4 Parcours 4 : Studio / Production (revendication)

| Étape | Action | Détail |
|-------|--------|--------|
| 1 | Studio découvre fiche son film | Fiche œuvre |
| 2 | Clique "Revendiquer cette fiche" | Formulaire demande |
| 3 | Fournit preuves | Documents, liens officiels |
| 4 | Soumet | Ticket créé, file modération |
| 5 | Validation manuelle | Admin vérifie, approuve/refuse |
| 6 | Si approuvé | Accès édition limitée (corrections) |

---

## 3. Écrans détaillés

### 3.1 HOME (Landing + Connecté)

**Layout :**
┌─────────────────────────────────────────┐
│  LOGO    [Recherche...]    [Connexion]  │
├─────────────────────────────────────────┤
│                                         │
│  [FILTRE: Tous | Films | Séries | ...]  │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │  FEED PRINCIPAL                 │   │
│  │  • Post utilisateur             │   │
│  │  • Nouvelle fiche ajoutée       │   │
│  │  • Débat en tendance            │   │
│  │  • ...                          │   │
│  └─────────────────────────────────┘   │
│                                         │
│  [+ Nouveau post]  [Découvrir]          │
│                                         │
└─────────────────────────────────────────┘
plain


**Comportements :**
- Non connecté : feed global chronologique, CTA connexion
- Connecté : feed personnalisé (suivis + tendances)
- Infinite scroll
- Pull-to-refresh

### 3.2 FICHE ŒUVRE

**Sections (ordre vertical) :**

1. **Header**
   - Poster (lien Wikidata)
   - Titre (canonical + original)
   - Année, durée, pays
   - Genres (tags)
   - Note globale communautaire (moyenne pondérée)
   - Bouton "Noter" | "Vu" | "À voir"

2. **Synopsis**
   - Description générée par IA
   - Mention "Généré par IA" (interne uniquement)

3. **Cast principal** (5-6 têtes d'affiche)
   - Photo + nom + rôle
   - "Voir tout" → section Cast complète

4. **Crew clé**
   - Réalisateur
   - Scénariste
   - DOP (director of photography)
   - Monteur
   - Compositeur
   - "Voir tout l'équipe technique" → section Crew

5. **Feed œuvre** (onglet par défaut)
   - Posts récents
   - Avis notés
   - Débats
   - Input "Partager votre avis"

6. **Informations** (onglet)
   - Dates de sortie par pays
   - Box-office (si dispo)
   - Versions (director's cut, etc.)
   - Langues, formats

7. **Similaires** (onglet)
   - Même réalisateur
   - Même genre
   - Même décennie

### 3.3 MODAL NOTATION

**Titre :** "Noter [Titre du film]"

**Contenu :**
┌─────────────────────────────────────────┐
│  Scénario                              │
│  [1]——[2]——[3]——[4]——[5]——[6]——[7]——[8]——[9]——[10]  │
│  Pas convaincu                    Chef-d'œuvre        │
│                                         │
│  Réalisation                           │
│  [1]————————————————————————————[10]   │
│                                         │
│  Interprétation                        │
│  [1]————————————————————————————[10]   │
│                                         │
│  Image (photographie)                  │
│  [1]————————————————————————————[10]   │
│                                         │
│  Son / Musique                         │
│  [1]————————————————————————————[10]   │
│                                         │
│  Technique (montage, VFX, déco...)     │
│  [1]————————————————————————————[10]   │
│                                         │
│  [Optionnel] Commentaire               │
│  [________________________]            │
│                                         │
│         [Annuler]  [Valider]           │
└─────────────────────────────────────────┘
plain


**Comportement :**
- Slider 1-10 avec labels contextuels
- Minimum 3 critères notés pour valider
- Moyenne calculée en temps réel
- Post automatique dans feed si commentaire renseigné

### 3.4 FICHE PERSONNE

**Sections :**

1. **Header**
   - Photo (Wikidata)
   - Nom complet
   - Métier principal
   - Date naissance/décès
   - Nationalité

2. **Biographie**
   - Générée IA (si assez de sources)
   - Ou texte court Wikidata

3. **Filmographie** (onglet par défaut)
   - Tableau : Année | Titre | Rôle
   - Filtre par type (film, série...)
   - Filtre par métier (réal, acteur, monteur...)

4. **Technique** (onglet, si applicable)
   - Liste œuvres où rôle technique
   - Spécialisation (VFX, montage son, etc.)

5. **Collaborations fréquentes**
   - "Souvent travaille avec..."
   - Graphique liens (v2)

### 3.5 PROFIL UTILISATEUR

**Sections :**

1. **Header**
   - Avatar
   - Pseudo
   - Bio optionnelle
   - Membre depuis

2. **Statistiques**
   - Films notés
   - Heures de cinéma
   - Genres préférés (calculé)
   - Moyenne des notes données

3. **Activité** (onglet)
   - Posts
   - Commentaires
   - Évaluations

4. **Listes** (onglet, v2)
   - "Mes favoris"
   - "À voir"
   - Listes perso

5. **Paramètres** (onglet)
   - Notifications
   - Confidentialité
   - Auth liées (Google, Apple, X)
   - Suppression compte

---

## 4. Système de feeds

### 4.1 Types de feeds

| Type | Description | Durée | Exemples |
|------|-------------|-------|----------|
| **Éphémère** | Lié à une sortie | 2-4 semaines | "Sortie Dune 2" |
| **Événementiel** | Festival, cérémonie | Durée événement + 1 mois | "Cannes 2024" |
| **Thématique** | Sujet transversal | Permanent | "Science-fiction", "Nolan" |
| **Œuvre** | Une fiche = un feed | Vie de l'œuvre | Page film |
| **Global** | Toute l'activité | Permanent | Home |

### 4.2 Contenu d'un feed

**Types de posts :**
- Avis structuré (avec notes)
- Commentaire libre
- Partage de fiche
- Question / débat
- Actualité (auto : nouvelle fiche ajoutée)

**Interactions :**
- Upvote / Downvote
- Répondre (1 niveau de réponse)
- Signaler
- Partager

### 4.3 Algorithmie de feed (v1 simple)

**Home non connecté :**
1. Chronologique
2. Boost si tendance (votes récents)

**Home connecté :**
1. Posts des utilisateurs suivis
2. Posts sur œuvres notées/vues
3. Tendances globales
4. Découvertes (aléatoire pondéré)

**Feed œuvre :**
1. Chronologique pur
2. Pin les meilleurs avis (score communautaire)

---

## 5. Système de notation

### 5.1 Critères d'évaluation

| # | Critère | Description |
|---|---------|-------------|
| 1 | **Scénario** | Histoire, structure, dialogues |
| 2 | **Réalisation** | Mise en scène, rythme, choix artistiques |
| 3 | **Interprétation** | Jeu des acteurs, direction d'acteurs |
| 4 | **Image** | Photographie, direction artistique, cadre |
| 5 | **Son / Musique** | Bande-son, bruitages, mixage |
| 6 | **Technique** | Montage, VFX, costumes, maquillage, déco |

### 5.2 Calcul de la note globale
Note globale = moyenne pondérée des critères notés
Si utilisateur note tous les critères :
Scénario : 20%
Réalisation : 20%
Interprétation : 20%
Image : 15%
Son : 15%
Technique : 10%
Si utilisateur note partiellement :
Moyenne simple des critères renseignés
plain


### 5.3 Agrégation communautaire
Note œuvre = moyenne pondérée des notes utilisateurs
Pondération par utilisateur :
Nouveau : poids 1
Actif (10+ notes) : poids 1.2
Expert (100+ notes, variété) : poids 1.5
plain


---

## 6. Gestion des données

### 6.1 Cycle de vie d'une fiche
┌─────────────────┐
│  INGESTION      │ ← Wikidata fetch
│  (automatique)  │
└────────┬────────┘
▼
┌─────────────────┐
│  NORMALISATION  │ ← Structuration
│  (automatique)  │
└────────┬────────┘
▼
┌─────────────────┐
│  SCORING        │ ← Qualité données
│  (automatique)  │
└────────┬────────┘
▼
┌─────────────────┐
│  GENERATION IA  │ ← Description
│  (supervisé)    │
└────────┬────────┘
▼
┌─────────────────┐
│  PUBLICATION    │ ← Visible
│  (manuel ou     │   si score OK)
│  auto)          │
└─────────────────┘
plain


### 6.2 Revendication et correction

**Acteurs concernés :**
- Studios
- Distributeurs
- Professionnels (pour leur fiche)
- Producteurs de courts-métrages

**Processus :**
1. Formulaire demande (type, preuves, contact)
2. File modération (humain)
3. Vérification identité (documents)
4. Approbation / Refus / Demande compléments
5. Si approuvé : droits d'édition limités

**Droits d'édition :**
- Corriger des erreurs factuelles
- Ajouter des liens officiels
- Demander ajout image (soumis à validation)
- Ne peut PAS : modifier le cast canonique, supprimer la fiche

---

## 7. Administration et modération

### 7.1 Rôles admin

| Rôle | Droits |
|------|--------|
| **Super Admin** | Tout |
| **Modérateur** | Valider fiches, modérer posts, gérer tickets |
| **Vérificateur** | Traiter demandes revendication |

### 7.2 File de modération

**Tickets entrants :**
- Revendication fiche
- Signalement contenu (post, avis)
- Demande ajout œuvre (si pas dans Wikidata)
- Correction proposée

**Workflow :**
Nouveau → Assigné → En cours → Résolu (Approuvé/Refusé)
↓
Escalade (si doute)
plain


### 7.3 Règles de modération

**Contenu refusé :**
- Spoilers non balisés
- Haine, discrimination
- Spam, promotion non déclarée
- Faux avis (coordinateur détecte patterns)

**Sanctions :**
- 1er : avertissement
- 2e : suspension 7j
- 3e : suspension 30j
- 4e : bannissement

---

## 8. Glossaire fonctionnel

| Terme | Définition |
|-------|------------|
| **Feed** | Fil d'actualité thématique ou global |
| **Fiche** | Page canonique d'une œuvre ou personne |
| **Notation multi-critères** | Évaluation sur 6 dimensions indépendantes |
| **Revendication** | Processus pour obtenir droits d'édition sur une fiche |
| **Scoring qualité** | Évaluation interne de la fiabilité des données |
| **Feed éphémère** | Feed temporaire lié à une sortie |
| **Feed événementiel** | Feed lié à un festival ou cérémonie |

---

**Prochaine étape :** Fiche 03 — Technique (stack, architecture, schéma DB, APIs)
