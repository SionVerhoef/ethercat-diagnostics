# Working on this repository

These rules apply to anyone — human or agent — **editing this repository**. They are not
instructions for using the skill on your own projects, where your data is your own.

## This repository is public

`github.com/SionVerhoef/ethercat-diagnostics` is world-readable, mirrored by anyone who
clones it, and installed into other people's setups. Assume every commit is permanent and
already read by a stranger.

## Treat real machine data as if it were under NDA

Diagnostics dumps, error-counter logs, topology exports and code come from real machines
belonging to real customers. Handle all of it as confidential by default — not only what
someone explicitly marked as such.

**Never commit anything that identifies a customer, a site, a machine or a controller.**
That includes, in files *and* in commit messages, changelog entries, eval transcripts and
issue text:

- **Customer, site or end-user names**, machine and serial numbers, project codes —
  including inside device names, box names, comments and file paths
- **AMS NetIDs, IP addresses, MAC addresses, hostnames, credentials, licence keys**
- **ENI files, `.xti` / `.tsproj` exports and Wireshark captures** — they carry MAC
  addresses, serial numbers and the full topology of one plant; a capture also carries its
  process data. Never commit them raw.
- **Setpoints, cycle times, tuning parameters** — commercially sensitive
- **Safety configuration tied to one installation**, and screenshots, which carry all of
  the above in their tree views and title bars

## Anonymise by substitution, not by deletion

The point is to remove identity while keeping the evidence. Renaming a customer to
`Customer_A`, a machine to `Line1` and a box to `Term 3 (EL1008)` costs nothing and keeps
every signal that matters — which port lost link, how the error counters climbed, where
the WKC went wrong.

**Names are stand-ins; structure is real.** If a finding only makes sense with the original
names in place, describe it in prose instead. Naming a *product* is fine and often
necessary: "TwinCAT 4024", "EK1100", "EL7201" describe things anyone can buy. Naming
*whose* machine it is, is not.

**Use placeholders that identify nothing:** AMS NetId `1.2.3.4.1.1`, IP addresses from
`192.0.2.0/24`, MAC addresses from `00:00:5E:00:53:xx`. CI runs `scripts/check_private.py`
and fails on any other NetId, IP or MAC it finds. It cannot recognise a customer name — that
part is on you.

## What you may use, and what you may not

**Research is not redistribution.** Read any source you lawfully can — Beckhoff InfoSys,
device manuals, ETG specifications. Copyright protects expression, not facts: state that AL
status code 0x001B means a sync manager watchdog; do not paste the manual's table of codes.

- **Permissive** (MIT, BSD, Apache-2, 0BSD) — may ship. Copy the upstream `LICENSE` next to
  it and record the source in the same commit.
- **Copyleft** (GPL/LGPL) — do not vendor; it would relicense this skill. Ask first.
- **Proprietary** (InfoSys, vendor manuals, forums) — research only. Cite and link; reproduce
  no passages, tables, diagrams or screenshots.
- **Member-only specs** (ETG) — the technical facts can be stated independently; the
  document's text, tables and figures cannot, and the documents themselves never land here.

Read the licence from the upstream `LICENSE` file, never a GitHub badge. A citation is not
permission. Manuals, specs and real recordings stay in `research/` (gitignored) or outside
the repo.

## Everything here must apply to everybody

This is a general-purpose skill. Rules and examples must make sense to a reader with no
connection to the project they came from. Don't promote one shop's house style to the only
correct answer, and don't write a preference as a rule without saying so.

## If something private has already been committed

**Stop and tell the maintainer.** Do not quietly delete it in a follow-up commit: on a public
repo the old blob stays reachable in history on every clone and fork. Removing it needs a
history rewrite and a force-push, which is the maintainer's decision.
