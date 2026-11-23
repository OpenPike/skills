---
name: openpike-setup
description: Sets up a project or a coding agent (Claude Code, Codex, opencode, Cline, Aider, OpenAI SDK, Anthropic SDK, Vercel AI SDK, LangChain) to send its LLM requests through OpenPike, and explains OpenPike errors such as budget_exceeded, loop_detected, scope_paused and plan_limit_reached. Use when the user wants to use OpenPike, set a base URL for OpenPike, tag an agent's spend, or when a request fails with an OpenPike error code. Never handles provider keys.
---

# Set up OpenPike

OpenPike is a gateway: requests carry an OpenPike virtual key (`opk_live_...`), and OpenPike forwards them to the user's own provider key and records spend. Docs: https://openpike.dev/docs/

## Rules

- Never ask for, print or write a provider key (OpenAI, Anthropic, DeepSeek and so on). Provider keys are added by the user in the OpenPike dashboard, on the **Providers** page.
- Never write the virtual key into a file that is committed. Read it from the `OPENPIKE_API_KEY` environment variable. If the user has no virtual key, tell them to create one on the **Keys** page of the dashboard, with the tag `agent=<name>`, and to set it themselves. Do not ask them to paste it into the chat.
- Use the config below exactly. Do not invent base URLs, model names or header names.

## Steps

1. Find out which client the project or agent uses (ask if it is not clear), and which models the user wants.
2. Check that `OPENPIKE_API_KEY` is set without printing it (for example, test that it is non-empty). If not, give the user the command to set it, below.
3. Apply the config for that client from the sections below.
4. Choose the model name (see "Model names").
5. Add a tag so spend shows per agent (see "Tags").
6. Tell the user to send one request and look for it under **Logs** in the dashboard, with its agent tag.

Setting the variable (the user runs this; the key is theirs):

```bash
export OPENPIKE_API_KEY="opk_live_..."
```

```powershell
$env:OPENPIKE_API_KEY = "opk_live_..."
```

## Endpoints

| Format | Base URL | Used by |
| --- | --- | --- |
| Chat completions | `https://api.openpike.dev/v1` | OpenAI-compatible clients and SDKs |
| Anthropic messages | `https://api.openpike.dev` (no `/v1`; the client adds `/v1/messages`) | Claude Code, Anthropic SDK |
| Responses | `https://api.openpike.dev/v1` | Codex CLI |

A model must be called on the endpoint its provider speaks. Claude models are available only in the Anthropic format; they cannot be used through the chat completions clients (opencode, Cline, Aider, LangChain).

## Model names

- `provider/model`: the part before the first slash is the provider slug, the rest is the provider's own model ID, for example `deepseek/deepseek-flash`, `openai/gpt-5.4-mini`, `anthropic/claude-sonnet-5-5`.
- `openpike/<alias>`: an alias the user created on the **Routing** page, for example `openpike/auto`, which falls back across models. It works only if the alias exists; otherwise the request gets `400 invalid_model`.
- To list valid names: `curl https://api.openpike.dev/v1/models -H "Authorization: Bearer $OPENPIKE_API_KEY"` (PowerShell: `curl.exe https://api.openpike.dev/v1/models -H "Authorization: Bearer $env:OPENPIKE_API_KEY"`).

## Tags

`x-openpike-tags` is a comma-separated list of `key=value` pairs (at most 10, each part at most 64 characters). Use `agent=<tool name>`, for example `agent=claude-code`, `agent=codex`, `agent=aider`, so spend, budgets and loop limits apply per agent. Where a client cannot send headers (Cline, Aider), the tag goes on the virtual key in the dashboard instead.

## Clients

Docs for each: https://openpike.dev/docs/integrations/<page>/

### Claude Code (https://openpike.dev/docs/integrations/claude-code/)

