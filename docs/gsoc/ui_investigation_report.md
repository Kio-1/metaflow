Investigation: Local vs. Service Metadata Discrepancies
Overview
This investigation explores the differences in how Metaflow handles metadata when running in a pure local environment versus a service-backed environment. The goal is to identify "friction points" that the Metaflow UI 2.0 (Standalone Mode) must address to provide a seamless developer experience without backend infrastructure.

1. Structural Discrepancies
In the local datastore (.metaflow folder), metadata is fragmented across multiple files rather than being consolidated into a single database record:

0.attempt.json: Stores the Unix timestamp of the task attempt (e.g., 1774438653.47...).

0.runtime: Stores the task's final state (success: true, killed: false).

_meta/_self.json: Contains system-level tags like python_version, metaflow_version, and user_name.

Finding: A Standalone UI must implement a "collector" logic similar to the Python Client API to aggregate these fragmented files into a single "Run" view.

2. Identified Friction Points (Bugs)
During testing in a WSL 2 environment, the following issues were observed when attempting to view local runs in the current Metaflow UI:

A. DAG Rendering Failure (500 Error)
The DAG view consistently fails with an Internal Server Error 500 when accessing local runs.

Stack Trace: TypeError: expected str, bytes or os.PathLike object, not NoneType.

Implication: The current UI backend expects cloud or service-based pathing that does not exist in a local-only setup.

B. Artifact and Log Invisibility
Logs: The UI often displays "No logs" for local tasks, even when 0.runtime_stdout.log exists on disk and contains data.

Artifacts: Simple variables (e.g., a name string) are marked as "Artifact not currently displayable in the UI".

3. Tag Discovery Issues
While the Python Client API can discover tags like user:bilal and runtime:dev, these are notably absent from the task-level JSON files (0.task_begin, 0.task_end).

System Tags: These are only found in the hidden _meta directory.

Conclusion: The UI currently relies on the Metadata Service to "flatten" these tags into a database for easy retrieval.

Proposed Direction for UI 2.0
To achieve a "zero-infrastructure" local UI, we must:

Direct File Parsing: Build a backend capability to parse the .metaflow directory structure directly.

Robust Pathing: Fix the NoneType pathing errors in the DAG rendering logic to support local OS paths.

Local Artifact Rendering: Enable the UI to read and display local data.json objects without a middleman service.
