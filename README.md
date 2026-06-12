# FrankCode Action

A GitHub Action that automatically runs [FrankCode](https://github.com/joeyism/frankcode-py) on your repository whenever you leave a comment on an issue. FrankCode will autonomously investigate your codebase, implement a fix, self-verify the code, and submit a Pull Request.

## Usage

Create a new file in your repository at `.github/workflows/frankcode.yml`:

```yaml
name: FrankCode Issue Responder

on:
  issue_comment:
    types: [created]

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  frankcode-run:
    # Only run on issue comments (not PRs) that contain '@frankcode'
    if: ${{ !github.event.issue.pull_request && contains(github.event.comment.body, '@frankcode') }}
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run FrankCode Action
        uses: joeyism/frankcode-action@v1
        with:
          # Pass your LLM API Key (required)
          api-key: ${{ secrets.GEMINI_API_KEY }}
          
          # Pass the GitHub Token so FrankCode can create a PR (required)
          github-token: ${{ secrets.GITHUB_TOKEN }}
          
          # Optional Configuration (defaults shown below)
          # api-key-name: 'GEMINI_API_KEY'
          # model: 'google/gemini-3.1-pro-preview'
```

## Inputs

| Name | Description | Required | Default |
| --- | --- | --- | --- |
| `api-key` | Your LLM API key (passed as a secret) | `true` | |
| `github-token` | The `GITHUB_TOKEN` secret to open a PR | `true` | |
| `api-key-name` | The environment variable name for your API key | `false` | `GEMINI_API_KEY` |
| `model` | The specific LLM model to use with OpenCode | `false` | `google/gemini-3.1-pro-preview` |

## How it works

1. A user comments on a GitHub Issue mentioning `@frankcode`.
2. The Action triggers and checks out your code.
3. It installs `opencode` and `frankcode-py`.
4. It passes the issue title, description, and the user's comment to FrankCode.
5. FrankCode autonomously does its verification-driven development.
6. A Pull Request is opened with the fixed code!

## Supported Models

By default, the action uses `google/gemini-3.1-pro-preview`. You can switch models by passing a different `model` parameter and passing the corresponding API key under `api-key-name` (e.g. `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`).
