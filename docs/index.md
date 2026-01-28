# IsiOne Documentation

IsiOne is a collaborative task management app for recurring chores and responsibilities. Users create pods (shared spaces for households or teams) and define tasks with configurable timing for pending and overdue states. Tasks can be completed via web through the user interface, or by mobile scanning NFC tags.

A physical controller device with status LEDs provides at-a-glance visibility of task states—no need to open an app to see what needs doing.

This documentation covers interfaces, APIs, and flows between components. Implementation details live in each component's own repository.

## Architecture

- [Resources](architecture/resources.md) - Core domain concepts: pods, tasks, entries, memberships
- [Task Status](architecture/task-status.md) - How task status is calculated from completion history
- [Decisions](architecture/decisions.md) - Technical decisions affecting the API and interfaces

### Flows

- [Create Pod](architecture/flows/create-pod.md) - Creating a new collaborative space
- [Create Task](architecture/flows/create-task.md) - Adding a task to a pod
- [Complete Task](architecture/flows/complete-task.md) - Recording task completion via web or NFC
