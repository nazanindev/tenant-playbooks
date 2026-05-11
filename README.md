# Authoring and adding a new jurisdiction

Content lives in `content/<city-slug>/` as one markdown file per topic per language. Adding a city means writing those files, deploying, and running ingest. No schema changes required.

### 1. Create the content files

Each file follows this structure:

```
content/
  seattle/
    seattle-cant-pay-rent.en.md
    seattle-heat-not-working.en.md
    ...
```

File name convention: `<topic-slug>.<lang>.md`. Use the city slug as a prefix on the topic slug to keep topics globally unique (e.g. `seattle-cant-pay-rent`, not `cant-pay-rent`).

### 2. File format

Every file has a YAML frontmatter block followed by one statement per paragraph:

```markdown
---
jurisdiction: seattle
topic: seattle-cant-pay-rent
language: en
title: "Can't Pay Rent in Seattle — What Are My Options?"
intro: "One or two sentences framing the situation for the renter."
sources:
  - id: wa-rcw-5918057
    url: "https://app.leg.wa.gov/RCW/default.aspx?cite=59.18.057"
    publisher: "Washington State Legislature"
    jurisdiction: washington
    kind: statute
    locator: "RCW 59.18.057"
  - id: seattle-renter-guide
    url: "https://www.seattle.gov/rentinginseattle/renters"
    publisher: "City of Seattle"
    jurisdiction: seattle
    kind: gov_guidance
    locator: ""
---

Washington law requires a landlord to serve a written notice before starting eviction for nonpayment. [wa-rcw-5918057]

Contact your landlord in writing as soon as you know you will miss a payment. Early documentation helps. [editorial]
```

**Frontmatter fields**

| Field | Required | Notes |
|---|---|---|
| `jurisdiction` | yes | Slug of the city (e.g. `seattle`). Must be lowercase, hyphenated. |
| `topic` | yes | Slug for this playbook. Prefix with city slug to avoid collisions. |
| `language` | no | Defaults to `en`. Use ISO 639-1 codes (`es`, `zh`, etc.). |
| `title` | yes | Shown as the playbook heading. |
| `intro` | no | Short framing paragraph shown above the statements. |
| `sources` | yes | At least one source is required. |

**Source `kind` values**: `statute`, `regulation`, `gov_guidance`, `nonprofit`, `editorial`

**Citation syntax**

End each statement paragraph with one or more citation tokens. The parser rejects any prose paragraph without a citation — there is no way to publish an uncited claim.

```
[source-id]            — cites the source using its frontmatter locator
[source-id:Section 5]  — overrides the locator for this statement only
[editorial]            — marks practical advice not traceable to a single statute
```

Multiple citations on one statement: `[wa-rcw-5918057] [editorial]`

### 3. Deploy and ingest

```bash
fly deploy --no-cache          # rebuilds the image with the new content baked in
fly ssh console --app defensiverenting -C "/ingest -content /content"
```

`--no-cache` is necessary because Depot sometimes serves stale content layers from its build cache.

The ingest command is idempotent — re-running it on existing content is safe.

---
