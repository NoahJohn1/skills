# Refund Confirmation Modal - Implementation Summary

## What Was Built

A jQuery UI dialog-based refund confirmation modal (`_RefundConfirmationModal.cshtml`) that collects:

1. **Refund Amount** - text input with autoNumeric formatting (consistent with existing amount fields)
2. **Refund Category** - parent reason dropdown (from `ReasonCache.GetParentRefundReasonList()`)
3. **Refund Reason** - cascading child reason dropdown (from `ReasonCache.GetChildRefundReasonList()`, filtered by selected category)
4. **Notes** - multiline textarea with placeholder text

## UX Decisions

### Visual Hierarchy
- Warning banner at top with yellow background communicates the action's significance
- Amount field is bold and prominent since it's the most critical value
- Error messages appear in red at the top of the dialog

### Consistency with Existing Codebase
- Uses **jQuery UI `.dialog()`** (not Bootstrap) matching all other modals in the app (customer merge, email edit, payment details, etc.)
- Uses `.ModalWindow` class, `.rounded-corners`, and `.button` classes from Site.css
- Cascading parent/child reason dropdowns replicate the exact pattern from RefundOrder.cshtml
- `@@` escaping used for the JSDoc `@param` in inline script (Razor requirement)

### Validation
- Client-side validation before submission: amount > 0, category selected, reason selected
- Error messages displayed inline within the modal (no alert boxes)
- Cancel button always available; Escape key closes the dialog

### API Design
- `openRefundConfirmation(options)` function accepts pre-filled values and a callback
- Callback receives `{ amount, parentReasonId, reasonId, notes }` on confirmation
- Decoupled from form submission logic so it can be integrated into any refund workflow

## How to Integrate

1. Add `@Html.Partial("_RefundConfirmationModal")` to the page
2. Instead of submitting the refund form directly, call:

```javascript
openRefundConfirmation({
    amount: '25.00',
    currency: '$',
    onConfirm: function(result) {
        // result.amount, result.parentReasonId, result.reasonId, result.notes
        $('#refundForm').submit();
    }
});
```

## File
- `_RefundConfirmationModal.cshtml` - Drop-in Razor partial view
