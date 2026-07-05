# Bluesky Machine Translation Discourse

Web and Social Media Search and Analysis — course project (UniMiB, BSc AI).

## Objective

We study how Bluesky discourse frames machine translation (MT): as a useful
tool, a threat to translators' livelihoods, or both. Posts are collected
around MT-related queries and hashtags (`machine translation`, `AI
translation`, `#localization`, `DeepL`, `MTPE`, `post-editing`, ...) and
authors are labeled `general` / `professional` / `tech` based on their bio,
to compare how people with different relationships to translation talk
about it.

**Research questions:**
1. Sentiment & framing — is MT discussed as a tool, a threat, or both?
2. Professional impact discourse — how do working translators talk about
   MT's effect on their careers, compared to the general public?
3. Utility vs. displacement — do people acknowledge MT's usefulness while
   also lamenting professional erosion?

## Repo structure

```
notebook.ipynb            Data collection (Bluesky/atproto API) + preprocessing
analysis.ipynb            Network analysis: centrality, community detection, visualizations
content_analysis.ipynb    Content analysis: sentiment, emotion, NER, visualizations
data/                     Collected/derived CSVs and generated plots (see below)
docs/                     Course material (gitignored, not part of the deliverable)
```

Run order: `notebook.ipynb` first (collects and cleans the data, saves the
CSVs in `data/`), then `analysis.ipynb` and `content_analysis.ipynb`
independently (both just load the saved CSVs).

## Data

- `data/seeds.csv`, `posts_raw.csv`, `posts_clean.csv` — collected posts at
  each pipeline stage (raw API responses → cleaned/filtered/labeled).
- `data/author_profiles.csv` — author metadata, bio, `user_type` label.
- `data/follow_edges.csv`, `graph.csv`, `reply_edges.csv` — network edges
  (follow + reply relationships) used for the network analysis.
- `data/posts_content_analysis.csv` — output of `content_analysis.ipynb`:
  posts enriched with sentiment (VADER + RoBERTa), emotion (NRCLex +
  DistilRoBERTa), and extracted entities.
- `data/*.png` — all visualizations, saved by both notebooks.

## Setup

```bash
pip install pandas atproto python-dotenv networkx matplotlib \
            vaderSentiment nrclex spacy wordcloud transformers torch
python -m spacy download en_core_web_sm
```

`notebook.ipynb` needs Bluesky credentials to hit the API — create a `.env`
file in the project root with:

```
BSKY_HANDLE=your-handle.bsky.social
BSKY_PASSWORD=your-app-password
```

(Only needed to re-run data collection from scratch; `analysis.ipynb` and
`content_analysis.ipynb` only need the CSVs already in `data/` and don't
touch the API.)

## Models used

**Network analysis** (`analysis.ipynb`): degree/betweenness/closeness
centrality; community detection via Louvain, Greedy Modularity, FluidC, and
Girvan-Newman (compared by modularity).

**Content analysis** (`content_analysis.ipynb`), each with two independent
methods compared against each other rather than taken at face value:
- Sentiment: VADER (lexicon-based) vs. `cardiffnlp/twitter-roberta-base-sentiment`
  (transformer).
- Emotion: NRCLex (dictionary-based) vs.
  `j-hartmann/emotion-english-distilroberta-base` (transformer).
- Named entity recognition: spaCy (`en_core_web_sm`).

The dual-method comparisons are a deliberate part of the analysis, not
redundant work: they surface real weaknesses in the lexicon-based tools
(e.g. NRCLex's dominant-emotion output is close to invariant to actual
content — see `content_analysis.ipynb` for details) that a single-method
analysis would have taken at face value.
