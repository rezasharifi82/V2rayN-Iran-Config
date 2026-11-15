# v2rayN + Xray: Iran Direct / Non-Iran Proxy Routing

A complete, tested guide and configuration for **v2rayN** (with **Xray** core) to:

- **Bypass proxy (DIRECT)** for **Iranian traffic** (Iran domains and IP ranges)
- **Use proxy** for **non-Iran traffic**
- Keep the **proxy server itself reachable** (critical when the server/domain/IP is in Iran or can be misclassified)
- **Add explicit domains** to bypass or force-proxy on demand

> **Privacy:** All examples are sanitized. Replace placeholders like `<SERVER_ADDRESS>`, `<UUID>`, etc. with your own values. **Never commit real credentials** (UUIDs, keys, subscription URLs).

## How it works

Xray routing evaluates rules **top-to-bottom** (first match wins). This setup:

1. Forces the **proxy server endpoint** to `direct` (so the tunnel can always be established)
2. Sends **private/LAN** traffic to `direct`
3. Sends **Iran domains/IPs** to `direct`
4. Sends **everything else** to `proxy`

Recommended `domainStrategy`: **`IPIfNonMatch`** (resolve domains when needed so IP-based rules like `geoip:ir` can apply reliably).

## Geo data requirements (important)

Routing depends on your installed geo files (`geosite.dat` / `geoip.dat`):

| Purpose | Tag used in this config | Notes |
|---|---|---|
| Iran domains | `geosite:category-ir` | On many data sets this is the correct tag. Some sets also provide `geosite:ir`; use the tag that exists in **your** `geosite.dat` (using a non-existent tag can break core startup, e.g. `illegal domain rule: geosite:ir ... EOF`). |
| Iran IP ranges | `geoip:ir` | Requires `geoip.dat` to include `IR`. |
| Private/local | `geosite:private`, `geoip:private` | Recommended. |
| Ads (optional) | `geosite:category-ads-all` | Optional block rule. |

**v2rayN:** keep geo files updated via the app’s geo/rule-set update feature. Tag names must match your installed data files.

## Routing rules (Xray)

Order matters — first match wins.

| # | Match | Outbound | Purpose |
|---|---|---|---|
| 1 | `domain: full:<YOUR_PROXY_SERVER_DOMAIN>` | `direct` | Always reach proxy server (domain) |
| 2 | `ip: <YOUR_PROXY_SERVER_IP>/32` | `direct` | Always reach proxy server (IP) |
| 3 | `ip: geoip:private` | `direct` | LAN/private IPs |
| 4 | `domain: geosite:private` | `direct` | Local/private domains |
| 5 | `port: 443, network: udp` | `block` | Block QUIC/UDP-443 (optional, reduces leaks/issues) |
| 6 | `protocol: bittorrent` | `direct` | BitTorrent direct (optional) |
| 7 | `domain: geosite:category-ads-all` | `block` | Ads block (optional) |
| 8 | `domain: geosite:category-ir` | `direct` | Iran domains |
| 9 | `ip: geoip:ir` | `direct` | Iran IPs |
| 10 | `port: 0-65535` | `proxy` | **Catch-all: non-Iran/others via proxy** |

See: [`configs/xray-routing.json`](configs/xray-routing.json) and [`configs/xray-config.template.json`](configs/xray-config.template.json).

## Step-by-step (v2rayN GUI)

1. **Open routing settings:** `Settings` → `Routing Setting`.
2. **Create/select a ruleset** (e.g. `IR-Direct_Others-Proxy`). Do **not** use `V-Blacklist` for this goal (Blacklist defaults non-listed traffic to **direct**, so most non-Iran foreign traffic would bypass the proxy).
3. **Add rules in order** (or import/replicate the rules table above):
   - Rule: Outbound **direct**, Domain `full:<YOUR_PROXY_SERVER_DOMAIN>`
   - Rule: Outbound **direct**, IP `<YOUR_PROXY_SERVER_IP>/32`
   - Rule: Outbound **direct**, IP `geoip:private`
   - Rule: Outbound **direct**, Domain `geosite:private`
   - Rule: Outbound **block**, Port `443`, Network `udp` (optional)
   - Rule: Outbound **direct**, Protocol `bittorrent` (optional)
   - Rule: Outbound **block**, Domain `geosite:category-ads-all` (optional)
   - Rule: Outbound **direct**, Domain `geosite:category-ir` (verify tag in your geodata)
   - Rule: Outbound **direct**, IP `geoip:ir`
   - Rule: Outbound **proxy**, Port `0-65535` (**final catch-all**)
