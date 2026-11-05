---
name: cc-customize-styles
description: Override Call Center styles globally with custom_styles.css for the shell (login, banner, navigation, global search) and the hub (home page and every module page).
---

# Global style overrides

Two stylesheets load at startup and override the base CSS:

| Scope | File |
|---|---|
| Shell: login page, top banner, left navigation, global search | `callcenter-code/shell-ui/assets/custom_styles.css` |
| Hub: home page, tabs, every module page | `callcenter-code/call-center-hub/assets/custom_styles.css` |

For styles on one page only, use the component's own SCSS in that route's `src-custom`
instead.

## 1. Find selectors

Ask the user to inspect the element in DevTools. Prefer stable hooks: Carbon classes
(`cds--*`), shell classes (`cui-*`), component tags (`buc-cc-alerts-widget`), and `tid`
attributes (`[tid*="..."]`). Avoid broad selectors such as `div` or `*`, which affect every
page. Base styles use specific selectors, so `!important` may be needed.

## 2. Write the overrides

Shell:

```css
.cui-topbanner-header { background-color: #0f62fe !important; color: #fff !important; }
.cui-topbanner-header svg { fill: #fff !important; stroke: #fff !important; }
.cui-ribbon .ribbon-products-container { background-color: #161616; }
.feature-item.selected { background-color: #393939 !important; }
[tid*="CategorySearchComponentSelectCategoryDropdown"] button { background: #262626 !important; }
cui-login button, cui-login input { border-radius: 4px; }
```

Hub:

```css
cc-tabs-container { font-family: "IBM Plex Sans", sans-serif; }
.search-header { background-color: #e5f6ff; }
buc-cc-alerts-widget a { color: #0f62fe; }
```

## 3. Prepare for deployment

Before building the package, copy `call-center-hub/assets/custom_styles.css` to
`callcenter-code/call-center-hub/custom_styles.css`. The shell file ships as is.

## Verify

Both folders are mounted into `om-call-center-base` in the DTK, so a reload picks up
changes. Ask the user to reload Call Center and check the login page, banner,
navigation, home page, and one module page.

If nothing changes, the selector isn't specific enough (add `!important`) or the file isn't in the mounted
folder. If unintended pages changed, the selector is too broad.
