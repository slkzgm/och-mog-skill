# OCH MOG Skill

Standalone installable skill for operating public OCH MOG variants through wallet signing plus the public REST API.

This repository packages the skill at:

- `skills/och-mog`

## Install

For Codex-style installers that accept GitHub repo paths:

```bash
python install-skill-from-github.py --repo slkzgm/och-mog-skill --path skills/och-mog
```

For skills ecosystems that install by repo and skill name:

```bash
npx skills add slkzgm/och-mog-skill@och-mog
```

## Scope

The skill covers only public MOG surfaces:

- SIWE login for public OCH MOG hosts
- session cookie handling
- key balance checks, key sends, and transfer history
- onchain key purchase flow plus REST purchase processing
- run creation, resume, inspection, and gameplay actions over REST
- normal and World run handling where publicly available
- map, fog, state, item, portal, enemy, and upgrade interpretation
- weekly reward and jackpot claim flows

The skill intentionally excludes unannounced variants, private configuration, contract addresses, source snippets, and local environment values.

## Strategy Credits

Strategy guidance and high-level gameplay heuristics were informed in part by Gucci.

- X: https://x.com/gucci_gcc
- GitHub: https://github.com/gcc1996
- Discord: `gcc8500`
