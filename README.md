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
4. Network structure — do professional translators and general users form
   distinct communities in the follow/reply networks, and which accounts
   bridge them?

## Repo structure

```
01_data_collection.ipynb  Data collection (Bluesky/atproto API, Colab) + preprocessing
02_network_analysis.ipynb Network analysis: centrality, community detection, visualizations
03_content_analysis.ipynb Content analysis: sentiment, emotion, NER, visualizations
data/                     Collected/derived CSVs and generated plots (see below)
docs/                     Course material (gitignored, not part of the deliverable)
```

Run order: `01_data_collection.ipynb` first (Colab only — collects and
cleans the data, saves the CSVs to Drive), then `02_network_analysis.ipynb`
and `03_content_analysis.ipynb` independently (both just load the saved
CSVs from `data/`, no API access needed).

## Data

- `data/seeds.csv`, `posts_raw.csv`, `posts_clean.csv` — collected posts at
  each pipeline stage (raw API responses → cleaned/filtered/labeled).
- `data/author_profiles.csv` — author metadata, bio, `user_type` label
  (`general` / `professional` / `tech` / `unknown` — `unknown` means a
  profile was fetched but had no bio text, kept separate from `general` so
  "no information" isn't silently treated as "ordinary user").
- `data/follow_edges.csv`, `graph.csv`, `reply_edges.csv` — network edges
  (follow + reply relationships) used for the network analysis. Reply
  edges are derived from each post's `reply_to` field during
  preprocessing (not collected at crawl time), which is idempotent —
  re-running preprocessing cannot corrupt `graph.csv`.
- `data/posts_content_analysis.csv` — output of `03_content_analysis.ipynb`:
  posts enriched with sentiment (VADER + RoBERTa), emotion (NRCLex +
  DistilRoBERTa), and extracted entities.
- `data/*.png` — all visualizations, saved by both analysis notebooks.

## Setup

```bash
pip install pandas atproto networkx matplotlib \
            vaderSentiment nrclex spacy wordcloud transformers torch
python -m spacy download en_core_web_sm
```

`01_data_collection.ipynb` runs in Google Colab and needs Bluesky
credentials as Colab secrets (`BSKY_HANDLE`, `BSKY_PASSWORD`) and a mounted
Drive — see the notebook's Setup section. `02_network_analysis.ipynb` and
`03_content_analysis.ipynb` only need the CSVs already in `data/` and run
anywhere (no Colab, no API access).

## Models used

**Network analysis** (`02_network_analysis.ipynb`): degree/betweenness/closeness
centrality; community detection via Louvain, Greedy Modularity, FluidC, and
Girvan-Newman (compared by modularity).

**Content analysis** (`03_content_analysis.ipynb`), each with two independent
methods compared against each other rather than taken at face value:
- Sentiment: VADER (lexicon-based) vs. `cardiffnlp/twitter-roberta-base-sentiment`
  (transformer).
- Emotion: NRCLex (dictionary-based) vs.
  `j-hartmann/emotion-english-distilroberta-base` (transformer).
- Named entity recognition: spaCy (`en_core_web_sm`).

The dual-method comparisons are a deliberate part of the analysis, not
redundant work: they surface real weaknesses in the lexicon-based tools
(e.g. NRCLex's dominant-emotion output is close to invariant to actual
content — see `03_content_analysis.ipynb` for details) that a single-method
analysis would have taken at face value.

## Collection limitations

Documented in full in `01_data_collection.ipynb`'s final section: ego-network
sample (follow edges only exist around authors with ≥ 2 posts), a 100-follower
cap per author, followers-not-following direction, a multilingual corpus
(keyword-filtered, not language-filtered), handles used as node IDs (DIDs
stored as a fallback), and collection ethics (public data only, polite
request rates, official API).
