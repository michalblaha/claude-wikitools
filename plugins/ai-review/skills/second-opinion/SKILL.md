---
name: second-opinion
description: "Získej druhý názor od modelů OpenAI (Codex), Google (Gemini) nebo Anthropic (Claude Code). Použij při validaci architektonických rozhodnutí, kontrole bezpečnosti kódu, porovnávání implementačních přístupů nebo když multi-model konsenzus přidá hodnotu. Podporuje všechny tři hlavní AI CLI nástroje pro maximální flexibilitu."
---

# Second Opinion via Codex, Gemini & Claude Code CLI

Get external AI perspective from OpenAI (via Codex CLI), Google (via Gemini CLI), or Anthropic (via Claude Code CLI) to validate decisions or compare approaches.

## Quick Patterns

### Using Codex (OpenAI)

#### Simple Question
```bash
codex -a never exec --skip-git-repo-check -m gpt-5.5 -c 'model_reasoning_effort="high"' --output-last-message /tmp/claude/answer.txt "Your question here"
cat /tmp/claude/answer.txt
```

#### Structured Analysis (Recommended)

**IMPORTANT:** When using `--output-schema`, ALL objects (including nested ones) must have:
- `"additionalProperties": false`
- `"required": [...]` with all property names

```bash
cat > /tmp/claude/schema.json << 'EOF'
{
  "type": "object",
  "properties": {
    "assessment": { "type": "string" },
    "strengths": { "type": "array", "items": { "type": "string" } },
    "concerns": { "type": "array", "items": { "type": "string" } },
    "recommendation": { "type": "string" }
  },
  "required": ["assessment", "strengths", "concerns", "recommendation"],
  "additionalProperties": false
}
EOF

codex -a never exec --skip-git-repo-check -m gpt-5.5 -c 'model_reasoning_effort="high"'  --output-schema /tmp/claude/schema.json \
  --output-last-message /tmp/claude/result.json \
  "Analyze [topic]. Provide structured assessment."

cat /tmp/claude/result.json
```

### Nested Objects Example
```bash
# Nested objects MUST also have additionalProperties: false
cat > /tmp/claude/nested_schema.json << 'EOF'
{
  "type": "object",
  "properties": {
    "summary": { "type": "string" },
    "details": {
      "type": "object",
      "properties": {
        "score": { "type": "string" },
        "items": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["score", "items"],
      "additionalProperties": false
    }
  },
  "required": ["summary", "details"],
  "additionalProperties": false
}
EOF
```

### Using Gemini (Google)

#### Simple Question
```bash
gemini -p "Your question here" --output-format text > /tmp/claude/answer.txt
cat /tmp/claude/answer.txt
```

#### JSON Output
```bash
gemini -p "Analyze [topic]. Respond in JSON with: assessment (string), strengths (array), concerns (array), recommendation (string)" \
  --output-format json > /tmp/claude/result.json
cat /tmp/claude/result.json
```

**Note:** Gemini CLI doesn't support JSON Schema validation like Codex. For strict structured output, use Codex with `--output-schema`.

### Using Claude Code (Anthropic)

#### Simple Question
```bash
claude -p "Your question here" --model claude-opus-4-6 --output-format text > /tmp/claude/answer.txt
cat /tmp/claude/answer.txt
```

#### JSON Output
```bash
claude -p "Analyze [topic]. Respond in JSON with: assessment (string), strengths (array), concerns (array), recommendation (string)" \
  --model claude-opus-4-6 --output-format json > /tmp/claude/result.json
cat /tmp/claude/result.json
```

#### With Cheaper Model (Simple Tasks)
```bash
claude -p "Your question here" --model claude-sonnet-4-5-20250929 --output-format text > /tmp/claude/answer.txt
cat /tmp/claude/answer.txt
```

**Note:** Claude Code CLI uses `--print` (`-p`) for non-interactive mode. The spawned instance runs independently without access to the current session context, providing a genuinely fresh perspective. JSON output via `--output-format json` wraps the response in a structured message object.

## Use Cases

### Architecture Review

