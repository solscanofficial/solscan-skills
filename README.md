# Solscan Skills

[Agent Skills](https://agentskills.io/) that teach AI coding assistants (Claude Code and others) how to query Solana blockchain data through Solscan tooling, instead of scraping solscan.io or guessing at the API.

---

## Available Skills

| Skill | Description |
|-------|-------------|
| [`solscan-cli`](skills/solscan-cli/SKILL.md) | Drives the [`solscan` CLI](https://github.com/solscanofficial/solscan-cli) (Solscan Pro API v2.0) — accounts, tokens, transactions, NFTs, blocks, markets, programs, network-wide analytics, and API monitoring. Covers command shape, pagination/time-filter conventions, an action index, common workflows, and error handling, with full per-resource parameter tables in [`skills/solscan-cli/references/`](skills/solscan-cli/references/). |

---

## Prerequisites

The `solscan-cli` skill calls the `solscan` binary, so install the CLI first and configure an API key:

```bash
npm install -g solscan-cli
solscan config set-api-key <YOUR_API_KEY>
```

Get a Solscan Pro API key at [Solscan User Profile's Api Management](https://solscan.io/user/profile#api_management). See the [solscan-cli README](https://github.com/solscanofficial/solscan-cli#readme) for details.

---

## Installation

### With the `skills` CLI

The [`skills` CLI](https://skills.sh/) links a skill into your agent's skill directory:

```bash
# Directly from GitHub
npx skills add solscanofficial/solscan-skills --skill "solscan-cli"

# Or from a local clone
git clone https://github.com/solscanofficial/solscan-skills.git
cd solscan-skills
npx skills add . --skill "solscan-cli"
```

Add `-g` to either command to install the skill globally instead of per-project.

### Manually (Claude Code)

Copy or symlink the skill folder into your Claude Code skills directory:

```bash
git clone https://github.com/solscanofficial/solscan-skills.git
mkdir -p ~/.claude/skills
ln -s "$(pwd)/solscan-skills/skills/solscan-cli" ~/.claude/skills/solscan-cli
```

If your AI agent tries to scrape solscan.io or call the Pro API directly instead of using the CLI, tell it: "Use the `solscan` CLI for all Solscan/Solana data lookups."

---

## Repository Structure

```
solscan-skills/
├── skills/
│   └── solscan-cli/
│       ├── SKILL.md              # Agent-facing usage guide
│       └── references/           # Full per-resource parameter reference
│           ├── <resource>.md     # account, token, transaction, nft, block, market, program, monitor, network
│           └── responses/        # Response-shape docs per action
├── LICENSE
└── README.md
```

---

## Contributing

Skills must stay in sync with the actual CLI behavior. When a command is added or renamed in [solscan-cli](https://github.com/solscanofficial/solscan-cli) (`src/commands/*.js`), update the matching `skills/solscan-cli/references/<resource>.md` file (and the action index in `SKILL.md`) here.

---

## License

MIT
