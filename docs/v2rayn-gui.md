# v2rayN GUI: Setup & Explicit Domain Rules

## Create the routing ruleset

1. `Settings` → `Routing Setting`.
2. Add/edit a ruleset (example name: `IR-Direct_Others-Proxy`).
3. Set **Domain strategy** to `IPIfNonMatch`.
4. Add rules in this order (first match wins):

| Order | Outbound | Match fields | Example value(s) |
|---|---|---|---|
| 1 | direct | Domain | `full:<YOUR_PROXY_SERVER_DOMAIN>` |
| 2 | direct | IP | `<YOUR_PROXY_SERVER_IP>/32` |
| 3 | direct | IP | `geoip:private` |
| 4 | direct | Domain | `geosite:private` |
| 5 | block | Port + Network | `443`, `udp` (optional) |
| 6 | direct | Protocol | `bittorrent` (optional) |
| 7 | block | Domain | `geosite:category-ads-all` (optional) |
| 8 | direct | Domain | `geosite:category-ir` |
| 9 | direct | IP | `geoip:ir` |
| 10 | proxy | Port | `0-65535` (catch-all) |

5. Enable/select this ruleset for your active profile.
6. **Reconnect/reload** after saving.

> Use `geosite:category-ir` if your `geosite.dat` provides it; avoid `geosite:ir` if that tag is not present in your data (it can prevent the core from starting).

## Explicit domains

### Bypass a domain (DIRECT, no proxy)
- Add to a **direct** rule **above** the final proxy catch-all.
- Formats: `domain:example.ir`, `full:sub.example.ir`, `keyword:example`, `regexp:...`

### Force a domain through proxy
- Add to a **proxy** rule **above** the Iran-direct rules (`geosite:category-ir`, `geoip:ir`) so it overrides “Iran → direct”.
- Same formats apply.

## After changing server/profile

If the proxy server changes, update the **top direct-bypass** rules with the new server domain (`full:...`) and/or IP (`x.x.x.x/32`), then reconnect. Without this, routing can block the tunnel from establishing.

## Check which rule matched (Xray)

Enable/view core access logs. A line like:
- `accepted //opencode.ai:443 [socks -> proxy]` — proxied (expected for non-Iran)
- `accepted //opencode.ai:443 [socks -> direct]` — went direct (check rule order/matches and that the running core reloaded the latest config)
