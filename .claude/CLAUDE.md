# GitLearn Project Context

## Project Overview
GitLearn is a GitHub Action that automatically learns from merged pull requests and maintains AI context documentation. It analyzes PR diffs using AI to extract meaningful architectural and operational insights, then appends them to a unified context file (`claude.md`).

## Architecture
- **Type**: GitHub Action workflow (single-file tool)
- **Trigger**: PR merge events to main/master branch
- **Main File**: `.github/workflows/gitlearn.yml`

## Key Features
- Multi-provider AI support (Anthropic, OpenAI, OpenRouter)
- Intelligent learning extraction with filtering
- Unified context file system with symlinks
- Extended thinking capability for Anthropic models

## Development Guidelines
- Keep the workflow file well-commented
- Test changes in a fork before merging
- Follow YAML best practices for readability
- Document any new configuration variables in README

## Configuration Variables
- `GITLEARN_MODEL`: Override default AI model
- `GITLEARN_MAX_TOKENS`: Response token limit (default: 2048)
- `GITLEARN_THINKING_BUDGET`: Anthropic thinking tokens (default: 10000)
- `GITLEARN_SKIP_LABELS`: Custom PR labels to skip
- `GITLEARN_API_BASE`: Custom OpenAI-compatible endpoint

## Prompt Engineering Commands Available
Use `/project:improve-prompt` to enhance any prompt with best practices.
Use `/project:analyze-prompt` to identify weaknesses in prompts.
Use `/project:chain-of-thought` to add reasoning steps.
Use `/project:few-shot` to add examples to prompts.
Use `/project:prompt-template` to generate structured templates.
