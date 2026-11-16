# App Manager

The app manager is responsible for dsiscovering, registering, launching, monitoring, and terminating apps.

Its primary purpose is to provide modular application enviornment in which individual apps can operate independently without compromising the stability of the system

Apps run as serperate OS processes. The core remains active in the background and manages their lifecycle

## App structure

Apps are found in the Pi/apps directory. Each app has its own directory containing a manifest, source code, and assets

#### Example structure:
    Pi/
    └── apps/
        └── Telemetry/
            ├── manifest.json
            ├── src/
            │   └── main.py
            └── assets/
                  └── icon.png

Apps should be self contained and should not depend on the internal implementations of other applications.

Shared functionality, including communication with the Windows companion, should be accessed through interfaces provided via the HELM core.

## App Discovery

During initialization, the App Manager scans Pi/apps for applications containing a manifest.json file

Each manifest is read and validated before the app is registered

Apps with invalid or incomplete nabufests are excluded from registration. An invalid app must not prevent HELM core or other apps from functioning

Discovery must not execute app code. Apps are only launched when explicity selected through the launcher

## App Manifests
Every app must contain a manifest.json file describing its identity and metadata

The manifest is structured as such:
```json
{
    "id": "sample.id",
    "name": "sampleNAME",
    "version": "versionid",
    "description": "SampleDescription",
    "entry_point": "App.execution.entry.point",
    "icon": "assets/icon.png",
    "enabled": true/false,
    "category": "App Category"
}
```
App identifiers should use a consistent lowercase reverse-domain naming convention, such as "com.helm.telemetry".

The App Manager should validate manifest fields and verify that referenced resources are available before making an app available to the launcher

## App Lifecycle

Apps follow the following defined lifecycle:
- Discovery - The Application Manager locates application directories
- Validation - Application manifests and required resources are checked
- Registration - Valid and enabled applications are made available to the launcher
- Launch -  The Application Manager starts the selected application as a separate process
- Execution - The application runs independently while HELM Core remains active
- Termination - The application exits normally or is terminated following a failure
- Recovery - Control returns to the launcher

Initially, HELM will support one active graphical application at a time to avoid excess resource consumption

## Process Isolation
Each app runs in its own OS process. This provides several advantages:
- App failures do not directly terminate HELM Core
- App memory is released when its process exits
- Apps can be developed and tested independently
- Core can monitor application execution and recover from unexpected termination

HELM Core and its background services must remain operational while apps are running

## Communication with Core

Applications must not independently implement duplicate versions of shared HELM services.

Instead, they communicate with HELM Core via IPC interfaces

These interfaces provide access to shared functionality, including
- PC connection status
- Telemetry data
- Communication protocol services
- App error reporting
- Shared config

The App manager is responsible for app lifecycle management, while other Core components retain responsibility for their respective services

## Error Handling and Recovery

The App Manager monitors app processes and handles unexpected failures.

HELM uses 2 complementary error-detection mechanisms

### App-reported Errors

Apps can report errors to HELM Core via a shared error-reporting interface

Reports should includer relevant diagnostic info, such as:
-Application ID
-Error Category
-Exception type
-Error message
-Additional diagnostic details, when available

### Core-Detected Failures:
The App Manager independently monitors app processes.
It detects conditions such as:
- Unexpected process termination
- Nonzero process exit status
- App launch failures
- Termination without an error report
        
## Error Logging

HELM Core maintains an error log detailing:
- Unique Error event ID
- App ID and version
- Process ID
- Timestamp
- Error category and exception type
- Error message and traceback, when applicable

Error event IDs should identify individual incidents, while categories identify the general error type.
    
## Launcher Recovery

When an app terminates unexpectedly:
- Core detects and records the failure
- The App Manager restores the launcher
- The launcher displays a notification that an app encountered an error
- The notification includes the app name, error category, and event ID
- The user can dismiss the notification and continue using HELM
        
Normal app shutdowns must not generate error notifications

An app failure must not require a full HELM restart.

## Extensibility

The App Manager should support the addition of new apps without requiring modifications to HELM Core or additional apps

New apps should be discoverable via their respective manifests and should use established Core interfaces for shared functionality

The app architecture should remain modular so that additional app categories and capabilities can be introduced as HELM develops

## Future considerations

The following details are open for future design and testing:
- The exact IPC mechanism betweens apps and Core
- App startup and shutdown procedures
- Handling apps that fail to terminate normally
- App permission and resource restrictions
- Manifest versioning and compatibility
- App-specific config managment
- Error log storage, rotation, and retention
- Memory/CPU overhead associated with process isolation