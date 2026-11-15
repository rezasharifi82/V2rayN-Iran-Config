# Troubleshooting

## `illegal domain rule: geosite:ir ... EOF` / core fails to start

**Cause:** the routing references a geosite tag not present in your `geosite.dat`.

**Fix:** replace `geosite:ir` with the tag that exists in your data (commonly **`geosite:category-ir`**). Update the active ruleset in v2rayN, reconnect, and validate:

```bash
cd <v2rayN>/bin/xray
XRAY_LOCATION_ASSET=<v2rayN>/bin ./xray -test -config <v2rayN>/binConfigs/config.json
```

## Won’t connect / no traffic

- **Reload/reconnect** after any change (cores don’t hot-reload).
- Ensure **proxy server domain/IP is `direct` at the top** of rules (before Iran/catch-all). This is required even if the server is in Iran.
- Confirm the active ruleset is the intended one (not `V-Blacklist`).
- Check generated config: `binConfigs/config.json` (Xray) or `binConfigs/configPre.json` (sing-box/TUN).
- If using **TUN (sing-box)**, verify sing-box has the needed rule-sets; without Iran `.srs` rule-sets, Iran-bypass may not behave like Xray geoip/geosite. Try **TUN off** to compare.

## Non-Iran site goes direct (should be proxy)

- With a correct “Iran direct / others proxy” ruleset, non-Iran should hit the final `0-65535 → proxy`.
- Check for an earlier matching **direct** rule, wrong active ruleset, or stale running config (reconnect).
- Force-proxy a specific domain by adding `domain:...` to a **proxy** rule **above** Iran-direct rules.

## Iran site goes through proxy (should be direct)

- Verify `geosite:category-ir` / `geoip:ir` tags exist in your geo data and geo files are updated.
- Use `domainStrategy: IPIfNonMatch`.
- Check rule order: Iran-direct rules must be above the final proxy catch-all.

## `geoip:ir` seems ignored

- Your `geoip.dat` may not include IR ranges. Update geo data or use a data set that includes Iran. Domain-based `geosite:category-ir` will still help for many Iranian sites.

## Which mode/routes apply?

- **Xray/system-proxy:** `binConfigs/config.json`, `geosite.dat`/`geoip.dat`.
- **TUN (sing-box):** `binConfigs/configPre.json`, `bin/srss/*.srs`. Behavior can differ if equivalent Iran rule-sets are missing.
