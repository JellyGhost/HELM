# Core Lifecycle

The HELM Core Lifecycle controls how Core initializes, operates, handles failures, shuts down, and recovers. Core is supervised by systemd and manages the foundational infrastructure required by HELM applications.

## Initialization

- systemd launches Core during device startup
- Core loads and validates system and Core configuration
- Components initialize according to their individual dependencies. Independent components may initialize concurrently
- Essential components must initialize successfully. Nonessential components may fail without preventing startup
- Essential component failures are only allowed a limited amount of retry attempts. If all retry attempts are used, Core terminates and systemd handles restart attempts
- Nonessential startup failures are marked unavailable and recieve background recovery attempts
- Core launches essential applications, including Launcher
- Core becomes `Operational` only after all essential components and Launcher are ready

## Runtime Operation and Component Failures

- Core mantains its lifecycle state independently of individual component availability
- Component availability is tracked seperately
- Failure of an essential component causes immediate Core shutdown and systemd recovery
- Failure of a nonessential component causes it to be disabled and reported. It remains unavailable until manually restared or Core restarts
- Relevant applications recive notifications about service availability changes
- Applications can query current system and component states via IPC

## Shutdown

Core uses a hybrid graceful-shutdown process:
1. Core enters `ShuttingDown`
2. New application launches and communication channel requests are rejected
3. Core broadcasts `CORE_SHUTTING_DOWN`
4. Applications acknowledge the event and begin individual shutdown processes
5. Core monitors application termination
6. Applications receive and configureable grace period to finish operations.
7. Applications can submit shutdown warmings classified as `Informational`, `Warning`, or `Critical`
8. Core presents warnings centrally without requiring individual acknowledgment
9. Applications that remain active are presented to the user in a combined shutdown dialogue

The user can:
- Wait for applications to finish
- Force-terminate individual applications
- Force-terminate all remaining applications

Choosing Wait starts another configurable grace period. This repeats until applications exit or the user chooses force termination.

After all applications terminate, Core shuts down its components in reverse dependency order, flushes diagnostic information, and terminates.

## Core Failure and Restart

- Core attempts to preserve diagnostic information before terminating
- systemd supervises Core and performs a limited, configurable number of automatic restart attempts
- After restarting, Core examins available crash information
- Processes beloning to the previous Core instance are terminated to eliminate orphaned applications
- Essential applications, particularly Launcher, are relaunched
- Ordinary applications remain closed
- If all restart attempts are used, automatic recovery stops and the dedicated recovery enviornment is launched

## Recovery Enviornment

The recovery enviornment is an independent system-level program managed by systemd. It does not depend on Core or Launcher

Its capabilities include:
- Displaying failure information and error IDs
- Inspecting diagnostic logs and crash reports
- Identifying missing or invalid configuration files
- Viewing system and component status
- Manually restarting Core
- Rebooting and shutting down the device

Additional characteristics include:
- Configuation access is read-only
- Diagnostic export to USB is planned for a future version
- It uses a simplified HELM-style UI
- It supports both touchscreen and physical controls
- Detailed diagnostics are accessible via expandable views
- systemd automatically restarts it after failure, subject to limited retry attempts
- After a successful Core restart and Launcher initialization, it exits automatically and returns to normal HELM operation

## Unexpected Shutdown and Integrity Checks

Following unexpected power loss or forced hardware shutdown, HELM performs a lightweight integrity check pre-Core startup

Checks include:
- Essential config files
- Diagnostic log accessibility and incomplete records
- Essential system state
- Leftover application processes and incomplete shutdown state

Responses depend on severity:
|**Severity**|**Noncritical**|
|------------|---------------|
| Noncritical | Log the issue and continue startup |
| Recoverable | Attempt safe recovery, then continue |
| Critical | Prevent normal startup and enter recovery mode |

Integrity-check results and recovery actions are recorded through the existing operational logging system

## Lifecycle States and Events

Core uses the following lifecycle states:

| **State**|**Description**|
|---|---|
|`Stopped`|Core is not running|
|`Initializing`|Core is initializing components|
|`Operational`|Essential components and Launcher are ready|
|`ShuttingDown`|Core is performing an orderly shutdown|
|`Failed`|Core encountered a fatal failure and is terminating|

Component availability is tracked independently, using states such as `Initializing`, `Available`, `Unavailable`, and `Recovering`

Important lifecycle events broadcast via IPC:
- `CORE_OPERATIONAL`
- `CORE_SHUTTING_DOWN`
- `CORE_FAILED`

Application can subscrive to relevants events or query Core's current state