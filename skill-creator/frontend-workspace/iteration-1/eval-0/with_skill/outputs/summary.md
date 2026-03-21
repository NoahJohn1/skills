# Customer Search Page - UX Review & Implementation Summary

## UX Issues Identified

1. **Visual hierarchy**: Original had no clear heading, search controls were buried in a raw `<table>` layout with no visual grouping
2. **Layout & spacing**: Inconsistent padding, no separation between search area and results, table-based layout for form controls
3. **Interactive elements**: Generic "button rounded-corners" class with no hover/active/disabled states, no placeholder text in select dropdown
4. **Responsiveness**: Table-based form layout did not adapt to smaller viewports
5. **Accessibility**: Missing `<label>` elements for form controls, no ARIA attributes on search region, loading indicator, or results table, no `role` attributes
6. **User flow**: No empty state guidance -- users see a blank area with no indication of what to do; no keyboard shortcut (Enter to search); no select-all for merge workflow; no result count indicator
7. **Loading state**: Inline `display:none` style with `<center>` tag, no descriptive loading text

## Changes Made

### Search Panel
- Replaced raw `<table>` layout with flexbox-based `.search-controls` for responsive wrapping
- Added proper `<label>` elements associated with inputs
- Added placeholder text to the parameter `<select>` ("-- Select a parameter --")
- Card-style container with subtle shadow for visual hierarchy
- Added keyboard hint showing `Enter` to search

### Results Area
- Created a structured results panel with header showing result count
- Pre-defined semantic `<table>` with proper `<thead>`/`<tbody>`, `scope="col"` on headers, and `role="grid"`
- Added select-all checkbox in table header for batch merge operations
- Hover state on table rows for scan-ability
- Monospace styling on Customer ID column for readability

### Empty/Loading States
- Added empty state with guidance text ("Select a parameter and enter a value...")
- Loading state uses `role="status"` and `aria-live="polite"` for screen reader announcements
- JS toggle functions `showResults()` / `showEmpty()` manage state transitions

### Accessibility
- `aria-label` on search select, search button, results table, and merge dialog
- `aria-modal="true"` on merge dialog
- `role="dialog"` on merge window
- Keyboard shortcut: `Enter` key triggers search from any non-textarea input

### Responsiveness
- `@@media (max-width: 768px)` breakpoint stacks search controls vertically and tightens table padding
- `flex-wrap: wrap` on search controls ensures graceful wrapping at intermediate widths

### Merge Workflow
- Moved merge button below results (only visible when results are shown)
- Added live "N selected" count label
- Select-all checkbox in table header for efficiency

## Tradeoffs & Notes
- Kept all existing JS file references (`cc.customer.search.js`, `cc.customer.details.js`) and the `_DynamicQueryPartial` partial for backward compatibility
- Styles are inlined in the view rather than a separate CSS file to keep the change self-contained; could be extracted to `Site.css` in a future pass
- The results table skeleton is pre-rendered; the existing AJAX JS that populates `#search-result` will need minor updates to target `#search-result-body` tbody and call `showResults()` after loading
- Used Bootstrap-compatible class naming conventions even though the existing app doesn't use Bootstrap directly -- these custom styles follow similar patterns and won't conflict

## File Location
The view would be placed at: `CC/Views/Customers/Index.cshtml` (replacing the existing customer search page)
