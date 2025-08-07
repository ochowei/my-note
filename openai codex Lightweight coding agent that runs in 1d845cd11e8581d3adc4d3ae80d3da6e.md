# openai/codex: Lightweight coding agent that runs in your terminal

建立時間: April 17, 2025 3:18 PM
URL: https://github.com/openai/codex
狀態: Wait

# OpenAI Codex CLI

Lightweight coding agent that runs in your terminal

`npm i -g @openai/codex`

![](https://github.com/openai/codex/raw/main/.github/demo.gif)

- **Table of Contents**
    1. [Quickstart](https://github.com/openai/codex#quickstart)
    2. [Why Codex?](https://github.com/openai/codex#why-codex)
    3. [Features](https://github.com/openai/codex#features)
    4. [System Requirements](https://github.com/openai/codex#system-requirements)
    5. [Security Model & Permissions](https://github.com/openai/codex#security-model--permissions)
    6. [CLI Reference](https://github.com/openai/codex#cli-reference)
    7. [Memory & Project Docs](https://github.com/openai/codex#memory--project-docs)
    8. [Non‑interactive / CI mode](https://github.com/openai/codex#non%E2%80%91interactive--ci-mode)
    9. [Recipes](https://github.com/openai/codex#recipes)
    10. [Installation](https://github.com/openai/codex#installation)
    11. [FAQ](https://github.com/openai/codex#faq)
    12. [Contributing](https://github.com/openai/codex#contributing)
    13. [Security & Responsible AI](https://github.com/openai/codex#security--responsible-ai)
    14. [License](https://github.com/openai/codex#license)

## Quickstart

Install globally:

```
npm install -g @openai/codex
```

Run interactively:

```
codex
```

Or, run with a prompt as input (and optionally in `Full Auto` mode):

```
codex "explain this codebase to me"
```

```
codex --approval-mode full-auto "create the fanciest todo-list app"
```

That’s it – Codex will scaffold a file, run it inside a sandbox, install any missing dependencies, and show you the live result. Approve the changes and they’ll be committed to your working directory.

## Why Codex?

Codex CLI is built for developers who already **live in the terminal** and want ChatGPT‑level reasoning **plus** the power to actually run code, manipulate files, and iterate – all under version control. In short, it’s *chat‑driven development* that understands and executes your repo.

- **Zero setup** — bring your OpenAI API key and it just works!
- **Full auto-approval, while safe + secure** by running network-disabled and directory-sandboxed
- **Multimodal** — pass in screenshots or diagrams to implement features ✨

And it's **fully open-source** so you can see and contribute to how it develops!

## Security Model & Permissions

Codex lets you decide *how much autonomy* the agent receives and auto-approval policy via the `--approval-mode` flag (or the interactive onboarding prompt):

| Mode | What the agent may do without asking | Still requires approval |
| --- | --- | --- |
| **Suggest** (default) | • Read any file in the repo | • **All** file writes/patches • **All** shell/Bash commands |
| **Auto Edit** | • Read **and** apply‑patch writes to files | • **All** shell/Bash commands |
| **Full Auto** | • Read/write files • Execute shell commands | – |

In **Full Auto** every command is run **network‑disabled** and confined to the current working directory (plus temporary files) for defense‑in‑depth. Codex will also show a warning/confirmation if you start in **auto‑edit** or **full‑auto** while the directory is *not* tracked by Git, so you always have a safety net.

Coming soon: you’ll be able to whitelist specific commands to auto‑execute with the network enabled, once we’re confident in additional safeguards.

### Platform sandboxing details

The hardening mechanism Codex uses depends on your OS:

- 
    
    **macOS 12+** – commands are wrapped with **Apple Seatbelt** (`sandbox-exec`).
    
    - Everything is placed in a read‑only jail except for a small set of writable roots (`$PWD`, `$TMPDIR`, `~/.codex`, etc.).
    - Outbound network is *fully blocked* by default – even if a child process tries to `curl` somewhere it will fail.
- 
    
    **Linux** – we recommend using Docker for sandboxing, where Codex launches itself inside a **minimal container image** and mounts your repo *read/write* at the same path. A custom `iptables`/`ipset` firewall script denies all egress except the OpenAI API. This gives you deterministic, reproducible runs without needing root on the host. You can read more in [`run_in_container.sh`](https://github.com/openai/codex/blob/main/codex-cli/scripts/run_in_container.sh)
    

Both approaches are *transparent* to everyday usage – you still run `codex` from your repo root and approve/reject steps as usual.

## System Requirements

| Requirement | Details |
| --- | --- |
| Operating systems | macOS 12+, Ubuntu 20.04+/Debian 10+, or Windows 11 **via WSL2** |
| Node.js | **22 newer** (LTS recommended) |
| Git (optional, recommended) | 2.23+ for built‑in PR helpers |
| ripgrep (optional) | `rg` accelerates large‑repo search |
| RAM | 4‑GB minimum (8‑GB recommended) |

> 
> 
> 
> Never run `sudo npm install -g`; fix npm permissions instead.
> 

## CLI Reference

| Command | Purpose | Example |
| --- | --- | --- |
| `codex` | Interactive REPL | `codex` |
| `codex "…"` | Initial prompt for interactive REPL | `codex "fix lint errors"` |
| `codex -q "…"` | Non‑interactive "quiet mode" | `codex -p --json "explain utils.ts"` |

Key flags: `--model/-m`, `--approval-mode/-a`, and `--quiet/-q`.

## Memory & Project Docs

Codex merges Markdown instructions in this order:

1. `~/.codex/instructions.md` – personal global guidance
2. `codex.md` at repo root – shared project notes
3. `codex.md` in cwd – sub‑package specifics

Disable with `--no-project-doc` or `CODEX_DISABLE_PROJECT_DOC=1`.

## Non‑interactive / CI mode

Run Codex head‑less in pipelines. Example GitHub Action step:

```
- name: Update changelog via Codex
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "update CHANGELOG for next release"
```

Set `CODEX_QUIET_MODE=1` to silence interactive UI noise.

## Recipes

Below are a few bite‑size examples you can copy‑paste. Replace the text in quotes with your own task.

| ✨ | What you type | What happens |
| --- | --- | --- |
| 1 | `codex "Refactor the Dashboard component to React Hooks"` | Codex rewrites the class component, runs `npm test`, and shows the diff. |
| 2 | `codex "Generate SQL migrations for adding a users table"` | Infers your ORM, creates migration files, and runs them in a sandboxed DB. |
| 3 | `codex "Write unit tests for utils/date.ts"` | Generates tests, executes them, and iterates until they pass. |
| 4 | `codex "Bulk‑rename *.jpeg → *.jpg with git mv"` | Safely renames files and updates imports/usages. |
| 5 | `codex "Explain what this regex does: ^(?=.*[A-Z]).{8,}$"` | Outputs a step‑by‑step human explanation. |

## Installation

- **From npm (Recommended)**
    
    ```
    npm install -g @openai/codex
    # or
    yarn global add @openai/codex
    ```
    
- **Build from source**
    
    ```
    # Clone the repository and navigate to the CLI package
    git clone https://github.com/openai/codex.git
    cd codex/codex-cli
    
    # Install dependencies and build
    npm install
    npm run build
    
    # Run the locally‑built CLI directly
    node ./dist/cli.js --help
    
    # Or link the command globally for convenience
    npm link
    ```
    

## Configuration

Codex looks for config files in **`~/.codex/`**.

```
# ~/.codex/config.yaml
model: o4-mini # Default model
fullAutoErrorMode: ask-user # or ignore-and-continue
```

You can also define custom instructions:

```
# ~/.codex/instructions.md
- Always respond with emojis
- Only use git commands if I explicitly mention you should
```

## FAQ

- How do I stop Codex from touching my repo?
    
    Codex always runs in a **sandbox first**. If a proposed command or file change looks suspicious you can simply answer **n** when prompted and nothing happens to your working tree.
    
- Does it work on Windows?
    
    Not directly, it requires [Linux on Windows (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) – Codex is tested on macOS and Linux with Node ≥ 22.
    
- Which models are supported?
    
    Any model available with [Responses API](https://platform.openai.com/docs/api-reference/responses). The default is `o3`, but pass `--model gpt-4o` or set `model: gpt-4o` in your config file to override.
    

## Contributing

This project is under active development and the code will likely change pretty siginificantly. We'll update this message once that's complete!

More broadly We welcome contributions – whether you are opening your very first pull request or you’re a seasoned maintainer. At the same time we care about reliability and long‑term maintainability, so the bar for merging code is intentionally **high**. The guidelines below spell out what “high‑quality” means in practice and should make the whole process transparent and friendly.

### Development workflow

- Create a *topic branch* from `main` – e.g. `feat/interactive-prompt`.
- Keep your changes focused. Multiple unrelated fixes should be opened as separate PRs.
- Use `npm run test:watch` during development for super‑fast feedback.
- We use **Vitest** for unit tests, **ESLint** + **Prettier** for style, and **TypeScript** for type‑checking.
- Make sure all your commits are signed off with `git commit -s ...`, see [Developer Certificate of Origin (DCO)](https://github.com/openai/codex#developer-certificate-of-origin-dco) for more details.

```
# Watch mode (tests rerun on change)
npm run test:watch

# Type‑check without emitting files
npm run typecheck

# Automatically fix lint + prettier issues
npm run lint:fix
npm run format:fix
```

### Writing high‑impact code changes

1. **Start with an issue.** Open a new one or comment on an existing discussion so we can agree on the solution before code is written.
2. **Add or update tests.** Every new feature or bug‑fix should come with test coverage that fails before your change and passes afterwards. 100 % coverage is not required, but aim for meaningful assertions.
3. **Document behaviour.** If your change affects user‑facing behaviour, update the README, inline help (`codex --help`), or relevant example projects.
4. **Keep commits atomic.** Each commit should compile and the tests should pass. This makes reviews and potential rollbacks easier.

### Opening a pull request

- Fill in the PR template (or include similar information) – **What? Why? How?**
- Run **all** checks locally (`npm test && npm run lint && npm run typecheck`). CI failures that could have been caught locally slow down the process.
- Make sure your branch is up‑to‑date with `main` and that you have resolved merge conflicts.
- Mark the PR as **Ready for review** only when you believe it is in a merge‑able state.

### Review process

1. One maintainer will be assigned as a primary reviewer.
2. We may ask for changes – please do not take this personally. We value the work, we just also value consistency and long‑term maintainability.
3. When there is consensus that the PR meets the bar, a maintainer will squash‑and‑merge.

### Triaging labels

- `good first issue` – great for newcomers, usually well‑scoped and low risk.
- `help wanted` – higher impact, still looking for outside contributors.
- `discussion` – exploring the problem/solution space; code contributions are discouraged until the direction is clear.

### Community values

- **Be kind and inclusive.** Treat others with respect; we follow the [Contributor Covenant](https://www.contributor-covenant.org/).
- **Assume good intent.** Written communication is hard – err on the side of generosity.
- **Teach & learn.** If you spot something confusing, open an issue or PR with improvements.

### Getting help

If you run into problems setting up the project, would like feedback on an idea, or just want to say *hi* – please open a Discussion or jump into the relevant issue. We are happy to help.

Together we can make Codex CLI an incredible tool. **Happy hacking!** 🚀

### Developer Certificate of Origin (DCO)

All commits **must** include a `Signed‑off‑by:` footer.

This one‑line self‑certification tells us you wrote the code and can contribute it under the repo’s license.

### How to sign (recommended flow)

```
# squash your work into ONE signed commit
git reset --soft origin/main          # stage all changes
git commit -s -m "Your concise message"
git push --force-with-lease           # updates the PR
```

> 
> 
> 
> We enforce **squash‑and‑merge only**, so a single signed commit is enough for the whole PR.
> 

### Quick fixes

| Scenario | Command |
| --- | --- |
| Amend last commit | `git commit --amend -s --no-edit && git push -f` |
| GitHub UI only | Edit the commit message in the PR → add`Signed-off-by: Your Name <email@example.com>` |

The **DCO check** blocks merges until every commit in the PR carries the footer (with squash this is just the one).

## Security & Responsible AI

Have you discovered a vulnerability or have concerns about model output? Please e‑mail [**security@openai.com**](mailto:security@openai.com) and we will respond promptly.

## License

This repository is licensed under the [Apache-2.0 License](https://github.com/openai/codex/blob/main/LICENSE).