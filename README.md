# copilot-token-usage

Parse [GitHub Copilot CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli) log files and produce a machine-readable JSON summary of token usage per session and model.

## Example output

```
$ copilot-token-usage --summary --days 7

--- Token Usage Summary ---
  claude-opus-4.6-1m: 47.9m in, 113.5k out, 43.4m cached (Est. 600.1 Premium requests)
  claude-opus-4.6: 125.4m in, 439.3k out, 111.0m cached (Est. 1573.6 Premium requests)
  claude-haiku-4.5: 5.2m in, 121.8k out, 3.7m cached (Est. 22.1 Premium requests)

  Total: 246.1m in, 959.8k out, 220.6m cached
  Est. Premium requests: 3046.1
```

It also writes a detailed JSON file (default: `~/.copilot/token-usage.json`) with per-session, per-model breakdowns.

## Install

```bash
# Clone the repo
git clone https://github.com/srt32/copilot-token-usage.git
cd copilot-token-usage

# Copy to your scripts directory and symlink into PATH
cp copilot-token-usage ~/.copilot/scripts/
chmod +x ~/.copilot/scripts/copilot-token-usage
ln -sf ~/.copilot/scripts/copilot-token-usage ~/bin/copilot-token-usage
```

Or just run it directly:

```bash
./copilot-token-usage --summary
```

**Requirements:** Python 3.7+ (no external dependencies).

## Usage

```
copilot-token-usage [--days N] [--output PATH] [--pretty] [--session SESSION_ID] [--summary]
```

| Flag | Description |
|------|-------------|
| `--days N` | Only process log files modified within the last N days (default: all) |
| `--output PATH` / `-o` | Output JSON file path (default: `~/.copilot/token-usage.json`) |
| `--pretty` | Pretty-print JSON output |
| `--session ID` | Filter to a specific session UUID |
| `--summary` | Print a human-readable summary to stderr |

## JSON schema

```json
{
  "generated_at": "2026-02-18T20:01:39Z",
  "log_files_scanned": 122,
  "sessions": [
    {
      "session_id": "e7477e26-a9ff-4bd2-9429-79e6ed6853df",
      "log_files": ["process-1771444458732-41822.log"],
      "first_call_at": "2026-02-18T19:54:40.943Z",
      "last_call_at": "2026-02-18T19:59:50.600Z",
      "models": {
        "claude-opus-4.6-1m": {
          "calls": 128,
          "input_tokens": 7600000,
          "output_tokens": 17300,
          "cached_tokens": 6500000,
          "total_duration_ms": 123456,
          "estimated_premium_requests": 84.0
        }
      },
      "totals": {
        "calls": 128,
        "input_tokens": 7600000,
        "output_tokens": 17300,
        "cached_tokens": 6500000,
        "total_duration_ms": 123456,
        "estimated_premium_requests": 84.0
      }
    }
  ],
  "grand_totals": {
    "total_sessions": 72,
    "total_calls": 2848,
    "total_input_tokens": 246100000,
    "total_output_tokens": 959800,
    "total_cached_tokens": 220600000,
    "total_duration_ms": 9876543,
    "estimated_premium_requests": 3046.1,
    "by_model": { "..." : "..." }
  }
}
```

## How it works

Copilot CLI writes telemetry events to `~/.copilot/logs/process-*.log`. Each model call produces a `[Telemetry] cli.model_call` entry with token counts, model name, session ID, and duration. This script parses those entries and aggregates them.