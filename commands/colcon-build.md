---
name: colcon-build
description: Build ROS 2 workspace with colcon, auto-detecting workspace root and creating tracking file
argument-hint: "[colcon build options]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
---

Execute a ROS 2 colcon build with automatic workspace detection and configuration.

## Steps to Execute

### 0. Find Git Repository Root

First, find the Git repository root directory:

- Traverse up from the current working directory to find the directory containing `.git`
- Start from current directory and check each parent directory for `.git` directory or file
- The first directory containing `.git` is the Git repository root
- If no `.git` found, display error: "Error: Not in a Git repository. Please run this command from within a Git repository."
- Store this path as the "file operation directory" for all `.ros_workspace.txt`, `.gitignore`, and `colcon_build_options.yaml` operations

### 0.5. Manage colcon_build_options.yaml

Check and create colcon build options file in Git repository root:

- Check if `colcon_build_options.yaml` exists in the Git repository root
- If it does NOT exist:
  - Create `colcon_build_options.yaml` in the Git repository root with default template:
    ```yaml
    # Colcon build options for this ROS 2 workspace
    # This file will be copied to <workspace_root>/colcon.meta before building
    # See: https://colcon.readthedocs.io/en/released/user/configuration.html

    {
      "names": {
      }
    }
    ```
  - Inform user: "Created colcon_build_options.yaml in [git-repo-root] with default template"
- If it exists:
  - Inform user: "Found existing colcon_build_options.yaml in [git-repo-root]"

### 1. Find ROS 2 Workspace Root

**First, check if `.ros_workspace.txt` exists in the Git repository root:**

- If `.ros_workspace.txt` exists in the Git repository root:
  - Read the workspace root path from the file
  - Verify the path exists and is a valid directory
  - Use this path as the workspace root
  - Inform user: "Using workspace root from .ros_workspace.txt: [path]"
  - Skip to step 3 (Update .gitignore)

**If `.ros_workspace.txt` does NOT exist, detect workspace root:**

- Traverse up from the current working directory to find the ROS 2 workspace root:
  - Start from current directory
  - Check each parent directory for:
    - A `src/` subdirectory exists
    - No `package.xml` file in that directory (to distinguish workspace root from ROS 2 packages)
  - The first directory meeting both conditions is the workspace root
- If no workspace root found, display error: "Error: Could not find ROS 2 workspace root. Make sure you're inside a ROS 2 workspace with a 'src/' directory."

### 2. Create .ros_workspace.txt File

Once workspace root is detected (only if file didn't exist):

- Create `.ros_workspace.txt` in the **Git repository root** (found in step 0)
- Write the absolute path of the workspace root to this file
- Inform user: "Created .ros_workspace.txt in [git-repo-root] with workspace root: [workspace-path]"

### 3. Update .gitignore

Ensure `.ros_workspace.txt` and `colcon_build_options.yaml` are ignored by git:

- Check if `.gitignore` exists in the **Git repository root** (found in step 0)
- If `.gitignore` does not exist:
  - Create `.gitignore` in the **Git repository root** with content:
    ```
    .ros_workspace.txt
    colcon_build_options.yaml
    ```
  - Inform user: "Created .gitignore in [git-repo-root] and added .ros_workspace.txt and colcon_build_options.yaml"
- If `.gitignore` exists:
  - Read the file and check if `.ros_workspace.txt` is already present
    - If NOT present: Append `.ros_workspace.txt` to `.gitignore`
  - Check if `colcon_build_options.yaml` is already present
    - If NOT present: Append `colcon_build_options.yaml` to `.gitignore`
  - Inform user with appropriate message about what was added

### 4. Discover Packages in Current Directory and Update Configuration

Discover packages in current working directory and workspace:

**4a. Find packages in current working directory:**
- Execute `colcon list --names-only` in the **current working directory** (where Claude Code was started)
- Parse the output to get list of package names in current directory
- Store this list as "current_directory_packages"
- If packages found: Inform user: "Found [N] package(s) in current directory: [package names]"
- If no packages found: Inform user: "No packages found in current directory, will build entire workspace"

**4b. Update colcon_build_options.yaml with all workspace packages:**
- Change to the workspace root directory
- Execute: `colcon list --names-only`
- Parse the output to get list of all package names in workspace
- Read `colcon_build_options.yaml` from Git repository root and parse as YAML/JSON
- For each package name from `colcon list`:
  - Check if package exists in `colcon_build_options.yaml` under `names` key
  - If package does NOT exist:
    - Add package entry to `names` with empty configuration: `"package_name": {}`
- Write the updated content back to `colcon_build_options.yaml`
- Inform user: "Updated colcon_build_options.yaml: added [N] new package(s)" (if any were added)

### 5. Copy Build Options to colcon.meta

Before building, copy updated configuration to workspace:

- Read the content of `colcon_build_options.yaml` from Git repository root
- Copy the content to `<workspace_root>/colcon.meta`
  - Overwrite if `colcon.meta` already exists
- Inform user: "Copied colcon_build_options.yaml to [workspace-root]/colcon.meta"

### 6. Execute Colcon Build

Run the colcon build command with intelligent package selection:

- Change to the workspace root directory
- Determine build command based on user arguments and current directory packages:

  **Case 1: User provided package selection arguments**
  - If user arguments contain `--packages-select`, `--packages-up-to`, `--packages-skip`, or `--packages-above`:
    - Use user-provided arguments as-is: `colcon build [user-provided-arguments]`
    - Inform user: "Building with user-specified package selection"

  **Case 2: Packages found in current directory (no user package selection)**
  - If "current_directory_packages" is not empty AND user did not specify package selection:
    - Build with `--packages-up-to` for current directory packages
    - Command: `colcon build --packages-up-to [current_directory_packages] [other-user-arguments]`
    - Example: `colcon build --packages-up-to pkg1 pkg2 pkg3 --symlink-install`
    - Inform user: "Building packages in current directory and their dependencies: [package names]"

  **Case 3: No packages in current directory (no user package selection)**
  - If "current_directory_packages" is empty AND user did not specify package selection:
    - Build entire workspace: `colcon build [user-provided-arguments]`
    - Inform user: "Building entire workspace"

- Display the output of the build process to the user
- If build fails, show the error and exit code

## Important Notes

- All file operations (creating .ros_workspace.txt, colcon_build_options.yaml, modifying .gitignore) should happen in the **Git repository root** (directory containing .git), not the current working directory or workspace root
- The command requires being run from within a Git repository
- Use absolute paths when writing to .ros_workspace.txt
- Preserve existing .gitignore content when adding new entries
- **Intelligent package selection**:
  - `colcon list --names-only` is executed in both current directory and workspace root
  - If packages are found in current directory, automatically builds only those packages and dependencies with `--packages-up-to`
  - If no packages in current directory, builds entire workspace
  - User can override with explicit `--packages-select` or `--packages-up-to` arguments
- New packages are automatically added to `colcon_build_options.yaml` with empty configuration
- `colcon_build_options.yaml` is copied to `<workspace_root>/colcon.meta` before every build
- Run colcon build from the workspace root directory (detected automatically)
- Pass through any arguments the user provides to the colcon build command