**Codex:**
```bash
codex -a never exec --skip-git-repo-check -m gpt-5.5 -c 'model_reasoning_effort="high"' --output-last-message /tmp/claude/arch.txt \
  "Review this architecture decision: [description].
   Assess: scalability, maintainability, security risks, alternatives."
cat /tmp/claude/arch.txt
```

**Gemini:**
```bash
gemini -p "Review this architecture decision: [description].
  Assess: scalability, maintainability, security risks, alternatives." \
  --output-format text > /tmp/claude/arch.txt
cat /tmp/claude/arch.txt
```

**Claude Code:**
```bash
claude -p "Review this architecture decision: [description].
  Assess: scalability, maintainability, security risks, alternatives." \
  --model claude-opus-4-6 --output-format text > /tmp/claude/arch.txt
cat /tmp/claude/arch.txt
```

### Security Audit

**Codex:**
```bash
codex -a never exec --skip-git-repo-check -m gpt-5.5 -c 'model_reasoning_effort="high"' --output-last-message /tmp/claude/security.txt \
  "Security review of [file/code]:
   - Input validation
   - Authentication/authorization
   - Data exposure risks
   Provide specific vulnerabilities and fixes."
cat /tmp/claude/security.txt
```

**Gemini:**
```bash
gemini -p "Security review of [file/code]:
  - Input validation
  - Authentication/authorization
  - Data exposure risks
  Provide specific vulnerabilities and fixes." \
  --output-format text > /tmp/claude/security.txt
cat /tmp/claude/security.txt
```

**Claude Code:**
```bash
claude -p "Security review of [file/code]:
  - Input validation
  - Authentication/authorization
  - Data exposure risks
  Provide specific vulnerabilities and fixes." \
  --model claude-opus-4-6 --output-format text > /tmp/claude/security.txt
cat /tmp/claude/security.txt
```

### Code Review

**Codex:**
```bash
codex -a never exec --skip-git-repo-check -m gpt-5.5 -c 'model_reasoning_effort="high"' --output-last-message /tmp/claude/review.txt \
  "Review [file] for: bugs, performance issues, maintainability.
   Provide line-level recommendations."
cat /tmp/claude/review.txt
```

**Gemini:**
```bash
gemini -p "Review [file] for: bugs, performance issues, maintainability.
  Provide line-level recommendations." \
  --output-format text > /tmp/claude/review.txt
cat /tmp/claude/review.txt
```

**Claude Code:**
```bash
claude -p "Review [file] for: bugs, performance issues, maintainability.
  Provide line-level recommendations." \
  --model claude-opus-4-6 --output-format text > /tmp/claude/review.txt
cat /tmp/claude/review.txt
```

## Key Options

### Codex CLI Options

| Option | Purpose |
|--------|---------|
| `-m gpt-5.5` | Best model (recommended) |
| `-c 'model_reasoning_effort="high"'` | Reasoning level high. Other values 'medium','low','xhigh' | 
| `-m o4-mini` | Faster, cheaper for simple tasks |
| `--output-schema file.json` | Structured JSON with schema validation |
| `--output-last-message file.txt` | Save response to file |
| `-i image.png` | Include image for analysis |

### Gemini CLI Options

| Option | Purpose |
|--------|---------|
| `--model gemini-3-pro-preview` | Best reasoning (explicit selection) |
| `--model gemini-3-flash-preview` | Fast and capable |
| Auto-routing (default) | CLI selects best model automatically |
| `--output-format text` | Plain text output |
| `--output-format json` | JSON output (no schema validation) |
| `-p "prompt"` | Non-interactive mode (required) |

### Claude Code CLI Options

| Option | Purpose |
|--------|---------|
| `--model claude-opus-4-6` | Most capable model (recommended) |
| `--model claude-sonnet-4-5-20250929` | Balanced speed and quality |
| `--model claude-haiku-4-5-20251001` | Fastest, cheapest for simple tasks |
| `--output-format text` | Plain text output (recommended) |
| `--output-format json` | JSON message object output |
| `--output-format stream-json` | Streaming JSON output |
| `-p "prompt"` | Non-interactive print mode (required) |
| `--max-turns N` | Limit agentic turns (default: no limit) |

