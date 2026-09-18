# VidSeek — Hybrid Semantic + Knowledge-Graph Video Search

A search engine for finding dashcam incident videos using plain-English queries. Built for the **Motive AI Hackathon (Problem 2: Video Indexing & Search)**, it combines dense semantic retrieval with a structured knowledge graph so results are both *meaning-aware* and *precise*.

> Example: a query like *"hard cornering on a rural road during overcast weather"* returns matching clips ranked by how well they fit semantically **and** how well their metadata (event type, location, objects seen) lines up with the query.

## How It Works

The pipeline runs in four stages:

1. **Ingest** — Loads per-video metadata (`video_features.tsv`) from an S3 bucket into a DataFrame.
2. **Describe** — Generates a rich natural-language description of each video's incident using **Claude (via Amazon Bedrock)**, based on its structured features.
3. **Index** — Encodes descriptions into embeddings with `sentence-transformers` (`intfloat/e5-base-v2`) and builds a **FAISS** index for fast approximate nearest-neighbor search. In parallel, a **NetworkX** knowledge graph links each video to its event type, location (street/city/state), and detected objects.
4. **Search** — A two-stage hybrid retrieval process:
   - **Retrieve:** FAISS returns the top semantic candidates for a query.
   - **Re-rank:** Claude parses the query into key entities (e.g. `["rollover", "guardrail"]`), and candidates are boosted based on how many of those entities appear as neighbors in the knowledge graph.

Top results are displayed with their semantic score, graph-match score, combined final score, an inline video preview, and the generated description.

## Features

- Natural-language video search over structured + unstructured dashcam metadata
- LLM-generated video descriptions (Claude on Bedrock)
- Dense semantic retrieval via FAISS
- Knowledge-graph-based re-ranking for higher precision on entity-specific queries (locations, objects, event types)
- Inline HTML5 video preview rendering for top results directly in the notebook

## Tech Stack

Python · pandas · FAISS · Sentence-Transformers · NetworkX · boto3 (AWS S3 & Bedrock) · Jupyter / Google Colab

## Example Queries

```
"videos with snow"
"night time driving"
"vehicle rollovers involving a guardrail"
"cracked windshield"
"find stop sign violations on Woodford Avenue"
"find hard cornering events on a rural road during overcast weather"
"find t-bone collision videos with a silver SUV"
```

## Setup

1. **Clone the repo and install dependencies:**
   ```bash
   pip install pandas pyarrow faiss-cpu sentence-transformers scikit-learn pyyaml boto3 networkx
   ```

2. **Configure AWS credentials via environment variables** — do **not** hardcode credentials in the notebook:
   ```bash
   export AWS_ACCESS_KEY_ID="your-access-key"
   export AWS_SECRET_ACCESS_KEY="your-secret-key"
   export AWS_DEFAULT_REGION="us-east-1"
   ```
   `boto3.client()` picks these up automatically with no code changes needed. For local development, a git-ignored `.env` file with `python-dotenv` also works well.

3. **Set your data source** — update `Config.BUCKET` and `Config.KEY_FEATURES` to point at your own S3 bucket and metadata file.

4. **Run the notebook** in Jupyter or Google Colab, and call `hybrid_search_and_preview(your_queries, assets)` with your own list of natural-language queries.

## Security Note

This project reads from a private S3 bucket and calls Amazon Bedrock, both of which require credentials. **Never commit AWS keys, API tokens, or `.env` files to version control.** Use environment variables, a secrets manager, or IAM roles instead, and add any local credential files to `.gitignore`.

## Acknowledgments

Built for the Motive AI Hackathon (Problem 2: Video Indexing & Search).

## License

Add a license of your choice (e.g. MIT) if you intend to share this repository publicly.
