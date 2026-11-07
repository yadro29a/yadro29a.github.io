+++
title       = 'Writing Cheatsheet'
date        = 2026-11-07
draft       = true
description = 'Every markup feature this theme supports, in one page. Keep as draft.'
categories  = ['meta']
tags        = ['reference']
numbered    = false
+++

Reference page for the Void theme's features. `draft = true` keeps it out of the
built site -- run `hugo server -D` to view it locally.

## Callouts

Five types: `info` (default), `tip`, `warning`, `danger`, `neutral`.

{{< callout type="danger" title="Authorized testing only" >}}
This payload drops a shell. Run it only against a lab VM you own.
{{< /callout >}}

{{< callout type="warning" title="Breaks on patched builds" >}}
Fixed in 2.14.1 -- the offset below only applies to 2.14.0 and earlier.
{{< /callout >}}

{{< callout type="tip" >}}
`curl --path-as-is` keeps `../` sequences intact, which matters for traversal tests.
{{< /callout >}}

Collapsible variant, useful for long exploit dumps:

{{< callout type="neutral" title="Full request/response" collapsible="true" collapsed="true" >}}
Hidden until clicked.
{{< /callout >}}

## Code blocks

Language hint gives you the label + copy button in the block header:

```python
import requests

r = requests.get("http://127.0.0.1:8080/", headers={"X-Forwarded-For": "127.0.0.1"})
print(r.status_code, len(r.content))
```

```http
GET /admin HTTP/1.1
Host: internal.example
X-Original-URL: /public
```

Inline code: `mprotect(page, 0x1000, PROT_READ|PROT_WRITE|PROT_EXEC)`.

## Table of contents

Generated automatically from `##`/`###` headings -- collapsible in the article,
and a sticky sidebar on screens wider than 1472px with scroll-spy. Nothing to
configure per post. Click any heading to copy its permalink.

## Math

Rendered at build time by KaTeX, no client-side JavaScript:

inline $2^{128}$ keyspace, and display:

$$ H(X) = -\sum_{i=1}^{n} p(x_i) \log_2 p(x_i) $$

## Tables

| Mitigation | Blocks | Bypassed by |
|---|---|---|
| ASLR | fixed-address ROP | info leak |
| CFI | arbitrary indirect calls | data-only attacks |
| CSP `nonce` | injected inline script | dangling markup |

## Footnotes and quotes

Request smuggling depends on a parser disagreement.[^1]

> The front end and the back end must agree on where a request ends.

[^1]: Specifically, `Content-Length` vs `Transfer-Encoding` precedence.

## Images

Dropped in `static/images/`, auto-converted to responsive WebP with srcset:

```markdown
![Heap layout after the free](/images/heap-layout.png)
```

## Series

Add `series = ['http-request-smuggling']` to the front matter of several posts
and the theme renders previous/next navigation between them automatically.
