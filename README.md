# Chômage Tracker

Outil personnel de veille et de suivi de recherche d'emploi : il récupère chaque jour les offres correspondant à mon profil, les classe selon mes critères avec une justification lisible, et suit l'état de mes candidatures pour savoir qui relancer.

> **Statut : en cours de développement (V1).** Projet personnel, non commercial, réalisé dans le cadre de ma recherche d'emploi et comme projet portfolio.

---

## Fonctionnalités

| Version | Fonctionnalité | Statut |
|---|---|---|
| V1 | Ingestion quotidienne des offres (multi-sources, normalisation, déduplication) | En cours |
| V1 | Parsing du CV en profil structuré | À faire |
| V1 | Scoring des offres selon des critères pondérés, avec justification | À faire |
| V1 | Dashboard des offres | À faire |
| V2 | Suivi des candidatures via Gmail (lecture seule) : réponses, refus, entretiens | À faire |
| V2 | Détection des relances à faire et génération de brouillons (jamais envoyés automatiquement) | À faire |
| V3 | Chat RAG sur les offres et candidatures | À faire |
| V3 | Aide à la rédaction de lettres de motivation | À faire |
| V3 | Orchestration du flux quotidien par un agent LangGraph | À faire |
| V4 | Évaluation (RAGAS), observabilité (Langfuse), documentation | À faire |

---

## Architecture

```
Sources (Adzuna, France Travail*)    CV (PDF)    Gmail (lecture seule)
              │                          │                │
              ▼                          ▼                ▼
   ┌─────────────────────── Pipeline quotidien (Airflow) ──────────────────────┐
   │  Ingestion des offres   │    Parsing du CV     │    Lecture des mails     │
   │  normalise, déduplique  │   profil structuré   │    classe, rattache      │
   └───────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
                        PostgreSQL + pgvector (local)
                                     │
                                     ▼
   ┌─────────────────── Couche IA (LangChain / LangGraph) ─────────────────────┐
   │    Matching hybride     │      Relances        │        Chat RAG          │
   │  BM25 + vecteurs + LLM  │   brouillons J+7     │    questions, lettres    │
   └───────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
                       API FastAPI  ──►  Front Vue 3
```

\* sous réserve d'obtention de l'accès à l'API.

### Principe du scoring

Pour limiter le coût de calcul, le LLM n'est appelé que sur une sélection réduite d'offres :

1. **Filtres SQL** sur les critères stricts (localisation, type de contrat, salaire).
2. **Recherche hybride** (BM25 + similarité vectorielle) entre le profil et les offres restantes.
3. **Scoring par LLM** des 20 à 30 meilleures offres, avec une justification lisible.

---

## Stack technique

- **Langage** : Python 3.12
- **API** : FastAPI
- **Base de données** : PostgreSQL 16 + pgvector
- **ORM et migrations** : SQLAlchemy, Alembic
- **Orchestration** : Apache Airflow
- **IA** : LangChain, LangGraph, Ollama (Mistral 7B), modèle d'embeddings local
- **Front** : Vue 3
- **Infrastructure** : Docker Compose
- **Évaluation et observabilité** : RAGAS, Langfuse

---

## Usage des données

Ce projet est un **outil strictement personnel et non commercial**. Il n'a qu'un seul utilisateur (son auteur) et n'est pas mis en ligne publiquement.

### Sources

| Source | Données | Conditions |
|---|---|---|
| [Adzuna API](https://developer.adzuna.com/) | Offres d'emploi | [Conditions d'utilisation Adzuna](https://www.adzuna.fr/terms-and-conditions.html) |
| [API Offres d'emploi – France Travail](https://francetravail.io/) | Offres d'emploi | [CGU francetravail.io](https://francetravail.io/cgu), licence de réutilisation (accès en cours de demande) |
| Gmail API (V2) | Métadonnées des mails liés à mes candidatures | Accès OAuth en lecture seule à ma propre boîte |

### Engagements

- **Aucune rediffusion** : les offres sont affichées uniquement dans l'interface locale. Elles ne sont ni publiées, ni revendues, ni partagées à des tiers.
- **Lien vers la source** : chaque offre conserve un lien vers l'annonce d'origine.
- **Volumétrie limitée** : une collecte par jour, de l'ordre de 10 à 20 requêtes, dans le respect des limites de chaque API.
- **Conservation limitée** : les offres sont supprimées automatiquement 30 jours après leur expiration.
- **Traitement IA 100 % local** : le scoring et les embeddings sont calculés sur ma machine via Ollama (Mistral 7B). Aucune donnée issue des API n'est envoyée à un service d'IA tiers.
- **Pas d'entraînement** : aucune offre n'est utilisée pour entraîner ou affiner un modèle.
- **Mails** : seules les métadonnées et la catégorie (accusé de réception, refus, entretien…) sont stockées, pas le contenu complet. Aucun mail n'est envoyé automatiquement.
- **Secrets** : les identifiants d'API sont stockés dans un fichier `.env` local, exclu du dépôt.
- **Pas de scraping** de sites dont les conditions d'utilisation l'interdisent (LinkedIn notamment).

Le code de ce dépôt est public. Les données collectées ne le sont pas et ne sont jamais versionnées.

---

## Installation

> À venir avec la première version fonctionnelle.

Prérequis prévus : Docker et Docker Compose, Ollama, et des identifiants API Adzuna (et France Travail si disponibles).

```bash
git clone https://github.com/ShinoTetsuny/Chomage-Tracker.git
cd chomage-tracker
cp .env.example .env   # renseigner les identifiants d'API
docker compose up -d
```

---

## Auteur

**Tom Ollivier** – développeur fullstack, Mastère Big Data & IA (IPSSI)
[LinkedIn](https://www.linkedin.com/in/tom-ollivier-01502023b/) · [GitHub](https://github.com/ShinoTetsuny)

## Licence

Code sous licence MIT. Les données issues des API restent soumises aux conditions de leurs fournisseurs respectifs.