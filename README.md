![pluginpool — ten focused Claude Code plugins](./assets/hero.svg)

# pluginpool

**Ten focused Claude Code plugins for everyday developer productivity.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Plugins: 10](https://img.shields.io/badge/plugins-10-7c3aed.svg)](#the-plugins)
[![Tests: 89 passing](https://img.shields.io/badge/tests-89%20passing-success.svg)](#tests)
[![Python: stdlib only](https://img.shields.io/badge/python-stdlib%20only-blue.svg)](https://docs.python.org/3/library/)
[![Status: stable](https://img.shields.io/badge/status-stable-success.svg)](#)

> Ten tiny, deterministic Python helpers wrapped as Claude Code slash commands. Each one tackles a single real-world productivity pain. No magic. No `pip install`. Hermetic tests. MIT.

---

## What's inside

| # | Plugin | What it does | Tests |
|---:|---|---|---:|
| 1 | [`commit-narrator`](https://github.com/mturac/pluginpool-commit-narrator) | Semantic commit message from `git diff --cached`, including the *why* | 8 |
| 2 | [`pr-storyteller`](https://github.com/mturac/pluginpool-pr-storyteller) | PR title + body + test plan from commits and diff vs base branch | 6 |
| 3 | [`test-gap`](https://github.com/mturac/pluginpool-test-gap) | Lines in your diff lacking test coverage (Cobertura / lcov / coverage.json) | 9 |
| 4 | [`deps-doctor`](https://github.com/mturac/pluginpool-deps-doctor) | Multi-ecosystem dependency audit (npm, pip, cargo, go) — one report | 8 |
| 5 | [`env-lint`](https://github.com/mturac/pluginpool-env-lint) | `.env` vs `.env.example` key parity — never prints values | 7 |
| 6 | [`secret-guard`](https://github.com/mturac/pluginpool-secret-guard) | Pre-commit secret scanner: pattern + entropy, redacted output | 10 |
| 7 | [`standup-gen`](https://github.com/mturac/pluginpool-standup-gen) | Daily standup notes from git activity across one or many repos | 8 |
| 8 | [`todo-harvest`](https://github.com/mturac/pluginpool-todo-harvest) | TODO/FIXME/HACK scan with `git blame` author + age | 9 |
| 9 | [`flaky-detector`](https://github.com/mturac/pluginpool-flaky-detector) | Run a test command N times, report per-test flakiness % | 11 |
| 10 | [`changelog-forge`](https://github.com/mturac/pluginpool-changelog-forge) | Conventional commits → CHANGELOG section + semver bump suggestion | 13 |

**89 hermetic tests across the suite. All green.**

---

## Why this exists

Most "developer productivity" tooling sells you on a magic black box and asks you to trust the output. That's the wrong shape for tools you run dozens of times a day. These plugins are the opposite:

- **Deterministic helpers.** Each plugin's core is a small Python script that does exactly one thing and emits structured output you can audit.
- **Claude Code is the runtime, not the brain.** The slash command makes the helper easy to invoke; Claude reasons over the output. You can always run the helper without Claude and get the same data.
- **No `pip install` to use a plugin.** Python 3 standard library only at runtime.
- **Hermetic tests.** Every test that touches git builds its own temp repo with explicit identity. No network. No leakage.
- **Each plugin is its own repo.** Install one. Install all. Vendor what you want.

---

## Installing one plugin

```sh
git clone https://github.com/mturac/pluginpool-<plugin-name> ~/.claude/plugins/<plugin-name>
```

Restart Claude Code; the slash command `/<plugin-name>` appears under `/help`.

## Installing them all

```sh
for p in commit-narrator pr-storyteller test-gap deps-doctor env-lint \
         secret-guard standup-gen todo-harvest flaky-detector changelog-forge; do
  git clone "https://github.com/mturac/pluginpool-$p" "$HOME/.claude/plugins/$p"
done
```

## Using a helper standalone (no Claude Code required)

Every helper has a plain CLI. For example:

```sh
cd ~/.claude/plugins/commit-narrator
git -C ~/your/repo diff --staged | python3 scripts/narrate.py --diff - --format json
```

This is how the test suites exercise the helpers — and how you can wire any of them into CI.

---

## Anatomy of a plugin

```
<plugin>/
├── .claude-plugin/
│   └── plugin.json              # name, version, description, author
├── commands/
│   └── <plugin>.md              # slash command (YAML frontmatter + body)
├── scripts/
│   └── <helper>.py              # Python 3 stdlib-only CLI
├── tests/
│   ├── test_<helper>.py         # hermetic pytest suite
│   └── fixtures/                # synthetic fixtures (where applicable)
├── examples/
│   └── README.md                # copy-pasteable input → output simulations
├── Makefile                     # `make test`
├── LICENSE                      # MIT
├── .gitignore
└── README.md
```

Everything is intentionally small. The biggest helper (`flaky.py`) is ~240 lines. The smallest (`envlint.py`) is ~70.

---

## Tests

Per-plugin:

```sh
cd <plugin>
make test
# or
python3 -m pytest tests/ -v
```

Full sweep across all 10 plugins (run from a directory that contains all of them as siblings):

```sh
for p in commit-narrator pr-storyteller test-gap deps-doctor env-lint \
         secret-guard standup-gen todo-harvest flaky-detector changelog-forge; do
  (cd "$p" && python3 -m pytest tests/ -q --no-header)
done
```

You should see ten lines like `8 passed in 1.45s` summing to 89.

---

## Quality

Each plugin had a multi-pass code review with regression tests for every issue found. Notable invariants enforced by the suite:

- **No `shell=True`, no `eval`, no `exec`** in any helper (`grep -r` clean).
- **env-lint** writes a fake value `SUPERSECRETVALUE123` into a temp `.env` during its tests and asserts that string never appears in JSON or markdown output. Redaction is a tested property, not a comment.
- **secret-guard** asserts that every finding's raw matched string is absent from output — the redaction (`rule + first 4 chars + …`) is verified per pattern.
- **test-gap** explicitly distinguishes "no coverage report present" from "100% covered" — a missing report yields a non-zero exit and a clear warning, never a silent green.
- **deps-doctor** treats missing audit tools as `"skipped: tool not installed"` — never silently green.
- **flaky-detector** parses both `pytest -v` per-test lines and `pytest -q` tail-summary lines, and exits non-zero with a warning if neither produced parseable output.

---

## License

Each plugin ships under MIT. See the `LICENSE` file inside each plugin directory.

---

## Writing

**LinkedIn**
- 🗡️ [Çift Yüzlü Katana: Yapay Zeka Dönüşümlerinin Gerçekçi Bir Analizi](https://www.linkedin.com/pulse/%C3%A7ift-y%C3%BCzl%C3%BC-katana-yapay-zeka-d%C3%B6n%C3%BC%C5%9F%C3%BCmlerinin-ger%C3%A7ek%C3%A7i-bir-mehmet-turac-80h7f) — AI transformations realistic analysis. The 5 illusions that compound into expensive, fragile systems.
- 📄 [LinkedIn Articles](https://www.linkedin.com/in/mehmetturac/recent-activity/articles/) — All published articles
- 📊 [LinkedIn Documents](https://www.linkedin.com/in/mehmetturac/recent-activity/documents/) — Research papers and technical documents

**Dev.to**
- [Why AI Agents Fail?](https://dev.to/turacthethinker/why-ai-agents-fail-ddg)
- [We Ship to Production Without Tests. Here's How It Destroyed Us.](https://dev.to/turacthethinker/we-ship-to-production-without-tests-heres-how-it-destroyed-us-i4i)
- [I built a product in one AI session. Here's the system that made it ship right.](https://dev.to/turacthethinker/i-built-a-product-in-one-ai-session-heres-the-system-that-made-it-ship-right-3mb3)
- [Remote Work Didn't Break Productivity — It Broke Human Connection](https://dev.to/turacthethinker/remote-work-didnt-break-productivity-it-broke-human-connection-288o)
- [Hermes vs OpenClaw: Which AI assistant would you actually trust?](https://dev.to/turacthethinker/hermes-vs-openclaw-which-ai-assistant-would-you-actually-trust-bbl)
- [Strategic LLM Adoption: A Director's Guide to Fine-Tuning Models](https://dev.to/turacthethinker/strategic-llm-adoption-a-directors-guide-to-fine-tuning-models-for-domain-specific-applications-4e37)
- [The Context Window Lie: Why Your LLM Remembers Nothing](https://dev.to/turacthethinker/the-context-window-lie-why-your-llm-remembers-nothing-5h1p)
- [Stop Your AI Agent From Building Tools That Already Exist](https://dev.to/turacthethinker/stop-your-ai-agent-from-building-tools-that-already-exist-6o9)
- [Why Versioned SQL Beats Vector RAG for Agent Memory Systems](https://dev.to/turacthethinker/why-versioned-sql-beats-vector-rag-for-agent-memory-systems-1jo3)
- [I Got Access to 136 AI Models for Free — NVIDIA NIM API Deep Dive](https://dev.to/turacthethinker/i-got-access-to-136-ai-models-for-free-nvidia-nim-api-deep-dive-111o)
- [Your Agent Isn't Reflecting. It's Performing Reflection.](https://dev.to/turacthethinker/your-agent-isnt-reflecting-its-performing-reflection-b41)
- [How I Stopped My AI Agent From Reinventing the Wheel](https://dev.to/turacthethinker/how-i-stopped-my-ai-agent-from-reinventing-the-wheel-24eo)

---

## Repo index

- https://github.com/mturac/pluginpool-commit-narrator
- https://github.com/mturac/pluginpool-pr-storyteller
- https://github.com/mturac/pluginpool-test-gap
- https://github.com/mturac/pluginpool-deps-doctor
- https://github.com/mturac/pluginpool-env-lint
- https://github.com/mturac/pluginpool-secret-guard
- https://github.com/mturac/pluginpool-standup-gen
- https://github.com/mturac/pluginpool-todo-harvest
- https://github.com/mturac/pluginpool-flaky-detector
- https://github.com/mturac/pluginpool-changelog-forge
