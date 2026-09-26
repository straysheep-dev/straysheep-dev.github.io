# CLAUDE.md - straysheep-dev.github.io

Scope: this repo is a personal Zensical/mkdocs-material site, not part of the
`packer-configs` IaC mono repo tree. The global `~/src/AGENTS.md` (or
`CLAUDE.md`) policy does not govern this repo's content; this file is
self-contained and only covers drafting resource entries for
`docs/resources.md` and `docs/notes/*`.

The site is public (GitHub Pages). Don't include secrets, private URLs, or
anything not meant for a public audience.

## Tasks

### General Guidance

Always write in plain ASCII and do not overuse dashes. If unicode characters
are *necessary*, share the first draft with them included and mention why
for review.

### For resources.md Entries

Given a **tool name** and **at least one URL** (repo, homepage, or docs page),
draft one admonition block in the same style as existing entries in
`docs/resources.md`, plus a recommendation for where it belongs in the file.

**Output only.** Do not edit `docs/resources.md` unless the user explicitly
asks you to insert the draft.

#### Gathering the Content

- Fetch the given URL(s) if possible. Use the tool's own README/homepage copy
  to write an accurate one-line description and confirm the project is real
  and active.
- If the fetch fails or is blocked, say so plainly and produce the draft with
  a `<!-- description needed -->` placeholder instead of guessing.
- Never invent a "discovered on..." attribution (podcast, newsletter, video,
  SANS diary, etc.). Only include that sentence if the user tells you where
  they found the tool. Otherwise leave it out - don't fabricate a source.
- Never fabricate a quoted blockquote. Only use a `>` quote if you can source
  it verbatim from the tool's own README/site. If nothing quotable is short
  and clean, write a plain-prose description instead of a quote.

#### Block Syntax (Match Existing Entries Exactly)

- Collapsible admonition: `??? <type> "<Title>"`. Reserve non-collapsible
  `!!!` for section-level overview blocks only - never for a single tool
  entry.
- Body is indented **one tab**, one level deep. No extra nesting.
- Blank line after the opener line, and blank lines between paragraphs/lists
  inside the body.
- Title is the tool's own name/capitalization. Only prefix it with an icon
  shortcode (`:material-x:`, `:simple-x:`, `:fontawesome-brands-x:`,
  `:octicons-x-16:`, `:lucide-x:`) if you can confirm the exact slug exists
  in Simple Icons / Material Design Icons / Font Awesome / Octicons / Lucide.
  If you can't verify the slug, suggest one that would be relevant.

#### Choosing the Admonition Type

The type isn't decorative - it signals to the reader *why* the tool is being
flagged. Pick one, don't default to `info` reflexively:

| Type | Use for |
|---|---|
| `note` | Durable reference apps, personal knowledge tools (note-taking apps, settings/config references) |
| `info` | Default - neutral utility software that doesn't fit another tag |
| `tip` | Something actively recommended/popular as the preferred pick among alternatives (a popular OS, browser, etc.) |
| `example` | Labs, VM/hypervisor platforms, sandboxed environments, or alternative implementations demonstrating a concept |
| `question` | Recon/enumeration/investigative/"what is this" tooling (scanners, file-identification, forensic discovery tools) |
| `bug` | Vulnerability-adjacent tooling - bug bounty platforms, CVE PoCs, detection rules/signatures (YARA, Sigma), exploit-focused utilities |
| `warning` | A specific CVE, gotcha, or caveat the reader needs before using something |
| `danger` | Offensive/red-team tradecraft - C2 frameworks, persistence, defense evasion, LOLBins/GTFOBins, pentest distros |
| `success` | Confirmed best practices / validated usage patterns (rare - not typical for a first-pass tool entry) |
| `quote` | A direct quote attributed to a person or content creator, not a tool |

State the type you picked and a one-line reason. The user may override it.

#### Body Content, Order

1. *(Optional)* A `>` blockquote - one to three sentences, verbatim from the
   tool's own README or site copy.
2. One short paragraph, in plain language: what the tool does, and (only if
   supplied by the user) how/where it was found, why it's relevant with a
   markdown link to that source.
3. A bullet list of links using plain autolinks (`<https://...>`) - primary
   repo/site first, then docs/downloads.
4. *Only* if there's real supporting detail worth capturing: one more bullet
   list (features, supported platforms) or a bolded `**Subheading**` for a
   second topic. Don't pad the entry just to add sections.

#### Placement

`docs/resources.md` is organized as nested H2/H3 (occasionally H4) sections,
each with a Material icon in the heading, e.g.:

```
## Utilities
## Note Taking
### Documentation Building
### Diagrams
## Web Browsers
## Operating Systems
## Hypervisors
## Labs & Simulations
## Hardware
## Networking
## DevOps
### Securing GitOps / Supply Chain / Git / CI-CD / SOPS / age / GitHub / VSCode / Ansible / Docker / HashiCorp / Python / Go / JavaScript / YAML / Jinja
## Information Technology
## Information Security
### DNS / NTP
## Offense
### Methodology & Resources / Bug Bounties / Network / Web Application /
    Linux & Unix-like / Windows / macOS / Active Directory / Wireless /
    Cloud / ICS & OT / C2
## Defense
### Security Platforms / Windows / Linux & Unix-like / Threat Hunting
## GRC
## Reverse Engineering
## Malware Analysis
### Known Samples
## Firmware
### Information / Projects / Vulnerabilities
## Forensics
### Memory Acquisition
## OSINT
### Internet Research / Vulnerability Research / Malware Research /
    Software Research / Legal & Corporate Research / Science & Medical
    Research / GEOINT
## Cryptocurrency
## Artificial Intelligence
### Platforms / Tools / Harnesses / Attacks
## Blogs & Authors
```

(Re-check the live headings before relying on this list - it may drift as
the doc grows.)

Match the tool to the **most specific existing subsection** based on what it
*does*, not its marketing category. If nothing genuinely fits, propose a new
H3 under the closest H2 and flag it for the operator - don't invent new
top-level structure unilaterally, and don't reorganize existing sections as
a side effect of adding one entry.

Section contents aren't strictly alphabetized - new entries are generally
appended to the end of the matched section/subsection. Match the blank-line
and indentation spacing of the neighboring entries.

#### Output Format

Respond with, in order:

1. The proposed location, e.g. `## Offense > ### Windows`.
2. The chosen admonition type and a one-line reason.
3. The full block, in a fenced code block, ready to paste.

Do not touch `docs/resources.md` itself unless the user explicitly asks you
to insert the draft.

### For docs/notes/* Entries

<!-- TODO -->