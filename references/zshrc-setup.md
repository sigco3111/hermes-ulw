# zshrc Setup — API Key Configuration

## The Basics

Claude Code needs `ANTHROPIC_API_KEY` in its environment. The cleanest way on macOS:

```bash
# 1. Get your API key from https://console.anthropic.com/
# 2. Add to ~/.zshrc (one line, replace with real key)
echo 'export ANTHROPIC_API_KEY="sk-ant-api03-YOUR_KEY_HERE"' >> ~/.zshrc

# 3. Apply now
source ~/.zshrc

# 4. Verify
echo "${ANTHROPIC_API_KEY:0:10}..."  # masked preview

# 5. Test Claude Code
zsh -i -c 'claude auth status'
```

**Expected output if working:**
```json
{
  "loggedIn": true,
  "authMethod": "oauth_token",
  "apiProvider": "firstParty",
  "apiKeySource": "ANTHROPIC_API_KEY"
}
```

## Using minimax Token Plan (Proxy)

If you're routing through minimax or similar aggregator:

```bash
# minimax uses ANTHROPIC_AUTH_TOKEN instead of ANTHROPIC_API_KEY
echo 'export ANTHROPIC_AUTH_TOKEN="minimax_token_here"' >> ~/.zshrc

# Workaround: symlink the variable name
echo 'export ANTHROPIC_API_KEY="$ANTHROPIC_AUTH_TOKEN"' >> ~/.zshrc

source ~/.zshrc
zsh -i -c 'claude auth status'
```

> ⚠️ Check whether minimax officially supports Claude Code CLI before depending on this.

## Using OpenRouter (or Any OpenAI-Compatible Proxy)

```bash
# Get an OpenRouter key from https://openrouter.ai/
export ANTHROPIC_API_KEY="sk-or-..."
export ANTHROPIC_BASE_URL="https://openrouter.ai/api"
export ANTHROPIC_AUTH_TOKEN="sk-or-..."  # sometimes required

# Verify
zsh -i -c 'claude auth status'
```

> ⚠️ Not all Claude Code features work through proxies (extended thinking, certain tools). Test before relying on it for production.

## Multi-line API Key With Special Characters

If your key contains `$`, `"`, `\` or other shell-special chars, use single quotes only and escape carefully:

```bash
# Safe: keys without quote chars
export ANTHROPIC_API_KEY='sk-ant-api03-real_key_with_no_specials'

# Risky: keys with literal "$" can be expanded by zsh
# Workaround: use a file or 1Password CLI
```

## Using 1Password CLI (Recommended)

```bash
# 1Password SDK retrieves on demand
export ANTHROPIC_API_KEY="$(op read 'op://Personal/Anthropic/api_key')"
```

Add to `.zshrc`, re-source. No plaintext secrets in `.zshrc`.

## Using macOS Keychain

```bash
# Store once
security add-generic-password -a "$USER" -s "anthropic-api-key" -w "sk-ant-YOUR_KEY"

# Retrieve in .zshrc
export ANTHROPIC_API_KEY="$(security find-generic-password -a "$USER" -s "anthropic-api-key" -w)"
```

Added to `.zshrc`, key never touches disk plaintext.

## Why `zsh -i -c` in Invocation?

Each `terminal` invocation from Hermes creates a new bash process. That bash doesn't read `~/.zshrc` automatically. By wrapping in `zsh -i`, you force sourcing:

```bash
# Without zsh -i: fails with "Invalid API key"
terminal(command="claude -p 'hello'")

# With zsh -i: works
terminal(command="zsh -i -c 'claude -p \"hello\"'")
```

> 💡 **Alternative**: Add the key to `~/.bash_profile` so plain bash sessions see it:

```bash
cat >> ~/.bash_profile << 'EOF'
if [ -f ~/.zshrc ]; then
  source ~/.zshrc
fi
EOF
```

> ⚠️ `source ~/.zshrc` from bash sometimes fails (zsh-specific syntax). Safer: just use `zsh -i -c '...'` wrapper, or duplicate the line in `~/.bashrc`.

## Troubleshooting

### Symptom: "Invalid API key"

Check these in order:

```bash
# 1. Is the variable set?
zsh -i -c 'echo "key: ${ANTHROPIC_API_KEY:0:10}..."'

# 2. Is zshrc correct? (value masked)
grep "ANTHROPIC_API_KEY" ~/.zshrc | sed 's/"[^"]*"/"***"/'

# 3. Did you source after editing?
source ~/.zshrc

# 4. Is Claude Code looking at the right env?
zsh -i -c 'claude auth status'

# 5. Is it actually a valid key?
# Try logging into https://console.anthropic.com/ with the same key
```

### Symptom: Multiple key paths

If you have both `ANTHROPIC_API_KEY` and `ANTHROPIC_AUTH_TOKEN`, Claude Code prefers `ANTHROPIC_API_KEY`. The variable name is the only differentiator.

### Symptom: Key visible in plain `~/.zshrc` file

If you don't want the key in plaintext:

1. Move key to Keychain (see above)
2. Move key to 1Password
3. Add `chmod 600 ~/.zshrc` (single-user protection only)
4. Add `~/.zshrc` to gitignore (but you lose it on new machines)

## Related Environment Variables

| Var | Purpose |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key (required) |
| `ANTHROPIC_AUTH_TOKEN` | OAuth-style token (alternative) |
| `ANTHROPIC_BASE_URL` | Custom API endpoint (proxies) |
| `CLAUDE_CODE_EFFORT_LEVEL` | Default effort: low\|medium\|high\|max\|auto |
| `MAX_THINKING_TOKENS` | Cap thinking tokens (0 = disable thinking) |
| `MAX_MCP_OUTPUT_TOKENS` | Cap MCP output (default varies) |
| `CLAUDE_CODE_NO_FLICKER=1` | Alt-screen rendering, no flicker |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | Strip credentials from subprocesses |

## Pinning Effort / Model for All Sessions

```bash
# Default to opus + high effort for everything (expensive but capable)
echo 'export CLAUDE_CODE_EFFORT_LEVEL=high' >> ~/.zshrc
echo 'export ANTHROPIC_DEFAULT_MODEL=opus' >> ~/.zshrc

# Or default to haiku (cheap) for trivial
echo 'export ANTHROPIC_DEFAULT_MODEL=haiku' >> ~/.zshrc
```

Per-call override still works:

```bash
claude -p "..." --model sonnet  # overrides default
```