4. **Set domain strategy:** `IPIfNonMatch` for the ruleset/core routing.
5. **Activate/apply** the ruleset to your connection/profile.
6. **Update geo files** in v2rayN so `category-ir` / `geoip:ir` are available and current.
7. **Reconnect/reload** the profile (disconnect/connect or re-apply) after any routing change — cores load config at (re)start.

## Add explicit domains (bypass or force proxy)

Xray domain match formats:
- `domain:example.com` or `example.com` — domain **and subdomains**
- `full:host.example.com` — **exact** host only
- `keyword:example` — contains keyword
- `regexp:...` — regular expression

**Rule order is first-match-wins.**

### Bypass a specific domain (no proxy, DIRECT)
- Add the domain to a **direct** rule placed **above** the final `0-65535 → proxy` catch-all.
- Example: add `domain:mybank.ir` or `full:intranet.example.com` to the Iran/private direct rule (or a dedicated direct rule).

### Force a specific domain through the proxy
- Add the domain to a **proxy** rule placed **above** the Iran-direct rules (rules 8/9), so it matches before “Iran → direct”.
- Example: create a rule (Outbound **proxy**, Domain `domain:example.ir`) and move it above `geosite:category-ir` / `geoip:ir`.

After editing, **save and reconnect/reload** in v2rayN.

## TUN mode vs system-proxy (Xray) mode

- **System-proxy / Xray mode:** uses Xray routing with `geosite.dat`/`geoip.dat` (the rules in this repo target this). Recommended if you have correct `category-ir`/`geoip:ir` data.
- **TUN mode (sing-box):** routing is handled by sing-box and typically uses `.srs` rule-sets (e.g. `bin/srss/*.srs`) and a generated config (e.g. `binConfigs/configPre.json`). If your install does not include Iran rule-sets (no `geoip-ir.srs`/`geosite-ir.srs`), sing-box cannot mirror “Iran direct” the same way. For reliable Iran-bypass with this policy, prefer **Xray/system-proxy mode** or install matching Iran rule-sets for sing-box.

Always **reconnect/reload** after switching modes or routing.

## Verify

1. **Validate Xray config** (from the Xray binary directory so assets resolve, or set asset path):
   ```bash
   cd <v2rayN>/bin/xray
   XRAY_LOCATION_ASSET=<v2rayN>/bin ./xray -test -config <v2rayN>/binConfigs/config.json
   ```
   Expect: `Configuration OK`.
2. **Check egress IP through proxy** (default local mixed/SOCKS port often `10808`):
   ```bash
   curl -x socks5h://127.0.0.1:10808 https://api.ip.sb/ip
   ```
   Should return your **proxy exit IP** (non-Iran) for non-Iran destinations.
3. **Check Iran reachability/bypass:**
   ```bash
   curl -I -x socks5h://127.0.0.1:10808 https://www.irna.ir
   ```
4. **Check access log behavior (Xray):** non-Iran domains should show `[... -> proxy]`, Iran-matched should show `[... -> direct]` (based on your rules/log level).

## Troubleshooting

- **Core fails with `illegal domain rule: geosite:ir ... EOF`**: your `geosite.dat` doesn’t have `ir`; use **`geosite:category-ir`** (or the correct tag present in your data).
- **Nothing connects / tunnel won’t establish**: ensure the **proxy server domain/IP is a top `direct` rule** (before Iran/catch-all). If the server is in Iran or resolves to an IR IP, a missing bypass can break connectivity. Reconnect after changes.
- **Domain goes direct but should be proxied**: a direct rule matched earlier (e.g. mis-tagged Iran list, private list, or wrong order). Move/adjust rules or add an explicit **proxy** rule for that domain above Iran-direct rules.
- **`geoip:ir` not effective**: your `geoip.dat` may lack IR data — update geo files; confirm tag availability for your distribution.
- **TUN behaves differently than expected**: TUN uses sing-box routing/rule-sets, not Xray dat matching. Test with TUN off to isolate.
- **v2rayN overwrote generated config**: edit the **active routing ruleset** in v2rayN (DB/UI source), not only generated files, then reconnect.

## Repository contents

- [`README.md`](README.md) — this guide
- [`configs/xray-routing.json`](configs/xray-routing.json) — routing block (rules + strategy)
- [`configs/xray-config.template.json`](configs/xray-config.template.json) — sanitized full Xray config template (fill placeholders)
- [`docs/v2rayn-gui.md`](docs/v2rayn-gui.md) — GUI-focused checklist and explicit-domain recipes
- [`docs/troubleshooting.md`](docs/troubleshooting.md) — common errors and fixes

## References

- [Xray routing](https://xtls.github.io/config/routing.html)
- [v2rayN](https://github.com/2dust/v2rayN)
- Geo data commonly used with v2rayN/Xray (v2fly/Loyalsoldier-style `geosite.dat`/`geoip.dat`) — tag availability depends on the data set you install
