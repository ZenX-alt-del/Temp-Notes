# Workflows
- A workflow is a set of instructions that tells `PowerCenter` how and when to run ETL jobs.
- It contains tasks (like sessions, commands, events, and emails) that can be linked together.
- The integration service executes workflows to move data from sources to targets.

**Why Workflows Matter**
- They control execution order of ETL jobs.
- Allow automation (scheduling jobs at specific times).
- Provide error handling and notifications.
- Enable parallelism for faster data processing.

## Components Of A Workflow

### 1. Tasks
- It is the building block of a workflow.

Types of tasks include:
- **Session Task:** Runs a mapping (source -> transformation -> target).
- **Command Task:** Executes shell commands or scripts.
- **Email Task:** Sends notifications.
- **Event Wait/Trigger Task:** Controls execution based on events.

### 2. Sessions
- It is a specific task that runs a mapping.
- It defines how the integration service should extract, transform and load the data.

### 3. Links
- Connect tasks together.
- Can be sequential or concurrent.

## Exmaple Workflow
```text
[ Workflow ]
   |
   +--> [ Session: Load Customer Data ]
   |
   +--> [ Session: Load Sales Data ]
   |
   +--> [ Command Task: Run Cleanup Script ]
   |
   +--> [ Email Task: Notify Admin ]
```

## Mappings And Workflows - Comparison

| **Aspect** | **Mapping** | **Workflow** |
| -- | -- | -- |
| **Focus** | Data flow logic | Execution control |
| **Defines** | How data moves and transforms | How and when those mappings are run |
| **Contains** | Sources, transformations, targets | Tasks, sessions, dependencies |
| **Built in** | Powercenter Designer | Workflow Manager |
| **Executed By** | Session (via Integration Service) | Workflow (via Integration Service) |
| **Example** | Filter sales data -> aggregate -> load | Run sales mapping nightly at 2 AM |

# Tasks And Sessions
**Tasks:** They are a single unit of work defined in the workflow. They allow you to control workflow logic, sequencing, and automation beyond just running mappings.

Types of tasks include:
- *Session task:* runs a mapping
- *Command task:* executes shell commands or scripts
- *Email task:* sends notifications
- *Timer task:* waits for a specified time before proceeding
- *Decision task:* applies conditional logic to workflow execution
- *Event-Wait / Event-Raise tasks:* synchronize workflows based on events

## Sessions
- A Session is a specific type of task that executes a mapping.
- It defines how data flows from source to target, including source and target connections, commit intervals, error handling, and performance options (e.g., partitioning, caching).
- Each mapping must be associated with a session to run inside a workflow.
- Sessions are the bridge between your mapping design (in Designer) and workflow execution (in Workflow Manager).

## Command Task
- Executes OS-level commands or scripts (`UNIX` shell, Windows batch).
- Useful for pre- or post-session activities like file transfers, cleanup, or archiving.
- *Example,* a command task deletes temporary files after a session completes.

## Email Task
- Sends an email notification to specified recipients.
- Often used for alerts (success, failure, or completion of workflows).
- *Example,* an email task notifies the admin if a session fails.

## Timer Task
- Introduces a delay or waits until a specific time before executing the next task.
- Useful for scheduling dependent jobs.
- *Example,* a timer task waits until midnight before starting a data load.

## Decision Task
- Applies conditional logic to workflow execution.
- Evaluates an expression and decides which path the workflow should follow.
- *Example,* if a file exists, run the session; otherwise, send an email alert.

## Event-Wait Task
- Pauses workflow execution until a specific event occurs (like a file arrival or external trigger).
- *Example,* waits until a sales report file is placed in a directory before starting the load.

## Event-Raise Task
- Signals that an event has occurred, allowing other workflows or tasks waiting on that event to proceed.
- *Example,* after a session completes, an event-raise task triggers another workflow that depends on this completion.

## Worklet Task
- A worklet is a reusable group of tasks (like a mini-workflow).
- A worklet task lets you insert that reusable logic into a workflow.
- *Example,* a worklet task encapsulates error-handling logic and is reused across multiple workflows.

