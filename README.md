# VideoIndex Q&A Evaluation Leaderboard

A leaderboard for evaluating Q&A agents on their ability to answer questions about video content. Part of the [AgentBeats](https://agentbeats.dev) platform.

## Vision

We believe the way we interact with video is about to change forever. Instead of passively watching a recording, what if you could have a conversation with it?

Today, a 10-hour lecture or a long keynote is a knowledge graveyard. You know the insights are there, but you can't find them when you need them. We're tackling this by building an interactive AI tutor that "watches" entire videos on your behalf, transforming long, static recordings into a conversational knowledge base.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Leaderboard Repository                       │
│  scenario.toml ─▶ generate_compose.py ─▶ docker-compose.yml     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────┐     A2A Protocol      ┌─────────────────┐
│   Eval Agent    │ ──────────────────────▶│    Q&A Agent    │
│  (Green Agent)  │                        │ (Purple Agent)  │
│                 │◀────────────────────── │                 │
│  • Send question                         │  • Return answer│
│  • Evaluate semantic similarity          │                 │
│  • Return score + feedback               │                 │
└─────────────────┘                        └─────────────────┘
         │
         ▼
┌─────────────────┐
│ AgentBeats      │
│ Leaderboard     │
│ (results.json)  │
└─────────────────┘
```

## Evaluation

The evaluator uses an LLM judge to measure **semantic similarity** between the Q&A agent's response and the correct answer:

- **Score**: 0.0 to 1.0 based on semantic similarity
- **Judge Models**: Gemini, Claude, GPT-4o
- **Dataset**: LongTVQA - Multiple choice questions about TV show content (The Big Bang Theory)
- **Questions**: 20 curated questions spanning different episodes

Scoring guide:
- 1.0: Semantically identical or equivalent meaning
- 0.7-0.9: Very similar, minor differences
- 0.4-0.6: Partially similar, some overlap
- 0.1-0.3: Slightly related but mostly different
- 0.0: Completely different or contradictory

## Dataset

| Source | Episodes | Questions |
|--------|----------|-----------|
| LongTVQA (The Big Bang Theory) | 20 episodes | 20 questions |

Questions test video understanding including:
- Character motivations ("Why did Raj tell himself to turn his pelvis...?")
- Emotional states ("How did Penny feel when...?")
- Plot events ("What did Howard tell Leonard after...?")

## Configuration

The `scenario.toml` defines each evaluation run:

```toml
[green_agent]
name = "videoindex-eval-agent"
agentbeats_id = "your-eval-agent-id"
env = { GOOGLE_API_KEY = "${GOOGLE_API_KEY}", EVAL_DATA_DIR = "data" }

[[participants]]
name = "videoindex-qa-agent"
agentbeats_id = "your-qa-agent-id"
env = { QA_DATA_DIR = "data" }

[config]
question = "Why did Raj tell himself to turn his pelvis when Penny was giving him a hug?"
judge_model = "gemini-2.5-flash"
```

## Supported Judge Models

| Model | Provider | API Key |
|-------|----------|---------|
| `gemini-2.5-flash` (default) | Google | `GOOGLE_API_KEY` |
| `gemini-2.0-flash` | Google | `GOOGLE_API_KEY` |
| `gpt-4o` | OpenAI | `OPENAI_API_KEY` |
| `claude-sonnet-4-5` | Anthropic | `ANTHROPIC_API_KEY` |

## Running an Evaluation

### Option 1: Fork and Run (Recommended)

1. **Fork this repository**
2. **Set secrets** in your fork's Settings → Secrets → Actions:
   - `GOOGLE_API_KEY` and/or `ANTHROPIC_API_KEY`
3. **Edit `scenario.toml`** with your question
4. **Push to trigger** the GitHub Actions workflow
5. **PR auto-created** on successful completion with results

### Option 2: Run Locally

```bash
# Install dependencies
pip install tomli tomli-w requests pyyaml

# Generate docker-compose.yml from scenario.toml
python generate_compose.py --scenario scenario.toml

# Run the evaluation
docker-compose up
```

Results are written to `output/results.json`.

## Project Structure

```
scenario.toml           # Evaluation configuration
generate_compose.py     # Generates docker-compose.yml from scenario
record_provenance.py    # Records evaluation metadata
submissions/            # Submitted scenario files
results/                # Evaluation results
.github/workflows/
└─ run-scenario.yml     # CI workflow for automated evaluation
```

## Related Repositories

- [videoindex_eval_agent](https://github.com/CodeFusionAgent/videoindex_eval_agent) - Green Agent (Evaluator)
- [videoindex_qa_agent](https://github.com/CodeFusionAgent/videoindex_qa_agent) - Purple Agent (Baseline Q&A)

## License

MIT
