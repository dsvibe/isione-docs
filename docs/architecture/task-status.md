# Task Status Calculation

Task status is calculated based on the time elapsed since the most recent [TaskEntry](resources.md#taskentry). Each [Task](resources.md#task) defines its own `pendingAfter` and `overdueAfter` durations.

## Statuses

### Done

The task has been completed recently and doesn't need attention yet. This is the "all good" state.

A task is **Done** when the time since last completion is less than `pendingAfter`.

### Pending

The task will need to be done soon. This is an early reminder before it becomes urgent.

A task becomes **Pending** when the time since last completion exceeds `pendingAfter` but is still less than `overdueAfter`.

### Overdue

The task should have been done by now and needs immediate attention.

A task becomes **Overdue** when the time since last completion exceeds `overdueAfter`.

### Never Done

The task has no completion history. It appears in this state when first created until someone completes it.

Never Done is treated as **Overdue** for display and notifications.

## Summary

| Status | Condition |
|--------|-----------|
| Done | `now - lastCompletedAt < pendingAfter` |
| Pending | `pendingAfter <= now - lastCompletedAt < overdueAfter` |
| Overdue | `now - lastCompletedAt >= overdueAfter` |
| Never Done | No task entries exist |

## Example

A task "Water the plants" with:
- `pendingAfter`: 5 days
- `overdueAfter`: 7 days

If completed on Monday:
- Monday → Friday: **Done**
- Saturday → Sunday: **Pending**
- Following Monday onwards: **Overdue**
