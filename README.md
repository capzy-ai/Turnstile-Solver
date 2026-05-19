<div align="center">

<img src="https://capzy.ai/capzy-logo.svg" alt="Capzy" width="220" />

# Cloudflare Turnstile Solver

**Bypass Cloudflare Turnstile with a single HTTP call. ~3s solves.**

[![Solve cost](https://img.shields.io/badge/from-%240.001%20%2F%20solve-%23ff5d2a)](https://capzy.ai/pricing)
[![Speed](https://img.shields.io/badge/avg%20solve-~3%20seconds-%2322c55e)](https://capzy.ai/products/turnstile)
[![Uptime](https://img.shields.io/badge/uptime-99.9%25-%2322c55e)](https://capzy.ai/status)
[![License: MIT](https://img.shields.io/badge/license-MIT-%23ff5d2a)](LICENSE)

[Live Demo](https://capzy.ai/products/turnstile/demo) ·
[Get Free $0.10 Credit](https://capzy.ai/auth/register) ·
[Dashboard](https://capzy.ai/dashboard) ·
[Full Docs](https://capzy.ai/docs) ·
[Pricing](https://capzy.ai/pricing)

</div>

---

## What this repo is

Copy-pasteable examples for solving **Cloudflare Turnstile** through the
[Capzy](https://capzy.ai) HTTP API — no SDK required. Pure curl, Python,
and Node.js using the raw API. Easy to read, easy to port, easy to audit.

## What is Cloudflare Turnstile?

Cloudflare Turnstile is a smart captcha widget deployed on 25M+ sites — login, signup, checkout, contact forms. Runs invisible behavioral checks in the background; either auto-passes or shows a simple checkbox. Capzy returns valid Turnstile tokens via API for authorized testing and automation.

## Why Capzy

- **From $0.001 per solve.** Flat pricing — no tiers, no retainer, no monthly minimum.
- **~3 seconds average solve.** Production-grade speed.
- **Drop-in compatible.** `createTask` / `getTaskResult` protocol. If your code already speaks the standard solver shape, swap the host to `https://api.capzy.ai`.
- **$0.10 in real credits on sign-up.** No card. 100 free test solves.

## Pricing

| Task type | When to use | Cost / solve |
|-----------|-------------|-------------:|
| `AntiTurnstileTaskProxyLess`             | Proxyless (Capzy supplies the IP) | **$0.001**   |
| `AntiTurnstileTask`                       | You supply the proxy              | **$0.001**   |

For consistency across the target site, use the proxy variant with the
**same proxy your session is already running through** — the solver
mints the token from that IP, so when you submit it back through the
same proxy everything looks consistent.

## 60-second quickstart

```bash
# 1. Sign up — gets you $0.10 in free credits (100 solves)
open https://capzy.ai/auth/register

# 2. Copy your API key from the dashboard
#    https://capzy.ai/dashboard/api-keys

# 3. Run any example
export CAPZY_KEY="capzy_..."
bash examples/curl/basic.sh
```

Minimal Python:

```python
import requests, time

KEY = "capzy_xxxxxxxxxxxxxxxxxxxxxxxx"

# 1) Create the task
created = requests.post("https://api.capzy.ai/createTask", json={
    "clientKey": KEY,
    "task": {
        "type": "AntiTurnstileTaskProxyLess",
        "websiteURL": "https://example.com/login",
        "websiteKey": "0x4AAAAAAADnPIDROrmt1Wwj"
    },
}).json()
task_id = created["taskId"]

# 2) Poll until ready
while True:
    result = requests.post("https://api.capzy.ai/getTaskResult", json={
        "clientKey": KEY, "taskId": task_id,
    }).json()
    if result["status"] == "ready":
        break
    time.sleep(2)

print(result["solution"])
```

That's the whole protocol. The rest of this repo is just that, in every
language we could think of.

## Pick your language

| Language        | Example                                       |
|-----------------|-----------------------------------------------|
| **curl / bash** | [`examples/curl/basic.sh`](examples/curl/basic.sh)    |
| **Python**      | [`examples/python/basic.py`](examples/python/basic.py) |
| **Node.js**     | [`examples/nodejs/basic.js`](examples/nodejs/basic.js) |

See [`examples/README.md`](examples/README.md) for setup details.

## Request envelope

```json
{
  "clientKey": "capzy_xxxxxxxxxxxxxxxxxxxxxxxx",
  "task": {
    "type": "AntiTurnstileTaskProxyLess",
    "websiteURL": "https://example.com/login",
    "websiteKey": "0x4AAAAAAADnPIDROrmt1Wwj"
  }
}
```

| Field | Type | Required | Notes |
|-------|------|:--------:|-------|
| `type` | `string` | yes | AntiTurnstileTaskProxyLess or AntiTurnstileTask |
| `websiteURL` | `string` | yes | Full URL of the page where Turnstile loads |
| `websiteKey` | `string` | yes | The Turnstile sitekey (starts with 0x). Find it in the data-sitekey attribute. |
| `proxyType` | `string` | no  | http | https | socks4 | socks5 (only for `AntiTurnstileTask`) |
| `proxyAddress` | `string` | no  | IP or hostname of your proxy (only for `AntiTurnstileTask`) |
| `proxyPort` | `integer` | no  | Port number of your proxy (only for `AntiTurnstileTask`) |
| `proxyLogin` | `string` | no  | Optional — omit if your proxy doesn't require auth (only for `AntiTurnstileTask`) |
| `proxyPassword` | `string` | no  | Optional — omit if your proxy doesn't require auth (only for `AntiTurnstileTask`) |

Full reference in [`docs/parameters.md`](docs/parameters.md).

## Response shape

When the task is ready (`status: "ready"`), `solution` contains:

| Field | Type | Notes |
|-------|------|-------|
| `token` | `string` | The Turnstile token. Submit as the 'cf-turnstile-response' field on the target site. |

### How to use the result

Set the value as the `cf-turnstile-response` form field, or the JSON body field your site's API expects. Tokens expire after 300 seconds — submit promptly.

## Features

- ~3 second average solve time
- 99%+ success rate across all Turnstile modes
- Supports managed, non-interactive, and interactive modes
- Drop-in compatible with Selenium / Puppeteer / Playwright / raw HTTP clients

## FAQ

**How do I find the sitekey?** Inspect the page for a `cf-turnstile` div with `data-sitekey`, or search the JS source for strings starting with `0x`.

**How long is the token valid?** 300 seconds from issue. Submit promptly.

**Full-page Cloudflare 'Checking your browser' interstitial — same endpoint?** No — those use a different task type (AntiCloudflareTask) that returns `cf_clearance` cookies, not a Turnstile token.

## What you'll need

- A Capzy API key — [sign up](https://capzy.ai/auth/register) (free, $0.10 credit).
- Network access to `https://api.capzy.ai`.

## Other captcha types

Capzy solves 25+ captcha types. Full catalog at
[capzy.ai/pricing](https://capzy.ai/pricing). Each type has its own
solver repo on [github.com/capzy-ai](https://github.com/capzy-ai).

## License

[MIT](LICENSE).

---

<div align="center">

**[Sign up for free credits →](https://capzy.ai/auth/register)**

Built by [Capzy](https://capzy.ai). Issues + PRs welcome.

</div>
