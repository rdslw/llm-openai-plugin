# llm-openai-plugin

[![PyPI](https://img.shields.io/pypi/v/llm-openai-plugin.svg)](https://pypi.org/project/llm-openai-plugin/)
[![Changelog](https://img.shields.io/github/v/release/simonw/llm-openai-plugin?include_prereleases&label=changelog)](https://github.com/simonw/llm-openai-plugin/releases)
[![Tests](https://github.com/simonw/llm-openai-plugin/actions/workflows/test.yml/badge.svg)](https://github.com/simonw/llm-openai-plugin/actions/workflows/test.yml)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](https://github.com/simonw/llm-openai-plugin/blob/main/LICENSE)

[LLM](https://llm.datasette.io/) plugin for [OpenAI models](https://platform.openai.com/docs/models).

This plugin **is a preview**. LLM currently ships with OpenAI models as part of its default collection, implemented using the [Chat Completions API](https://platform.openai.com/docs/guides/responses-vs-chat-completions).

This plugin implements those same models using the new [Responses API](https://platform.openai.com/docs/api-reference/responses).

Currently the only reason to use this plugin over the LLM defaults is to access [o1-pro](https://platform.openai.com/docs/models/o1-pro), which can only be used via the Responses API.

## Installation

Install this plugin in the same environment as [LLM](https://llm.datasette.io/).
```bash
llm install llm-openai-plugin
```
## Usage

To run a prompt against `o1-pro` do this:

```bash
llm -m openai/o1-pro "Convince me that pelicans are the most noble of birds"
```

Run this to see a full list of models - they start with the `openai/` prefix:

```bash
llm models -q openai/
```

Here's the output of that command:

<!-- [[[cog
import cog
from llm import cli
from click.testing import CliRunner
runner = CliRunner()
result = runner.invoke(cli.cli, ["models", "-q", "openai/"])
cog.out(
    "```\n{}\n```".format(result.output.strip())
)
]]] -->
```
OpenAI: openai/gpt-4o
OpenAI: openai/gpt-4o-mini
OpenAI: openai/gpt-4.5-preview
OpenAI: openai/gpt-4.5-preview-2025-02-27
OpenAI: openai/o3-mini
OpenAI: openai/o1-mini
OpenAI: openai/o1
OpenAI: openai/o1-pro
OpenAI: openai/gpt-4.1
OpenAI: openai/gpt-4.1-2025-04-14
OpenAI: openai/gpt-4.1-mini
OpenAI: openai/gpt-4.1-mini-2025-04-14
OpenAI: openai/gpt-4.1-nano
OpenAI: openai/gpt-4.1-nano-2025-04-14
OpenAI: openai/o3
OpenAI: openai/o3-2025-04-16
OpenAI: openai/o3-streaming
OpenAI: openai/o3-2025-04-16-streaming
OpenAI: openai/o4-mini
OpenAI: openai/o4-mini-2025-04-16
OpenAI: openai/codex-mini-latest
OpenAI: openai/o3-pro
OpenAI: openai/gpt-5
OpenAI: openai/gpt-5-mini
OpenAI: openai/gpt-5-nano
OpenAI: openai/gpt-5-2025-08-07
OpenAI: openai/gpt-5-mini-2025-08-07
OpenAI: openai/gpt-5-nano-2025-08-07
OpenAI: openai/gpt-5-codex
OpenAI: openai/gpt-5-pro
OpenAI: openai/gpt-5-pro-2025-10-06
```
<!-- [[[end]]] -->
Add `--options` to see a full list of options that can be provided to each model.

The `o3-streaming` model ID exists because o3 currently requires a verified organization in order to support streaming. If you have a verified organization you can use `o3-streaming` - everyone else should use `o3`.

## Web Search

Enable the `web_search` OpenAI platform side tool to let models search the web using openai index.

```bash
llm -m openai/gpt-5.1 -o openai_tools web_search "top 3 November 2025 news about Anthropic from leading news outlets"
```
> Here are three of the biggest Anthropic stories from November 2025 (US time):
> - Nov 12, 2025 — Anthropic announces a $50 billion buildout of custom US data centers (Texas and New York) with Fluidstack to meet AI compute demand, coming online in 2026. ([reuters.com](https://www.reuters.com/technology/anthropic-invest-50-billion-build-data-centers-us-2025-11-12/))
> - Nov 13–14, 2025 — Anthropic says it disrupted a largely automated cyber‑espionage campaign it links to a Chinese state‑sponsored group that misused Claude Code to target ~30 organizations; coverage underscores the rise of AI‑driven attacks. ([apnews.com](https://apnews.com/article/4e7e5b1a7df946169c72c1df58f90295?utm_source=openai))
> - Nov 11, 2025 — The Wall Street Journal reports Anthropic’s internal projections show a faster path to profitability than OpenAI (e.g., aiming to break even by 2028), highlighting sharply different spending strategies. ([wsj.com](https://www.wsj.com/tech/ai/openai-anthropic-profitability-e9f5bcd6?utm_source=openai))


### Caveats

- This is completly separate from local llm tools.
- You dont see whole tool_call/tool_response content.
- It costs additional API money (as of November $10 per 1k searches).


### Options

- **`web_search_domains`**: Limit results to specific domains (comma-separated, up to 20):
  ```bash
  llm -m openai/gpt-5.1 -o openai_tools web_search \
    -o web_search_domains "openai.com,wikipedia.org" "Search query"
  ```

- **`web_search_live`**: Control live internet access (default: true):
  ```bash
  llm -m openai/gpt-5.1 -o openai_tools web_search \
    -o web_search_live false "Cached results only"
  ```

- **`web_search_sources`**: Include all retrieved URLs in sources field:
  ```bash
  llm -m openai/gpt-5.1 -o openai_tools web_search \
    -o web_search_sources true "Query with sources"
  ```

**Note:** User location options (`user_location`) are not currently supported.

## Development

To set up this plugin locally, first checkout the code. Then create a new virtual environment:
```bash
cd llm-openai-plugin
python -m venv venv
source venv/bin/activate
```
Now install the dependencies and test dependencies:
```bash
llm install -e '.[test]'
```
To run the tests:
```bash
python -m pytest
```

This project uses [pytest-recording](https://github.com/kiwicom/pytest-recording) to record OpenAI API responses for the tests, and [syrupy](https://github.com/syrupy-project/syrupy) to capture snapshots of their results.

If you add a new test that calls the API you can capture the API response and snapshot like this:
```bash
PYTEST_OPENAI_API_KEY="$(llm keys get openai)" pytest --record-mode once --snapshot-update
```
Then review the new snapshots in `tests/__snapshots__/` to make sure they look correct.
