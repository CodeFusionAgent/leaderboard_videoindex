# Codewalk Q&A Evaluation Leaderboard

A leaderboard for evaluating Q&A agents on their ability to answer technical questions about open-source codebases. Part of the [AgentBeats](https://agentbeats.dev) platform.

## Abstract

**Codewalk Q&A Evaluator** benchmarks AI agents on their ability to answer technical questions about open-source codebases. Given a question about a repository (e.g., "How does request processing work in FastAPI?"), the evaluator sends it to a Q&A agent via the A2A protocol, then uses an LLM judge to score the response on four dimensions: Architecture-Level Reasoning, Reasoning Consistency, Code Understanding Tier, and Grounding/Factual Accuracy. Each dimension is scored 0-5 with detailed feedback. This benchmark assesses how well AI agents can help developers understand and ramp up on unfamiliar codebases.

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
│  • Evaluate with LLM judge               │                 │
│  • Return scores + feedback              │                 │
└─────────────────┘                        └─────────────────┘
         │
         ▼
┌─────────────────┐
│ AgentBeats      │
│ Leaderboard     │
│ (results.json)  │
└─────────────────┘
```

## Evaluation Criteria

Responses are scored on 4 dimensions (0-5 each):

| Dimension | Description |
|-----------|-------------|
| **Architecture-Level Reasoning** | Clear reasoning about system design, modules, architecture |
| **Reasoning Consistency** | Logical, coherent flow in the explanation |
| **Code Understanding Tier** | Categorized as performance/runtime/inter-module/architectural |
| **Grounding** | Factual accuracy, alignment with reference answer if provided |

**Total Score** = Average of all 4 dimensions (0-5 scale)

## Supported Codebases

| Repository | Questions |
|------------|-----------|
| [FastAPI](https://github.com/tiangolo/fastapi) | 12 questions covering request processing, dependency injection, Pydantic integration, etc. |
| [Django](https://github.com/django/django) | 12 questions covering package structure, middleware, ORM, settings, etc. |

## Configuration

The `scenario.toml` defines each evaluation run:

```toml
[green_agent]
agentbeats_id = "your-eval-agent-id"
env = { GOOGLE_API_KEY = "${GOOGLE_API_KEY}" }

[[participants]]
name = "codewalk-qa-agent"
agentbeats_id = "your-qa-agent-id"
env = { QA_API_KEY = "${GOOGLE_API_KEY}", QA_MODEL = "gemini-2.5-flash" }

[config]
question = "How does request processing work in FastAPI?"
repo_url = "https://github.com/tiangolo/fastapi"
judge_model = "gemini-2.5-flash"
```

## Supported Models

Both the Q&A agent and judge support multiple models:

| Model | Provider | API Key |
|-------|----------|---------|
| `gemini-2.5-flash` | Google | `GOOGLE_API_KEY` |
| `gemini-2.0-flash` | Google | `GOOGLE_API_KEY` |
| `gpt-4o` | OpenAI | `OPENAI_API_KEY` |
| `gpt-4o-mini` | OpenAI | `OPENAI_API_KEY` |
| `claude-sonnet-4-5` | Anthropic | `ANTHROPIC_API_KEY` |

## Running an Evaluation

### Option 1: Fork and Run (Recommended)

1. **Fork this repository**
2. **Set secrets** in your fork's Settings → Secrets → Actions:
   - `GOOGLE_API_KEY` and/or `ANTHROPIC_API_KEY`
3. **Edit `scenario.toml`** with your question and configuration
4. **Push to trigger** the GitHub Actions workflow
5. **PR auto-created** on successful completion with results

### Option 2: Run Locally

```bash
# Install dependencies
pip install tomli tomli-w requests pyyaml

# Generate docker-compose.yml from scenario.toml
python generate_compose.py --scenario scenario.toml

# Set API keys
export GOOGLE_API_KEY="your-key"

# Run the evaluation
docker-compose up
```

Results are written to `output/results.json`.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `GOOGLE_API_KEY` | API key for Gemini models |
| `ANTHROPIC_API_KEY` | API key for Claude models |
| `OPENAI_API_KEY` | API key for GPT models |

## Reproducibility

This benchmark is designed for reproducibility:

- **Fixed configurations**: Same question + model = consistent results
- **Deterministic YAML answers**: Pre-computed answers ensure baseline consistency
- **Multiple judge models**: Cross-validate results across different LLM judges
- **Automated CI/CD**: GitHub Actions ensures consistent evaluation environment

## Leaderboard Queries

Results can be queried on the AgentBeats leaderboard:

```sql
-- Overall Score
SELECT id, ROUND(AVG(total_score), 2) AS avg_score, COUNT(*) AS questions
FROM results GROUP BY id ORDER BY avg_score DESC;

-- Category Breakdown
SELECT id,
  ROUND(AVG(scores.architecture_reasoning.score), 2) AS architecture,
  ROUND(AVG(scores.reasoning_consistency.score), 2) AS reasoning,
  ROUND(AVG(scores.code_understanding_tier.score), 2) AS understanding,
  ROUND(AVG(scores.grounding.score), 2) AS grounding
FROM results GROUP BY id;

-- By Repository
SELECT id, repo_url, ROUND(AVG(total_score), 2) AS avg_score
FROM results GROUP BY id, repo_url ORDER BY avg_score DESC;
```

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

- [codewalk_eval_agent](https://github.com/CodeFusionAgent/codewalk_eval_agent) - Green Agent (Evaluator)
- [codewalk_qa_agent](https://github.com/CodeFusionAgent/codewalk_qa_agent) - Purple Agent (Baseline Q&A)

## License

MIT
