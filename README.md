# guns.lol View Botter

Fast, automated view botter for guns.lol profiles. Solves challenges server-side with zero credit waste.

> **Note:** Around 3-5% of views may fail to deliver due to network variance and proxy rotation. It is recommended to set `multiplier` to `1.05` or `1.10` in `config.json` to automatically overshoot and guarantee your target count.

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

**2. Verify Node.js is installed**
```bash
node --version
```
Download from https://nodejs.org (LTS) if not installed.

**3. Add proxies** (see Proxies section)

**4. Add your API key** to `config.json`

---

## Files

| File | Purpose |
|------|---------|
| `botter.py` | Main bot |
| `config.json` | All settings |
| `node_wasm_worker.mjs` | Internal solver worker, do not modify |
| `proxies_botter.txt` | Proxies for sending views |
| `proxies_harvester.txt` | Proxies for fetching challenges |
| `requirements.txt` | Python dependencies |

---

## API Keys

### Capsolver (recommended)
1. Go to https://capsolver.com
2. Sign up and add balance
3. Go to Dashboard and copy your API key
4. Paste it into `config.json` under `capsolver_api_key`

Pricing: ~$1.20 per 1,000 solves. Average solve time: 3-7 seconds.

### SolveCaptcha (alternative)
1. Go to https://solvecaptcha.com
2. Sign up and add balance
3. Copy your API key into `config.json` under `solvecaptcha_api_key`
4. Set `"solver": "solvecaptcha"` in `config.json`

Pricing: ~$0.80 per 1,000 solves. Average solve time: 11 seconds. Views per second will be lower.

---

## Proxies

One proxy per line. Two separate files are used:

**`proxies_botter.txt`** - used for view delivery (higher quality recommended)

**`proxies_harvester.txt`** - used for challenge fetching (lower quality is fine)

### Accepted formats
```
host:port
host:port:user:pass
user:pass@host:port
http://user:pass@host:port
socks5://user:pass@host:port
```

Rotating proxies with session support:
```
user-session-abc123:pass@host:port
```

### Recommended providers
- **nullproxies.com** - $0.75/gb (10gb lasts a very long time for this tool)
- **nodeproxies.xyz** - coming soon
- **any higher quality ones for constant uptime** - (most recommended)

Recommended settings: 10k proxies, sticky, EU-1 or EU-2, 1 minute rotation time.

---

## config.json

```json
{
  "speed_level": 7,
  "solver": "capsolver",
  "capsolver_api_key": "CAP-...",
  "solvecaptcha_api_key": "",
  "multiplier": 1.0,
  "loop": false,
  "workers": {
    "fetch_workers": 0,
    "wasm_workers": 0,
    "solver_workers": 0,
    "send_workers": 0
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
| `speed_level` | Controls throughput. Recommended: 7. See Speed Levels below. |
| `solver` | `"capsolver"` or `"solvecaptcha"` |
| `capsolver_api_key` | Your Capsolver API key |
| `solvecaptcha_api_key` | Your SolveCaptcha API key |
| `multiplier` | Multiplier on the view target. `1.05` sends 5% extra to cover the small failure rate. |
| `loop` | `true` to keep running in a loop after hitting the target |
| `workers.*` | Leave all at `0` to use the speed preset automatically. Set any to a number to override that stage. |
| `timeouts.*` | Network timeouts. Defaults work well for most setups. |

---

## Speed Levels

Speed level controls how many parallel workers are spawned. Higher levels use more CPU and proxies.

| Level | Recommended For |
|-------|----------------|
| 1-3   | Old or low-end PC, testing |
| 4-6   | Average PC |
| **7** | **Recommended default - fast PC with decent proxies** |
| 8-10  | High-end PC with many proxies |
| 11-15 | Server / VPS (beast tiers) |

Start at 7. Go up if CPU usage is low and you have enough proxies. Go down if you see a lot of errors.

---

## Running

Interactive (prompts for input):
```bash
python botter.py
```

Command line:
```bash
python botter.py -u <username> -v <views>
```

Examples:
```bash
python botter.py -u johndoe -v 500

python botter.py -u johndoe -v 1000 -s 7

python botter.py -u johndoe -v 500 -k CAP-xxxxx

python botter.py -u johndoe -v 500 --solver solvecaptcha

python botter.py -u johndoe -v 1000 -l
```

---

## CLI Arguments

| Argument | Short | Description |
|----------|-------|-------------|
| `--username` | `-u` | Target guns.lol username |
| `--views` | `-v` | Number of views to send |
| `--speed` | `-s` | Speed level 1-15 (overrides config) |
| `--multiplier` | `-m` | View target multiplier |
| `--api-key` | `-k` | API key (overrides config) |
| `--solver` | | `capsolver` or `solvecaptcha` |
| `--loop` | `-l` | Loop forever after hitting target |

---

## Progress Display

```
[====================] 50% | 500/1,000 | 1.45v/s | Conv:92% | Cap:500
```

| Indicator | Meaning |
|-----------|---------|
| `X/Y` | Views confirmed / target |
| `Xv/s` | Views delivered per second |
| `Conv:X%` | Delivery rate |
| `Cap:X` | Captchas solved |
| `http_429:X` | Rate limited, need more proxies |
| `http_403:X` | Challenge rejected, try lowering speed |
| `fetchF:X` | Challenge fetch failures, proxy issue |
| `netE:X` | Network errors on send |

---

## Troubleshooting

**Bot starts but no views go through**
- Proxies are bad or not formatted correctly
- Check that both proxy files have valid proxies

**Many captcha failures**
- Wrong API key, check `config.json`
- No balance, top up at capsolver.com

**`http_403` errors**
- Try lowering speed level, challenges may be expiring before your PC processes them

**`http_429` errors**
- Too many requests from same IP, add more proxies or lower speed level

**Very slow views**
- Raise `speed_level` for more throughput
- Recommended starting point is level 7

**Node.js errors on startup**
- Node.js is not installed or not on PATH
- Run `node --version` and install from https://nodejs.org if missing
