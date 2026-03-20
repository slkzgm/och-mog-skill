# OCH MOG Skill

Standalone installable skill for operating OCH MOG through wallet signing plus the public REST API.

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

The skill covers:

- EOA or AGW SIWE login for `mog.onchainheroes.xyz`
- session cookie handling
- key balance checks and key sends
- onchain key purchases with a wallet
- run creation, resume, inspection, and gameplay actions over REST
- map and state interpretation
- reward, weekly pool, and jackpot inspection

The skill intentionally treats claim and jackpot payout writes as unconfirmed until a live write endpoint has been directly observed.
