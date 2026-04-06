# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ice Breaker is a LangChain-powered Flask web app that generates personalized ice breakers by analyzing LinkedIn and Twitter profiles. Given a person's name, it discovers their social profiles via Tavily search, scrapes profile data, and uses OpenAI GPT to produce summaries, topics of interest, and conversation starters.

## Commands

```bash
# Install dependencies
pipenv install

# Run the app (serves on http://localhost:5000)
pipenv run python app.py

# Run tests
pipenv run pytest .

# Lint / format
pipenv run pylint <file>
pipenv run black <file>
pipenv run isort <file>
```

## Architecture

The pipeline flows through four stages:

1. **Agent lookup** (`agents/`) - ReAct agents (LangChain + GPT-4o-mini) use Tavily search to find LinkedIn/Twitter profile URLs from a person's name
2. **Data scraping** (`third_parties/`) - LinkedIn profiles scraped via Scrapin.io API; Twitter tweets fetched via Tweepy (with a mock gist fallback used by default in `ice_breaker.py`)
3. **LLM chains** (`chains/custom_chains.py`) - Three parallel RunnableSequence chains (summary, interests, ice breakers) each use GPT-3.5-turbo with Pydantic output parsers
4. **Web interface** - Flask app (`app.py`) serves a single-page UI (`templates/index.html`); POST to `/process` with a `name` field triggers the full pipeline and returns JSON

Key integration point: `ice_breaker.py:ice_break_with()` orchestrates the entire flow - agent lookups, scraping, and all three chain invocations.

## Environment Variables

Required in `.env`: `OPENAI_API_KEY`, `SCRAPIN_API_KEY`, `TAVILY_API_KEY`. Twitter keys and LangSmith tracing are optional (see README for full list).

## GitHub Actions

- `claude.yml` - Responds to `@claude` mentions in issues/PRs via claude-code-action
- `claude-code-review.yml` - Automatic code review on PR open/sync using the code-review plugin
