# Claude.ai Reverse-Engineered CLI Client

Security research CLI that replays your Claude.ai browser session over the same REST and SSE endpoints the web app uses. This is not an official API client and it does not use API keys.

Disclaimer: This project is for education and security research. It does not bypass billing, rate limits, account restrictions, or content policies. All requests are processed by Claude.ai and are tied to your account and its limits.

## What It Does
- Captures your Claude.ai session cookies and org ID once, then reuses them from the terminal.
- Streams responses in real time using Server-Sent Events (SSE).
- Keeps a single conversation for a session, with optional cleanup when you exit.
- Uses Chrome TLS fingerprint impersonation via `curl_cffi` to match browser behavior.

## How It Works

```
+-----------+     +-------------+     +------------------+
| Terminal  | --> |  curl_cffi  | --> |   claude.ai API   |
|  You>     |     | TLS spoof   |     | REST + SSE stream |
+-----------+ <-- | SSE parser  | <-- |                  |
```

Flow summary:
- Credential capture: real Chrome is launched via CDP, you log in, the tool extracts cookies and org ID.
- Session replay: the CLI posts to `/api/organizations/{org_id}/chat_conversations/{conv_id}/completion`.
- Streaming: SSE events are parsed and printed as they arrive.

## Features
- Live streaming output via SSE
- Model switching with `/model` during a session
- Session memory with optional cleanup on exit
- Auto-detected IANA timezone
- Local-only CDP (127.0.0.1) with a random ephemeral port
- Credentials stored outside the repo with restricted permissions on Unix
- Rate limit handling with auto-fallback to Haiku
- Batch mode and prompt file support

## Installation

Prereqs:
- Python 3.9+
- Google Chrome (for credential capture)
- A Claude.ai account

Setup:

```bash
git clone https://github.com/Codiologies/Claude_RE_Client.git
cd Claude_RE_Client
pip install -r Requirements.txt
```

Note: This tool uses your installed Chrome for credential extraction, not Playwright's managed browser.

Dependencies:
- `curl_cffi` - HTTP client with Chrome TLS fingerprint impersonation
- `playwright` - CDP connection to your real Chrome (no browser download needed)

## Usage

### Step 1: Extract Credentials (one-time)

Auto-fetch (recommended):

```bash
python Claude_RE_Client.Py --auto-fetch
```

Manual input:

```bash
python Claude_RE_Client.Py --manual
```

### Step 2: Chat

Interactive REPL:

```bash
python Claude_RE_Client.Py
```

Single prompt:

```bash
python Claude_RE_Client.Py --prompt "explain quantum computing"
```

Choose model:

```bash
python Claude_RE_Client.Py --model opus
```

Disable session cleanup:

```bash
python Claude_RE_Client.Py --no-stealth
```

## Interactive Commands

```
/help              Show all commands
/model <name>      Switch model (haiku, sonnet, opus)
/models            List available models
/stealth on|off    Toggle cleanup on exit
/new               Start a fresh conversation
/cleanup           Delete current session conversation
/clear-session     Wipe stored credentials
exit               Quit
```

## Model Shortcuts

| Shortcut     | Full model name              | Tier |
|--------------|------------------------------|------|
| haiku        | claude-haiku-4-5             | FREE |
| sonnet       | claude-sonnet-4-6            | FREE |
| sonnet-4-5   | claude-sonnet-4-5            | FREE |
| haiku-snap   | claude-haiku-4-5-20251001    | FREE |
| sonnet-snap  | claude-sonnet-4-5-20250929   | FREE |
| opus         | claude-opus-4-7              | PRO  |
| opus-3       | claude-3-opus-20240229       | PRO  |

Note: PRO models require a Claude Pro or Max subscription on your account.

## Batch Mode

```bash
python Claude_RE_Client.Py --batch prompts.txt
```

Results are written to `batch_results.json`.

## Prompt File

```bash
python Claude_RE_Client.Py --jailbreak prompt_file.txt
```

## Security Notes

- Your session file grants full access to your Claude.ai account. Treat it like a password.
- The session file is stored outside the repo:
  - Windows: `%APPDATA%/claude_re/claude_session.json`
  - Linux/Mac: `~/.config/claude_re/claude_session.json`
- During `--auto-fetch` and `--login`, Chrome is launched with a local-only debugging port bound to 127.0.0.1.
- Never launch Chrome with a remote debugging port bound to 0.0.0.0.

If you suspect compromise:
1) Run `python Claude_RE_Client.Py --clear-session`
2) Log out of Claude.ai in the browser to invalidate the session
3) Re-authenticate with `--auto-fetch`

## Project Layout

```
Claude_RE_Client/
  Claude_RE_Client.Py
  Requirements.txt
  .gitignore
  README.md
```

Generated at runtime (git-ignored):
- `claude_profile/` - Chrome profile used for CDP
- `last_response.txt` - last response cache
- `batch_results.json` - batch output

## Research Context

This project demonstrates:
- REST endpoint analysis of a production AI service
- SSE streaming protocol parsing
- TLS fingerprint matching via `curl_cffi`
- Credential extraction via CDP from a real browser session
- Session creation and cleanup through REST calls

## License

For educational and security research use only. No warranty is provided. Use at your own risk.
