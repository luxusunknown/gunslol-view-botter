# guns.lol View Botter

Automated view botter for guns.lol profiles. Uses a 1:1 pipeline so every captcha solve is paired directly with a valid WASM token — no wasted credits.

---

## Requirements

| Software | Version |
|----------|---------|
| Python   | 3.10+   |
| Node.js  | 18+     |

---

## Installation

**1. Install Python dependencies**
```bash
pip install -r requirements.txt
```
Or manually:
```bash
pip install aiohttp curl-cffi colorama
```

**2. Verify Node.js is installed**
```bash
node --version
```
If you get an error, download Node.js from https://nodejs.org (LTS version).

**3. Add proxies** (see Proxies section below)

**4. Add your API key** to `config.json`

---

## Files

| File | Purpose |
|------|---------|
| `botter.py` | Main bot — run this |
| `bottertester.py` | Dev/test copy — experiment here before touching botter.py |
| `config.json` | Settings (speed, API keys, solver) |
| `node_wasm_worker.mjs` | WASM solver worker — do not touch |
| `proxies_botter.txt` | Proxies for sending views |
| `proxies_harvester.txt` | Proxies for fetching challenges |
| `requirements.txt` | Python deps |

---

## Getting API Keys

### Capsolver (recommended — fastest)
1. Go to https://capsolver.com ($1.2/1k)
2. Sign up and add balance ($2–5 is plenty to start)
3. Go to **Dashboard → API Key**
4. Copy the key and paste it into `config.json` → `capsolver_api_key`

> Average cost: ~$0.001–0.002 per solve. ~5–10 seconds per solve.

### SolveCaptcha (alternative — slower)
1. Go to https://solvecaptcha.com ($0.8/1k)
2. Sign up and add balance
3. Go to **Dashboard → API Key**
4. Copy the key and paste it into `config.json` → `solvecaptcha_api_key`
5. Set `"solver": "solvecaptcha"` in `config.json`

> Average solve time: 15–30 seconds. Views/sec will be noticeably lower.

---

## Proxies

Place proxies in the files below. One proxy per line.

**`proxies_botter.txt`** — used for sending views (higher quality needed)

**`proxies_harvester.txt`** — used for fetching challenges (lower quality ok)

### Proxy format
```
user:pass@host:port
host:port:user:pass
http://user:pass@host:port
```

Rotating proxies with session support:
```
user-session-abc123:pass@host:port
```

### Recommended providers
- **nodeproxies.xyz** - coming soon (made by me btw)
- **nullproxies.com** - ($0.75/gb - 10gb is A LOT for this tool should last a very long time)

> perfect settings are 10k proxies,sticky,eu-1 or eu-2,rotation time 1 min

---

## config.json

```json
{
  "_comment": "speed_level 1=slowest 10=fastest. Set based on your PC.",
  "speed_level": 3,

  "solver": "capsolver",

  "capsolver_api_key": "CAP-...",
  "solvecaptcha_api_key": "",

  "multiplier": 1.0,
  "loop": false,

  "workers": {
    "wasm_workers": 0,
    "pipeline_workers": 0
  },

  "timeouts": {
    "challenge_max_age_sec": 25,
    "capsolver_max_wait_sec": 90,
    "fetch_timeout_sec": 15,
    "send_timeout_sec": 20
  }
}
```

### Field reference

| Field | Description |
|-------|-------------|
| `speed_level` | 1–10. Controls how many workers spawn. See Speed Levels table. |
| `solver` | `"capsolver"` or `"solvecaptcha"` |
| `capsolver_api_key` | Your Capsolver API key |
| `solvecaptcha_api_key` | Your SolveCaptcha API key |
| `multiplier` | Multiplier on view target. `1.0` = exact count. `1.2` = 20% extra to account for failures. |
| `loop` | `true` to run forever in a loop after hitting target |
| `workers.wasm_workers` | Override WASM pool size (0 = use preset) |
| `workers.pipeline_workers` | Override parallel worker count (0 = use preset) |
| `timeouts.challenge_max_age_sec` | Max age (seconds) of a challenge before it's discarded |
| `timeouts.capsolver_max_wait_sec` | Max wait for a captcha solve before timing out |

---

## Speed Levels

| Level | WASM Workers | Pipeline Workers | Recommended For |
|-------|-------------|-----------------|----------------|
| 1     | 2           | 10              | Very slow PC / testing |
| 2     | 4           | 20              | Old laptop |
| 3     | 6           | 30              | Average PC (default) |
| 4     | 8           | 50              | Good PC |
| 5     | 12          | 70              | Good PC + good proxies |
| 6     | 16          | 100             | Fast PC |
| 7     | 20          | 130             | Fast PC + many proxies |
| 8     | 28          | 160             | High-end PC |
| 9     | 40          | 200             | High-end PC + 10k+ proxies |
| 10    | 64          | 300             | Server / VPS |