Put this in `~/.claude/settings.json` (or `.claude/settings.local.json`; never the committed `.claude/settings.json`). The token value stays a placeholder here: the user fills it in, or sets the variables in their shell instead.

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.openpike.dev",
    "ANTHROPIC_AUTH_TOKEN": "opk_live_...",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "anthropic/claude-opus-5-5",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "anthropic/claude-sonnet-5-5",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "anthropic/claude-haiku-5-5",
    "ANTHROPIC_CUSTOM_HEADERS": "x-openpike-tags: agent=claude-code"
  }
}
```

Shell form: `export ANTHROPIC_BASE_URL=https://api.openpike.dev`, `export ANTHROPIC_AUTH_TOKEN="$OPENPIKE_API_KEY"`, the three `ANTHROPIC_DEFAULT_*_MODEL` variables, and `export ANTHROPIC_CUSTOM_HEADERS="x-openpike-tags: agent=claude-code"`. Check with `/status`: the base URL must show `https://api.openpike.dev`.

### Codex CLI (https://openpike.dev/docs/integrations/codex-cli/)

Works with `openai/`, `xai/`, `groq/` and `openrouter/` models only. In `~/.codex/config.toml`:

```toml
model = "openai/gpt-5.4"
model_provider = "openpike"

[model_providers.openpike]
name = "OpenPike"
base_url = "https://api.openpike.dev/v1"
env_key = "OPENPIKE_API_KEY"
wire_api = "responses"
http_headers = { "x-openpike-tags" = "agent=codex" }
```

### opencode (https://openpike.dev/docs/integrations/opencode/)

In `opencode.jsonc`; the key comes from `OPENPIKE_API_KEY`. Chat completions models only.

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "providers": {
    "openpike": {
      "name": "OpenPike",
      "env": ["OPENPIKE_API_KEY"],
      "package": "@opencode/ai/providers/openai-compatible",
      "settings": {
        "baseURL": "https://api.openpike.dev/v1"
      },
      "headers": {
        "x-openpike-tags": "agent=opencode"
      },
      "models": {
        "deepseek-flash": { "name": "DeepSeek Flash", "modelID": "deepseek/deepseek-flash" },
        "gpt-5.4-mini": { "name": "GPT-5.4 mini", "modelID": "openai/gpt-5.4-mini" }
      }
    }
  },
  "model": "openpike/deepseek-flash"
}
```

### Cline (https://openpike.dev/docs/integrations/cline/)

Settings (gear icon): **API Provider** `OpenAI Compatible`, **Base URL** `https://api.openpike.dev/v1`, **API Key** the virtual key (the user enters it in Cline's settings), **Model ID** an OpenPike name such as `deepseek/deepseek-flash` or `openpike/auto`. Cline has no header field, so the `agent=cline` tag must be on the virtual key.

### Aider (https://openpike.dev/docs/integrations/aider/)

```bash
export OPENAI_API_BASE=https://api.openpike.dev/v1
export OPENAI_API_KEY="$OPENPIKE_API_KEY"
aider --model openai/deepseek/deepseek-flash
```

The `openai/` prefix is Aider's, then the full OpenPike name. Aider warns about unknown models; that is expected. The `agent=aider` tag goes on the virtual key.

### OpenAI SDK (https://openpike.dev/docs/integrations/openai-sdk/)

```python
from openai import OpenAI
import os

client = OpenAI(
    base_url="https://api.openpike.dev/v1",
    api_key=os.environ["OPENPIKE_API_KEY"],
    default_headers={"x-openpike-tags": "agent=my-app"},
)
```

```ts
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://api.openpike.dev/v1",
  apiKey: process.env.OPENPIKE_API_KEY,
  defaultHeaders: { "x-openpike-tags": "agent=my-app" },
});
```

`client.chat.completions.create` works with every provider's chat models. `client.responses.create` works with OpenAI, xAI, Groq and OpenRouter models only.

### Anthropic SDK (https://openpike.dev/docs/integrations/anthropic-sdk/)

The base URL has no `/v1`.

```python
from anthropic import Anthropic
import os

client = Anthropic(
    base_url="https://api.openpike.dev",
    api_key=os.environ["OPENPIKE_API_KEY"],
    default_headers={"x-openpike-tags": "agent=my-app"},
)
```

```ts
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  baseURL: "https://api.openpike.dev",
  apiKey: process.env.OPENPIKE_API_KEY,
  defaultHeaders: { "x-openpike-tags": "agent=my-app" },
});
```

