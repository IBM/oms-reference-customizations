---
name: buc-find-building-blocks
description: Search the Order Hub/Call Center component library with the oms-ui-mcp tools before writing any UI. Use this before choosing Carbon, Material, or plain HTML; the library probably already has what you need.
---

# Find an existing component

Order Hub/Call Center ships its own Angular component library. Before you write UI code, search
it. Most of what a customization needs already exists, with the right styling,
accessibility, and theming built in.

**Never reach for a Carbon, Material, or plain-HTML element first.** A hand-rolled
control looks close in a screenshot and then drifts — wrong focus ring, wrong dark
mode, wrong spacing — and it won't follow the product when Order Hub/Call Center restyles.

## 1. Search

Use `search_components` from the `oms-ui-mcp` server. It takes a behavior, a
selector, or a class name:

```
search_components(query="loading spinner")
search_components(query="inline notification", category="components")
search_components(query="buc-table", limit=5)
```

| Parameter | Notes |
|---|---|
| `query` | Required. Keyword, selector (`buc-button`), or class name (`ButtonComponent`) |
| `category` | Optional. One of `components`, `directives`, `pipes`, `interfaces`, `injectables`, `classes`, `modules` |
| `limit` | Optional, 1–50, defaults to 20 |

Search for the *behavior* you want before trying the name you expect. The library's
naming doesn't always match what a feature is called in the UI.

## 2. Read the API before wiring it

A search result identifies a component but does not explain how to use it. Inspect
the component you selected:

```
get_component_details(name="buc-inline-loading")
```

That returns inputs, outputs, and usage examples. Check that the inputs
meet your requirements before choosing the component. A component that looks right but
can't be driven the way you need costs more than picking a different one early.

When the documented API isn't enough — you need to know how it behaves internally,
or the examples don't cover your case:

```
get_component_source(name="buc-inline-loading")
```

Both accept either the selector or the class name, plus the same optional
`category`.

## 3. When nothing matches

Broaden the query before concluding it doesn't exist. Drop the `category` filter,
try the behavior instead of the name, and try a synonym — "notification" vs
"toast" vs "banner".

Only after that, use a Carbon primitive, and confirm with the user that you didn't
miss something. Say which queries you tried, so the decision is reviewable.

## Whole building blocks have their own skills

For a table, search panel, search-result page, summary panel, or dashboard, don't
assemble one from components. Those have schematics that also wire up config and
translations — follow the matching skill:

| You need | Skill |
|---|---|
| A data table | [buc-generate-table-component](../buc-generate-table-component/SKILL.md) |
| A summary or field panel | [buc-generate-summary-component](../buc-generate-summary-component/SKILL.md) |
| A search criteria page | [buc-generate-search-panel](../buc-generate-search-panel/SKILL.md) |
| A search results page | [buc-generate-search-result](../buc-generate-search-result/SKILL.md) |
| A whole search experience from one config | [buc-generate-search-from-config](../buc-generate-search-from-config/SKILL.md) |
| A dashboard and its widget cards | [buc-generate-dashboard](../buc-generate-dashboard/SKILL.md) |

## If the tools aren't available

The `oms-ui-mcp` server isn't always connected. If the tools don't appear, ask the
user whether they can enable it — a connected server is much more reliable than
guessing at the library.

If they can't, search the installed library source under
`node_modules/@buc/common-components/` for a matching selector before writing
anything new. Tell the user you're working without the search tools, so they know
your component choices are less certain than usual.

## Confirm before you build

Once you've picked something, say which component you chose and why it fits, before
writing the code. A wrong choice is far cheaper to correct at that point than after
a template is built around it.

Use `get_component_details` and `get_component_source` to inspect a search result's
inputs, outputs, imports, and usage before implementing it.