## Provider Comparison

| Use Case | Codex (OpenAI) | Gemini (Google) | Claude Code (Anthropic) |
|----------|---------------|-----------------|------------------------|
| Complex architectural review | `gpt-5.5` | `gemini-3-pro-preview` or auto | `claude-opus-4-6` |
| Fast code review | `o4-mini` | `gemini-3-flash-preview` or auto | `claude-haiku-4-5-20251001` |
| Security audit (deep) | `gpt-5.5` | Auto-routing (recommended) | `claude-opus-4-6` |
| Structured output (schema) | `gpt-5.5` + schema | N/A (no schema support) | N/A (no schema support) |
| Balanced quality/speed | `gpt-5.5` | Auto-routing | `claude-opus-4-6` |
| Multi-provider consensus | All three! Run all and compare | All three! Run all and compare | All three! Run all and compare |

## Presenting Results

1. Label clearly which provider was used:
   - "Second opinion (OpenAI/Codex - gpt-5.5)"
   - "Second opinion (Google/Gemini - gemini-3-pro)"
   - "Second opinion (Anthropic/Claude Code - claude-opus-4-6)"
   - "Consensus (Codex + Gemini + Claude Code)" if using all three
2. Compare with your own analysis
3. Highlight areas of agreement and disagreement
4. Synthesize recommendation based on multiple perspectives
5. Optional: Get consensus by asking all providers and comparing outputs

## Prerequisites

Verify CLIs are available before use:
```bash
# Check Codex
codex --version || echo "Codex CLI not installed"

# Check Gemini
gemini --version || echo "Gemini CLI not installed"

# Check Claude Code
claude --version || echo "Claude Code CLI not installed"
```

**Authentication:**
- **Codex:** `codex login` or set `OPENAI_API_KEY` env var
- **Gemini:** Already authenticated via Google OAuth or set `GEMINI_API_KEY` env var
- **Claude Code:** Already authenticated via `claude login` or set `ANTHROPIC_API_KEY` env var

**Codex — mandatory `--skip-git-repo-check` flag:** Every `codex exec` call from this skill **must** include `--skip-git-repo-check` (and `--ask-for-approval never` for full non-interactivity). Codex defaults to refusing to run outside a Git repository as a safety check against unintended writes to unrelated files. This skill never writes via Codex — it only consumes the response from `--output-last-message` — so the check is unnecessary and would otherwise break calls when the caller's working directory happens to be outside a repo. Without these flags, calls fail with *"not inside a Git repository"* or hang on approval prompts.

**Fallback when `codex exec` fails:** If a Codex call fails despite the flags above (timeout, auth issue, runtime error), delegate the request to the `codex:codex-rescue` skill / subagent. It runs Codex through the `codex-companion.mjs` helper with more robust orchestration and handles many edge cases. Use only one retry; if that also fails, switch to a different provider (Gemini or Claude Code) and note the Codex failure in the *Uncertainties* section of your final answer.

## Multi-Provider Consensus Example

Get second opinions from all three providers and compare:

```bash
# 1. Ask Codex
codex -a never exec --skip-git-repo-check -m gpt-5.5 -c 'model_reasoning_effort="high"' --output-last-message /tmp/claude/codex_opinion.txt \
  "Should we use Redis or PostgreSQL for session storage in e-commerce app?"

# 2. Ask Gemini
gemini -p "Should we use Redis or PostgreSQL for session storage in e-commerce app?" \
  --output-format text > /tmp/claude/gemini_opinion.txt

# 3. Ask Claude Code
claude -p "Should we use Redis or PostgreSQL for session storage in e-commerce app?" \
  --model claude-opus-4-6 --output-format text > /tmp/claude/claude_opinion.txt

# 4. Compare outputs
echo "=== Codex (OpenAI) ===" && cat /tmp/claude/codex_opinion.txt
echo ""
echo "=== Gemini (Google) ===" && cat /tmp/claude/gemini_opinion.txt
echo ""
echo "=== Claude Code (Anthropic) ===" && cat /tmp/claude/claude_opinion.txt
```

Analyze agreement/disagreement across all three providers and synthesize final recommendation.