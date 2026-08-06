# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is not a software project — it's a personal collection of **Clash/OpenClash proxy configuration files**, published to `github.com/andyliukun/clash` and consumed as raw URLs by [subconverter](https://github.com/tindy2013/subconverter) and by OpenClash routers directly. There is no build system, package manager, linter, or test suite. "Correctness" means the files parse as valid subconverter/Clash syntax and the rule ordering produces the intended routing behavior.

## File formats and layout

- **Root `Clash-*.ini` / `Cash-All.ini`** — subconverter "external config" (ACL4SSR-style) files. These are *not* full Clash YAML configs; they're fed to a subconverter instance as the `config=` parameter to generate a full config from a subscription. Syntax:
  - `ruleset=<group-name>,<url-or-directive>` — maps a rule-provider (a `.list` file, hosted either in this repo's `list/` folder or a third-party repo like `blackmatrix7/ios_rule_script`) to a named rule group. Order matters — rules are evaluated top to bottom, first match wins. Special directives: `[]GEOIP,CN`, `[]FINAL`.
  - `custom_proxy_group=<name>` `` `type` `` `[]<option>` `` ... — defines a proxy group. Type is usually `select` or `url-test`. Entries prefixed `[]` reference another proxy group by name; `.*` means "match all remaining proxies."
  - `enable_rule_generator=true` / `overwrite_original_rules=true` — standard subconverter flags telling it to use these rules instead of the subscription's own.
  - Lines starting with `;` or `#` are comments/disabled entries.
  - Emoji-prefixed group names (✨, 📘, 🎮, 🌍, 🐟, etc.) are cosmetic labels shown in the Clash client UI — keep them when editing rather than renaming groups outright, since proxy-group references (`[]🚀 节点选择` etc.) depend on exact string matches.
- **`proxy/*.ini`** — a second, older/parallel set of subconverter external configs (lowercase filenames), with a much larger proxy-group topology (region-based groups like `所有-手动`, `港台日新韩-自动`, `美国-自动`, etc.) compared to the simpler root-level configs which mostly route everything through a single `🚀 节点选择` group.
- **`list/*.list`** — the actual rule-provider files referenced by `ruleset=` lines above, in Clash classical rule syntax (`DOMAIN`, `DOMAIN-SUFFIX`, `DOMAIN-KEYWORD`, `IP-CIDR`, etc., one per line, `#`-comments allowed). These are pulled via raw GitHub URLs from this repo by both the root and `proxy/` configs, so **editing a `.list` file changes behavior for every config that references it**, including ones fetched live by other machines/routers.
- **`main_router/openclash` and `second_router/openclash`** — full OpenWrt UCI config dumps (`/etc/config/openclash`) backed up from two physical routers. They differ only in a couple of settings (e.g. `append_wan_dns`) — check with `diff` before assuming a change should apply to both.
- **`Clash-Dev.txt`** — a firewall/init custom-script snippet (pasted into OpenClash's "developer options" plugin setting), not a config file itself. Pulls the anti-AD adblock list and reloads dnsmasq on startup.

## Key relationships to keep in mind when editing

- Root configs (e.g. `Clash-Full-new.ini`, `Clash-Full-2026-8-6.ini`) reference `list/*.list` files via **raw.githubusercontent.com URLs pointing at `liandu2024/clash`**, a different GitHub account than this repo's own remote (`andyliukun/clash`) — some rulesets also point at `andyliukun/ACL4SSR`. This repo is one of possibly several mirrors; don't assume the `list/` files here are the live source of truth for every config that references them without checking which account's URL is actually in use.
- `Clash-Full-2026-8-6.ini` and `Clash-Full-new.ini` are currently near-duplicates (dated snapshots) — check both when making a change intended to affect "the full config" and confirm which one is actually deployed before deleting either.
- A `.geekery.cn` GitHub-mirror prefix (`https://github.geekery.cn/raw.githubusercontent.com/...`) shows up in some `proxy/*.ini` files as a China-accessible mirror for fetching rules — don't strip it as if it were a typo.
- There's no automated way to validate these files in this repo. If you need to sanity-check a change, look for internal consistency instead: every `[]<group-name>` reference in a `custom_proxy_group` line should match a group actually defined elsewhere (either another `custom_proxy_group` name or a `ruleset=` group name), and every `ruleset=` group name should appear in some `custom_proxy_group` line so it isn't orphaned.
