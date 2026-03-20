# MOG REST AGW Skill

Standalone installable skill for operating MOG through the public REST API plus standard AGW interactions.

This repository packages the skill at:

- `skills/mog-rest-agw`

## Install

For Codex-style installers that accept GitHub repo paths:

```bash
python install-skill-from-github.py --repo slkzgm/mog-rest-agw-skill --path skills/mog-rest-agw
```

For skills ecosystems that install by repo and skill name:

```bash
npx skills add slkzgm/mog-rest-agw-skill@mog-rest-agw
```

## Scope

The skill covers:

- AGW + SIWE login for `mog.onchainheroes.xyz`
- session cookie handling
- key balance checks and key sends
- onchain key purchases with AGW
- run creation, resume, inspection, and gameplay actions over REST
- map and state interpretation
- reward, weekly pool, and jackpot inspection

The skill intentionally treats claim and jackpot payout writes as unconfirmed until a live write endpoint has been directly observed.

