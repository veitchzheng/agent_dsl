# AGENTS.md

## Project Overview

This repository contains **Dify AI application DSL definitions** — YAML-based workflow configurations for AI agents deployed on the Dify platform. Each `.yml` file defines a complete AI agent, including its graph-based workflow (nodes + edges), LLM prompts, knowledge retrieval settings, and feature flags.

- **Platform**: Dify (v0.1.2 DSL format)
- **App mode**: `advanced-chat` (conversational agents with graph workflows)
- **LLM Provider**: `openai_api_compatible` (Qwen3-30B-A3B-Instruct-2507 / Qwen3-30B-A3B-Thinking-2507)

### Workflow Node Types

| Type | Purpose |
|------|---------|
| `start` | Entry point; defines input variables (text, file, select) |
| `llm` | LLM call with prompt template and model config |
| `question-classifier` | Routes user input to branches based on intent classification |
| `knowledge-retrieval` | Retrieves documents from a Dify knowledge base |
| `if-else` | Conditional branching based on variable comparisons |
| `document-extractor` | Extracts text from uploaded document files |
| `assigner` | Assigns values to conversation/environment variables |
| `code` | Executes Python3 code for data transformation |
| `answer` | Terminal node; renders the final response |

### Variable System

Variables are referenced via selectors like `[node_id, field_name]` or system scopes:
- `sys.query` — user's input text
- `conversation.<var_name>` — conversation-scoped variables
- `{{#node_id.field#}}` — template syntax for prompt injection

## Build & Commands

There is no build step. These YAML files are imported directly into the Dify platform via its DSL import feature.

- **Import**: In Dify Studio, use "Import DSL" to load a `.yml` file as a new app
- **Export**: Use "Export DSL" in Dify Studio to update the YAML in this repo
- **Validation**: Dify validates DSL on import; no local validation tooling exists in this repo

## Code Style

- **File encoding**: UTF-8
- **Indentation**: 2 spaces (YAML standard)
- **Naming**: Chinese names for app titles and node labels; English for variable names and technical identifiers
- **Node IDs**: Unix millisecond timestamps (e.g., `1778138151321`)
- **Prompts**: Embedded inline in YAML as multi-line strings (`|` block scalar)
- **One app per file**: Each `.yml` file is a self-contained Dify app definition

## Testing

No automated testing framework exists in this repo. Testing is performed manually:

1. Import the DSL into a Dify test workspace
2. Run the app in Dify's "Preview" mode with sample inputs
3. Verify each workflow branch produces correct outputs
4. Check knowledge retrieval relevance and LLM response quality
5. Export the updated DSL back to the repo after changes are validated

## Security

- **Prompt injection**: User input (`sys.query`) is injected into LLM prompts via `{{#sys.query#}}`. Ensure system prompts include guardrails against prompt injection attacks.
- **Knowledge base access**: `knowledge-retrieval` nodes reference Dify dataset IDs (UUIDs). These IDs are platform-specific and should not be exposed publicly.
- **Code execution**: `code` nodes run Python3 on the Dify server. Avoid executing untrusted input in code nodes.
- **Sensitive word avoidance**: Both apps have `sensitive_word_avoidance.enabled: false`. Enable this feature in production if the app faces public users.

## Configuration

### App-Level Settings

Each YAML file's `app` section contains:
- `name` / `description` — display metadata
- `mode: advanced-chat` — must remain `advanced-chat` for graph workflows
- `icon` / `icon_background` — UI presentation

### Feature Flags (per app, under `workflow.features`)

- `file_upload` — controls file upload capabilities (disabled in both current apps)
- `retriever_resource` — enables knowledge base retrieval (enabled in both)
- `speech_to_text` / `text_to_speech` — voice I/O (disabled)
- `sensitive_word_avoidance` — content filtering (disabled)
- `opening_statement` — greeting message (empty in both)
- `suggested_questions` / `suggested_questions_after_answer` — suggestion prompts (empty/disabled)

### Model Configuration

Each `llm` and `question-classifier` node specifies:
- `model.provider: openai_api_compatible`
- `model.name` — the model identifier (e.g., `Qwen3-30B-A3B-Instruct-2507`)
- `model.mode: chat`
- `model.completion_params.temperature` — typically `0.7`

### Environment Variables

Both apps have `environment_variables: []`. If API keys or endpoints need configuration, add them here and reference via Dify's variable system.

## Current Apps

### 政策咨询 (Policy Consultation)
A policy advisory agent that classifies user input into `policy_consult` or `chitchat` branches. Policy queries go through knowledge retrieval then LLM answer generation; chitchat gets a polite redirection response.

### 文件校对 (Document Proofreading)
A document proofreading agent supporting text paste or file upload. Uses `if-else` to branch on input type, extracts document text if needed, runs LLM proofreading, then parses the JSON result via a `code` node to produce a formatted correction report.
