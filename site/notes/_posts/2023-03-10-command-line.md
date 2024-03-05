---
title: Command line
noindex: true
---

# Command line

Useful bits.

## Miscellaneous

- `date -Iseconds` creates an ISO 8601 timestamp with seconds; e.g. `pbpaste > "SubscriberNetworkConnectivityTests-Logs/tests-$(date -Iseconds).txt"`
- Base64 to hex: `base64 -d | xxd -ps -c0`

## tmux

- `tmux save-buffer foo.txt` saves the tmux "clipboard" to a file

## Git

- You can use `-w` in `git show` etc to ignore whitespace — this really helps with changes that have huge amounts of indentation changes
- To set `origin/HEAD`: `git remote set-head origin -a`
- cherry-pick a whole pull request with `git cherry-pick -m1 0064f74f473d4af3b5f51cfda9b697d2e8cba15f`
