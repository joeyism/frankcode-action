# FrankCode Action

A GitHub Action that automatically runs [FrankCode](https://github.com/joeyism/frankcode) on your repository whenever you leave a comment on an issue. FrankCode will autonomously investigate your codebase, implement a fix, self-verify the code, and submit a Pull Request.

## Basic Usage (Gemini)

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
        uses: joeyism/frankcode-action@master
        with:
          api-key: ${{ secrets.GOOGLE_GENERATIVE_AI_API_KEY }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Name | Description | Required | Default |
| --- | --- | --- | --- |
| `api-key` | Your primary LLM API key (passed as a secret) | `true` | |
| `github-token` | The `GITHUB_TOKEN` secret to open a PR | `true` | |
| `api-key-name` | The environment variable name for your API key | `false` | `GOOGLE_GENERATIVE_AI_API_KEY` |
| `model` | The specific LLM model identifier to use | `false` | `google/gemini-3.1-pro-preview` |
| `env` | Additional environment variables as a multiline string | `false` | `""` |

## Use Cases & Supported Providers

FrankCode relies on OpenCode natively, which means it supports any AI provider that OpenCode supports. Below are common configurations for different setups:

### 1. Using OpenAI (GPT-4o)
To use OpenAI, you must supply an OpenAI API key, override the `api-key-name` to what OpenCode expects, and specify the `model`.

```yaml
      - name: Run FrankCode Action
        uses: joeyism/frankcode-action@master
        with:
          api-key: ${{ secrets.OPENAI_API_KEY }}
          api-key-name: 'OPENAI_API_KEY'
          model: 'openai/gpt-4o'
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 2. Using Anthropic (Claude 3.5 Sonnet)
Similar to OpenAI, you must map the API key to Anthropic's expected environment variable name.

```yaml
      - name: Run FrankCode Action
        uses: joeyism/frankcode-action@master
        with:
          api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          api-key-name: 'ANTHROPIC_API_KEY'
          model: 'anthropic/claude-3.5-sonnet'
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 3. Using the OpenCode API (Qwen, DeepSeek, etc.)
If you use the OpenCode API gateway to access open-source or custom models.

```yaml
      - name: Run FrankCode Action
        uses: joeyism/frankcode-action@master
        with:
          api-key: ${{ secrets.OPENCODE_API_KEY }}
          api-key-name: 'OPENCODE_API_KEY'
          model: 'opencode-go/qwen3.7-max'
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 4. Passing Multiple API Keys (For Plugins, MCPs, or Fallbacks)
If you need to supply multiple API keys (for example, you are using Gemini as your primary model, but your custom MCP servers require a `CONTEXT7_API_KEY`, or you have secondary fallback models), you can use the `env` input block.

```yaml
      - name: Run FrankCode Action
        uses: joeyism/frankcode-action@master
        with:
          api-key: ${{ secrets.GOOGLE_GENERATIVE_AI_API_KEY }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          env: |
            OPENAI_API_KEY=${{ secrets.OPENAI_API_KEY }}
            CONTEXT7_API_KEY=${{ secrets.CONTEXT7_API_KEY }}
            MY_CUSTOM_SECRET=foobar
```

## Troubleshooting

### "GitHub Actions is not permitted to create or approve pull requests"
If your Action successfully runs FrankCode, but fails at the very end when trying to create the Pull Request with this error:
1. Go to your repository **Settings** on GitHub.
2. In the left sidebar, click on **Actions** > **General**.
3. Scroll down to the **Workflow permissions** section.
4. Check the box that says **"Allow GitHub Actions to create and approve pull requests"**.
5. Click **Save** and re-run your job.

### Agent Stuck / Timeout Errors
If a job is canceled or takes an unexpectedly long time, FrankCode will automatically dump its entire working memory and session logs as a `.zip` artifact attached to the GitHub Action run. You can download this artifact directly from the GitHub Actions UI to inspect `opencode.txt` and see exactly where the model got stuck.

