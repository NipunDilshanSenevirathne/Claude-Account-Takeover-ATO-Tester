# 🔐 ATO Security Tester — Claude Skill

A Claude skill for automated Account Takeover (ATO), IDOR, Broken Access Control (BAC), and logic vulnerability testing using two test accounts. Load the skill, hand Claude two authenticated sessions, and receive a full security audit report with confirmed findings and remediation guidance.

---

## Table of Contents

1. [Overview](#overview)
2. [What It Tests](#what-it-tests)
3. [How It Works](#how-it-works)
4. [Installation](#installation)
5. [Usage](#usage)
6. [Skill File Structure](#skill-file-structure)
7. [Audit Phases](#audit-phases)
8. [Report Output](#report-output)
9. [Safety Rules](#safety-rules)
10. [Requirements](#requirements)
11. [Contributing](#contributing)
12. [License](#license)

---

## Overview

Account takeover is one of the most damaging vulnerability classes in web applications. This skill gives Claude a complete, structured methodology to:

- Collect two authenticated test sessions interactively
- Discover all API endpoints in the target application
- Run 30+ specific test cases across four vulnerability categories
- Confirm every finding before reporting it
- Generate a CISO-ready remediation report

No external scanner needed — just Claude, two test accounts, and a browser's DevTools.

---

## What It Tests

### Account Takeover (ATO)
- Password reset token reuse and cross-account abuse
- Token entropy and expiry enforcement
- Email change without re-authentication
- Session fixation after login
- Logout invalidation (server-side session destruction)
- "Log out all devices" effectiveness
- OAuth/SSO state parameter and redirect URI manipulation
- Account pre-hijacking via email conflicts
- Account enumeration via login and password reset endpoints
- Brute force and rate limiting on authentication endpoints
- JWT vulnerabilities: `alg:none`, RS256→HS256 confusion, `kid` header injection
- Two-factor authentication bypass

### Insecure Direct Object Reference (IDOR)
- Horizontal read access — Account B reading Account A's objects
- Horizontal write access — Account B mutating Account A's objects
- ID enumeration (sequential integers, predictable UUIDs)
- Indirect IDORs via filter parameters, export endpoints, and webhooks
- Second-order IDORs (deferred data leakage)
- GraphQL node-based access control

### Broken Access Control (BAC)
- Forced browsing to admin and privileged paths
- Function-level access control — calling admin API functions as a regular user
- Role and permission parameter injection via request body
- HTTP method override (`X-HTTP-Method-Override`, `_method`)
- Mass assignment of undocumented fields (balance, credits, roles)
- CORS misconfiguration allowing cross-origin credentialed requests
- Tenant isolation failures in multi-tenant SaaS

### Logic Vulnerabilities
- Race conditions on one-time resources (coupons, free trials, credits)
- Negative and boundary value inputs (negative quantity, zero-amount payment, integer overflow)
- Workflow step skipping (jumping to checkout confirmation without payment)
- State machine abuse (using cancelled subscriptions, recovering deleted accounts)
- Client-supplied price and discount manipulation
- Self-referral abuse
- Cross-account data leakage after collaborator removal

---

## How It Works

The skill uses a **dual-session model**:

1. **Account A** acts as the victim — owns data and resources
2. **Account B** acts as the attacker — attempts to access or modify Account A's data

Claude collects both sessions interactively at the start of the audit, verifies they are live, then works through the full test matrix systematically. Every candidate finding is confirmed by re-running the proof-of-concept request at least twice and verifying the actual impact before being added to the report.

---

## Installation

### Step 1 — Download the skill file

Download `ato-security-tester.skill` from this repository.

### Step 2 — Install in Claude.ai

1. Open [claude.ai](https://claude.ai) and sign in.
2. Go to **Settings** → **Skills**.
3. Click **Upload skill** and select `ato-security-tester.skill`.
4. The skill will appear as **ato-security-tester** in your skills list.

### Step 3 — Verify installation

Start a new conversation and type:

```
I want to run an account takeover security audit on our app.
```

Claude should immediately begin Phase 0 — asking for your base URL and session details.

---

## Usage

### Starting an audit

Use any of the following prompts to trigger the skill:

- `"Run an ATO and IDOR audit on https://app.example.com"`
- `"Find account takeover vulnerabilities in our web app — I have two test accounts"`
- `"Test our app for broken access control and IDOR"`
- `"Run a full dual-account security audit"`

### What Claude will ask you

1. **Base URL and API prefix** of your application
2. **Authentication mechanism** — cookie, Bearer token, or both
3. **Account 1 session** — cookie/token from browser DevTools + user ID, email, role
4. **Account 2 session** — same details for a second account
5. **Any paths to exclude** from testing (optional)
6. **HAR file** (optional, greatly speeds up endpoint discovery)

### How to copy your session from DevTools

1. Open your browser and log in to the application.
2. Press **F12** to open DevTools and go to the **Network** tab.
3. Click any API request in the list.
4. Under **Request Headers**, copy the full `Cookie:` value (or `Authorization:` for Bearer tokens).
5. Paste it into the conversation when Claude asks.

---

## Skill File Structure

```
ato-security-tester/
├── SKILL.md                        # Main skill — phases, procedures, confirmation protocol
└── references/
    ├── discovery.md                # Endpoint discovery methodology (HAR, JS mining, probing)
    ├── idor.md                     # IDOR test cases, ID type recognition, advanced patterns
    ├── bac.md                      # BAC test cases — admin paths, mass assignment, CORS
    ├── ato.md                      # ATO test cases — reset flow, JWT, brute force, OAuth
    ├── logic.md                    # Logic bug test cases — race conditions, state machines
    └── report-template.md          # Full report template used in Phase 7
```

Claude loads reference files on demand during the relevant phase — only what is needed is read, keeping context usage efficient.

---

## Audit Phases

### Phase 0 — Scope & Session Collection
Collects base URL, auth mechanism, Account A session, Account B session, and verifies both sessions return `200` before any testing begins.

### Phase 1 — Endpoint Discovery
Analyses a HAR export, JavaScript bundle, or probes common API paths to build a complete endpoint test matrix categorised by sensitivity tier.

### Phase 2 — IDOR Testing
Tests every Tier 1 and Tier 2 endpoint with an object ID for horizontal read and write access using Account B's session against Account A's objects. Covers direct, indirect, and second-order IDORs.

### Phase 3 — Broken Access Control Testing
Probes admin paths via forced browsing, attempts role escalation via parameter injection, tests HTTP method overrides, mass assignment, and CORS misconfigurations.

### Phase 4 — Account Takeover Testing
Works through the full ATO kill chain: password reset flow, session fixation, logout invalidation, OAuth/SSO, account enumeration, rate limiting, and JWT attacks.

### Phase 5 — Logic Vulnerability Testing
Tests race conditions with parallel requests, boundary value inputs, workflow step skipping, illegal state transitions, and client-side trust violations.

### Phase 6 — Finding Confirmation
Every candidate finding passes a five-point confirmation checklist:
1. Reproduced at least twice
2. Response compared to expected safe response
3. Impact verified (actual data read, mutation confirmed, privilege confirmed)
4. Endpoint confirmed not to be a test/demo endpoint by design
5. Confirmed with the opposing account where applicable

Severity assigned using a CVSS-inspired four-tier system: Critical, High, Medium, Low.

### Phase 7 — Report Generation
Produces a full Markdown report with executive summary, findings table, per-finding proof-of-concept, impact assessment, remediation code snippets, verification steps, and a prioritised remediation roadmap.

---

## Report Output

The final report includes:

- **Executive Summary** — total findings by severity, highest-risk finding in plain language, and immediate action recommendation
- **Scope & Methodology** — URLs tested, masked account identifiers, techniques used
- **Findings Summary Table** — all findings sortable by severity and category
- **Detailed Findings** — one section per vulnerability with:
  - Exact `curl` proof-of-concept command
  - Actual response showing the vulnerable behaviour
  - Concrete impact statement
  - Language-specific remediation code
  - Verification steps to confirm the fix worked
- **Positive Findings** — security controls that were tested and passed
- **Remediation Roadmap** — findings grouped by urgency (24–48 hours, 1–2 weeks, 1 month) with effort estimates
- **Tested Endpoints Appendix** — full list of every endpoint tested and result

---

## Safety Rules

This skill is designed for **authorised security testing only**. The following rules are enforced throughout every audit:

1. **Authorisation required** — Never run against a production system without explicit written authorisation from the application owner.
2. **Two accounts only** — All object ID testing is scoped to Account A's and Account B's own objects. No mass enumeration of other users' IDs.
3. **Stop on disruption** — If any test causes unexpected data loss or service disruption, Claude stops immediately and notifies you.
4. **No password storage** — Only session tokens and cookies are handled, never plaintext passwords.
5. **Rate-limited requests** — All requests use a 5-second timeout and 500ms spacing unless explicitly doing rate-limit testing.

---

## Requirements

- **Claude.ai** account with Skills support (Pro or Team plan)
- **Two test accounts** on the target application (same or different roles — different roles provide broader coverage)
- **Browser DevTools** access to copy session cookies or tokens
- **Written authorisation** from the application owner before testing

Optional but recommended:

- A **HAR export** of normal application traffic (dramatically speeds up endpoint discovery)
- Account B with an **elevated role** (e.g. admin) to enable vertical privilege escalation tests
- Access to the **OpenAPI/Swagger spec** if one exists

---

## Contributing

Contributions are welcome. To add new test cases or vulnerability categories:

1. Fork this repository.
2. Edit the relevant reference file in `references/` or add a new one.
3. Update `SKILL.md` if new phases or confirmation steps are needed.
4. Update this README if the scope or structure changes.
5. Repackage the skill:
   ```bash
   python3 -m scripts.package_skill ./ato-security-tester ./dist
   ```
6. Open a pull request with a description of what was added and why.

### Ideas for future additions

- GraphQL-specific IDOR and introspection abuse module
- WebSocket authentication testing
- Mobile API certificate pinning bypass guidance
- Automated HAR file parsing script
- CI/CD integration mode (non-interactive, JSON output)

---

## License

MIT License — free to use, modify, and distribute with attribution.

---

## Disclaimer

This tool is provided for **defensive security purposes** — to help security teams and developers find and fix vulnerabilities in systems they own or have explicit permission to test. The authors accept no responsibility for misuse. Always obtain proper written authorisation before conducting security testing.
