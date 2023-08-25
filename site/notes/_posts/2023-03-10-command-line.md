---
title: Command line
noindex: true
---

# Command line

Useful bits.

## Miscellaneous

- `date -Iseconds` creates an ISO 8601 timestamp with seconds; e.g. `pbpaste > "SubscriberNetworkConnectivityTests-Logs/tests-$(date -Iseconds).txt"`
- Base64 to hex: `base64 -d | xxd -ps -c0`

## Git

- You can use `-w` in `git show` etc to ignore whitespace — this really helps with changes that have huge amounts of indentation changes
- To set `origin/HEAD`: `git remote set-head origin -a`