> Start at level 3. Increase if your CPU usage is low and you have enough proxies. Decrease if you see many `wasmF` or `fetchF` errors.

---

## Running

**Interactive mode** (prompts for username and view count):
```bash
python botter.py
```

**Command line mode:**
```bash
python botter.py -u <username> -v <views>
```

**Examples:**
```bash
# Send 500 views to "johndoe"
python botter.py -u johndoe -v 500

# Send 1000 views using speed level 7
python botter.py -u johndoe -v 1000 -s 7

# Override API key inline
python botter.py -u johndoe -v 500 -k CAP-xxxxx

# Use SolveCaptcha instead of Capsolver
python botter.py -u johndoe -v 500 --solver solvecaptcha

# Loop until manually stopped
python botter.py -u johndoe -v 1000 -l
```

---

## CLI Arguments

| Argument | Short | Description |
|----------|-------|-------------|
| `--username` | `-u` | Target guns.lol username |
| `--views` | `-v` | Number of views to send |
| `--speed` | `-s` | Speed level 1–10 (overrides config) |
| `--multiplier` | `-m` | View target multiplier |
| `--api-key` | `-k` | API key (overrides config) |
| `--solver` | | `capsolver` or `solvecaptcha` |
| `--workers` | `-w` | Number of pipeline workers (overrides config) |
| `--loop` | `-l` | Loop forever after hitting target |

---

## Progress Bar Explained

```
[████████████████████░░░░░░░░░░░░░░░░░░░░] 50% | Views:500/1,000 | Conv:92% | 1.45v/s | Cap:500 Fail:43
```

| Indicator | Meaning |
|-----------|---------|
| `Views:X/Y` | Views confirmed / target |
| `Conv:X%` | Conversion rate (views confirmed ÷ attempts sent) |
| `Xv/s` | Views delivered per second |
| `Cap:X` | Captchas solved successfully |
| `Cap:X/YF` | X solved, Y failed |
| `Fail:X` | Total failed send attempts |
| `http_429:X` | Rate limited — need more proxies |
| `http_403:X` | Blocked — challenge may be wrong |
| `wasmF:X` | WASM solve failures — Node.js issue |
| `fetchF:X` | Challenge fetch failures — proxy issue |
| `expW:X` | Challenges expired before WASM solved |
| `expC:X` | Challenges expired after WASM, before captcha |
| `netE:X` | Network errors on send |

---

## Troubleshooting

**`Sol:0` / WASM not working**
- Node.js is not installed or not on PATH → install from https://nodejs.org
- Run `node --version` to verify

**`Conv:0%` after 60+ attempts**
- Your proxies are bad or not rotating properly
- If you have no proxies, you're getting rate limited
- Check that `proxies_botter.txt` has valid proxies

**`Cap:X/XXXF` — many captcha failures**
- Wrong API key → double-check `config.json`
- No balance → top up at capsolver.com or solvecaptcha.com
- Capsolver balance: check at https://dashboard.capsolver.com

**`http_403` errors**
- Challenge parsing mismatch — `node_wasm_worker.mjs` may need updating
- Try lowering your speed level (challenge expiring too fast for your PC)

**`http_429` errors**
- Too many requests from same IP → add more proxies or lower speed level

**Very slow views (under 0.2 v/s)**
- WASM workers are the bottleneck → raise `speed_level` for more WASM workers
- Or try speed level 5–7 with `multiplier: 1.0`

**Token waste / balance draining fast**
- This botter uses a 1:1 pipeline: captcha is ONLY called after WASM succeeds
- If you see `expC:X` errors, your PC is too slow → lower speed level

**`ERR_MODULE_NOT_FOUND` on node_wasm_worker.mjs**
- Make sure all files are in the same folder
- Run `node node_wasm_worker.mjs` directly to see the Node.js error

---

## How It Works

1. **Fetch** — Downloads a random guns.lol profile page using a harvester proxy to extract the Turnstile challenge parameters
2. **WASM** — Solves the WebAssembly proof-of-work challenge using a pool of Node.js workers (`node_wasm_worker.mjs`)
3. **Captcha** — Submits the challenge to Capsolver/SolveCaptcha and waits for the Turnstile token (only after WASM succeeds — zero credit waste)
4. **Send** — POSTs to `guns.lol/api/analytics/record` with all required tokens to register a view

The 1:1 pipeline means captcha credits are **never spent unless a valid WASM result is in hand** — no token waste from the capsolver running ahead of the WASM.
