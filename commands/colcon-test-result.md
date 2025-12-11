---
name: colcon-test-result
description: Display detailed test results from the ROS 2 workspace
argument-hint: "[colcon test-result options]"
allowed-tools:
  - Bash
  - Read
---

Display detailed test results from the ROS 2 workspace with colcon test-result.

## Steps to Execute

### 0. Find Git Repository Root

First, find the Git repository root directory:

- Traverse up from the current working directory to find the directory containing `.git`
- Start from current directory and check each parent directory for `.git` directory or file
- The first directory containing `.git` is the Git repository root
- If no `.git` found, display error: "Error: Not in a Git repository. Please run this command from within a Git repository."

### 1. Find ROS 2 Workspace Root

**Check if `.ros_workspace.txt` exists in the Git repository root:**

- If `.ros_workspace.txt` exists in the Git repository root:
  - Read the workspace root path from the file
  - Verify the path exists and is a valid directory
  - Use this path as the workspace root
  - Inform user: "Using workspace root from .ros_workspace.txt: [path]"
- If `.ros_workspace.txt` does NOT exist:
  - Display error: "Error: .ros_workspace.txt not found. Please run /colcon-build first to set up the workspace."
  - Stop execution

### 2. Save Current Directory

- Get the current working directory (where Claude Code was started)
- Store this path to return to after displaying test results

### 3. Display Test Results

Execute colcon test-result in the workspace:

- Change to the workspace root directory (from .ros_workspace.txt)
- Execute: `colcon test-result --verbose [user-provided-arguments]`
  - Default: `colcon test-result --verbose` (shows detailed test results)
  - If user provided arguments (e.g., `--all`, `--test-result-base`), pass them to colcon test-result
  - The `--verbose` flag is always included unless user explicitly provides other verbosity flags
- Display the full output of the command to the user
- If command fails, show the error and exit code

### 4. Return to Original Directory

- Change back to the original directory (saved in step 2)
- Inform user: "Returned to original directory: [original-path]"

## Important Notes

- The command requires being run from within a Git repository
- Requires `.ros_workspace.txt` to exist (run `/colcon-build` first if it doesn't)
- Always returns to the original directory after displaying results
- Default verbosity is `--verbose` for detailed output
- User can override with options like `--all`, `--test-result-base`, etc.
- Does not modify any files or state
- Works from any directory within the Git repository

## Usage Examples

```bash
# Display detailed test results
/colcon-test-result

# Display all test results including passed tests
/colcon-test-result --all

# Use custom test result directory
/colcon-test-result --test-result-base custom_results
```
