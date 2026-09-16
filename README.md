# Gym Workout Builder Bot — RAG Project (Week 2)

A personal RAG app that builds workouts by retrieving from a self-authored library of
structured training programs, built no-code in n8n Cloud.

## Architecture

Two n8n workflows, built entirely from core nodes (no LangChain community nodes — this n8n
Cloud plan forbids installing unverified community packages):

- **`n8n_workflows/gym_bot_ingestion_http.json`** — embeds and upserts the corpus into Pinecone.
  Code node holds the 7 corpus docs and chunks them by section; HTTP Request node embeds each
  chunk via Nebius Token Factory (`Qwen/Qwen3-Embedding-8B`, truncated to 1024 dimensions);
  another HTTP Request node upserts the batch to Pinecone.
- **`n8n_workflows/gym_bot_chat_form.json`** — the user-facing bot. A Form Trigger gives a
  shareable web form; the submitted question is embedded, matched against Pinecone (top-4),
  assembled into a context block, and sent to Nebius's chat completion API
  (`Qwen/Qwen3-30B-A3B-Instruct-2507`) with a system prompt enforcing grounded answers and a
  strict refusal path for out-of-library requests. The answer is shown on the form's completion
  page.

## Corpus

`corpus/` — 7 markdown training programs I authored, each with YAML front-matter
(goal, split_type, equipment, experience_level, days_per_week) and day-by-day content.

## Setup

1. Create a Pinecone serverless index named `gym-workout-programs`, dimension `1024`, metric
   `cosine`.
2. In n8n, create two **HTTP Header Auth** credentials:
   - "Nebius Header Auth" — Header Name `Authorization`, Header Value `Bearer <your Nebius API key>`
   - "Pinecone Header Auth" — Header Name `Api-Key`, Header Value `<your Pinecone API key>`
3. Import `gym_bot_ingestion_http.json`. In the "Upsert to Pinecone" node, replace the URL host
   with your Pinecone index's host. Re-select both HTTP Header Auth credentials in their nodes.
   Run the workflow once (manual trigger) to populate the index.
4. Import `gym_bot_chat_form.json`. In the "Query Pinecone" node, replace
   `REPLACE-WITH-YOUR-PINECONE-INDEX-HOST` with the same Pinecone host. Re-select both
   credentials. Activate the workflow.
5. Open the Form Trigger node's **Production URL** to use the bot.

## project doc

See the project documentation in docs/Gym_Workout_Bot_Project_Documentation.docx for the full framework write-up, iteration history,
and evaluation results. Earlier attempts using n8n's LangChain-based nodes (Vector Store, Chat
Trigger, etc.) are not included here — they failed because this n8n Cloud plan blocks that
entire package; the documentation narrates that journey.
