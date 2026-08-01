---
status: accepted
---

# Central formatter owns all Telegram message layout

## Context

Telegram message layout was redefined in every notification workflow. Each of
the nine PR/CI handlers hand-built its own message string (some inline, some via
github-script), and five external repos called `send-telegram-message.yml`
directly with free-form `message:` strings for deploy and terraform events. The
format had already drifted: inconsistent field labels, the 300-char truncation
rule copy-pasted, and the two website repos sending bare one-line strings with no
repository link or actor. Any new caller could invent a fresh layout.

## Decision

A single reusable workflow, `telegram-notify.yml`, owns all presentation: header,
emoji, field labels, order, spacing, body truncation, and the derived Repository
line. Callers pass structured data through typed inputs and select layout with a
required `event_type`; they never supply a message body. The vocabulary covers
both families - PR/CI events and deploy/terraform/drift - so the whole estate
renders one format.

`send-telegram-message.yml` (the free-form escape hatch) is removed. There is no
generic passthrough event, because a caller-supplied body is exactly the
uncontrolled layout this replaces.

## Consequences

- The formatter is self-contained: it both formats and sends. This is forced, not
  stylistic. GitHub caps nested reusable workflows at four levels, and the chain
  is already `consumer -> notify.yml -> handler -> telegram-notify.yml`. Delegating
  the send to a further workflow would be a fifth level and would fail to run.
- Adding a field or event kind means editing one file. That is the point - it is
  the chokepoint that keeps every message consistent.
- Removing the escape hatch is a breaking change for direct callers, so it ships
  in phases: add the formatter and migrate the handlers, migrate the five consumer
  repos, then delete the old sender.
- The two website repos that sent bare strings are upgraded to the full format.
