# ATC Simulator

A realistic, text-based air traffic control game for Claude. You are the controller. Claude plays the pilots, vehicles, neighbouring sectors and the weather, gives you a decision each turn and coaches your phraseology.

```
/atc ESSA_TWR winter evening, snow showers, busy with arrivals
```

Swedish airspace is the default, but any ICAO airport or sector works. Positions look like `ESSA_TWR`, `ESGG_APP` or `ESOS_CTR`, and the scenario text is optional.

## Install

### Claude (web, desktop and mobile apps)

1. Download **[atc.zip](https://github.com/sparrisoppa/atc-skill/releases/latest/download/atc.zip)**.
2. In Claude, open **Settings → Capabilities → Skills**, choose **Upload skill** and pick `atc.zip`.
3. In a new chat, type `/atc` or "let's play the ATC game".

If you don't see Skills in settings, check that code execution is turned on under Capabilities. On Team and Enterprise plans an admin may need to allow skills.

### Claude Code

```
/plugin marketplace add sparrisoppa/atc-skill
/plugin install atc@atc-skill
```

Then start a game with `/atc:atc`, or ask Claude to play the ATC game. Run `/plugin marketplace update atc-skill` to get new versions.

## In-game commands

`question <text>` · `pause` · `start` · `restart` · `status` · `harder` / `easier` · `switch <POSITION>` · `debrief`

## Repository layout

```
.claude-plugin/marketplace.json      Claude Code marketplace
plugins/atc/.claude-plugin/plugin.json
plugins/atc/skills/atc/SKILL.md      the skill itself
.github/workflows/release.yml        builds atc.zip for each release
```

To publish a change, edit `SKILL.md`, bump `version` in `plugin.json` and push to `main`. The workflow creates a new release with a fresh `atc.zip`.

Made by [Mathias Kallmert](https://sparrisoppa.github.io/website/).