### Vercel AI SDK (https://openpike.dev/docs/integrations/vercel-ai-sdk/)

```ts
import { createOpenAI } from "@ai-sdk/openai";
import { createAnthropic } from "@ai-sdk/anthropic";

const headers = { "x-openpike-tags": "agent=my-app" };

// Chat completions models. Use .chat(): the default openai("model") calls the Responses API.
const openpike = createOpenAI({
  baseURL: "https://api.openpike.dev/v1",
  apiKey: process.env.OPENPIKE_API_KEY,
  headers,
});
const model = openpike.chat("deepseek/deepseek-flash");

// Claude models. This base URL keeps /v1, unlike Anthropic's own SDK.
const openpikeClaude = createAnthropic({
  baseURL: "https://api.openpike.dev/v1",
  apiKey: process.env.OPENPIKE_API_KEY,
  headers,
});
const claude = openpikeClaude("anthropic/claude-haiku-5-5");
```

### LangChain (https://openpike.dev/docs/integrations/langchain/)

Chat completions models only; leave `use_responses_api` off. Claude models need the Anthropic SDK instead.

```python
import os
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    base_url="https://api.openpike.dev/v1",
    api_key=os.environ["OPENPIKE_API_KEY"],
    model="deepseek/deepseek-flash",
    default_headers={"x-openpike-tags": "agent=my-app"},
)
```

```ts
import { ChatOpenAI } from "@langchain/openai";

const llm = new ChatOpenAI({
  apiKey: process.env.OPENPIKE_API_KEY,
  model: "deepseek/deepseek-flash",
  configuration: {
    baseURL: "https://api.openpike.dev/v1",
    defaultHeaders: { "x-openpike-tags": "agent=my-app" },
  },
});
```

## OpenPike errors

A `429` without an `x-openpike-provider` response header came from OpenPike itself, not the provider. Its `error.code` (on `/v1/messages`, its message) says which. Do not retry these in a loop: they do not clear in seconds. Stop, tell the user the code and the message, and what to change.

| Code | Meaning | What to do |
| --- | --- | --- |
| `budget_exceeded` | A hard budget for the workspace, key or tag is used up for this period. `retry-after` is the seconds until it resets. | Stop. Tell the user which budget (the message names it) and when it resets. They can raise the limit on the **Budgets** page. Do not switch keys or tags to get around it. |
| `loop_detected` | Loop detection paused this key or tag because its requests or spend per minute crossed a limit. `retry-after` is the seconds left on the pause. | The agent is probably repeating itself. Stop and look for the loop before resuming. The pause ends by itself after `retry-after`, or the user can **Resume** it on the **Agents** page. |
| `scope_paused` | The user paused this agent or key in the dashboard. No end time. | Stop. Ask the user to select **Resume** on the **Agents** page when they want it to continue. |
| `plan_limit_reached` | The organization used its plan's monthly requests (UTC), in block mode. `retry-after` is the seconds until the month resets. | Stop. The user can upgrade on the **Billing** page or wait for the reset. |

Related codes: `run_limit_exceeded` (the run in `x-openpike-run` hit its spend, request or time cap; the run stays blocked, so stop the task as described below; only a new task the user starts gets a new run ID), `plan_limit_throttled` (over the monthly requests in throttle mode: at most 10 requests a minute; retry after `retry-after`), `invalid_virtual_key` (401, the key is missing, unknown or revoked: check `OPENPIKE_API_KEY`), `organization_suspended` (403, OpenPike stopped every key of the organization: stop and tell the user to contact support@openpike.dev), `missing_provider_key` (424, add a key for that provider on the **Providers** page), `invalid_model` and `unsupported_format` (400, the model name or its endpoint is wrong: see "Model names" and "Endpoints"). All codes: https://openpike.dev/docs/api/errors/

Do not retry a block. On `budget_exceeded`, `loop_detected`, `scope_paused` or `run_limit_exceeded`, stop the task and tell the user. Do not keep retrying the same request, and do not restart the task to get around the block. Leave the SDK's default retries on: OpenPike refuses blocked requests before any provider call, so the few default retries on a `429` cost nothing, and they still cover provider `5xx` errors.
