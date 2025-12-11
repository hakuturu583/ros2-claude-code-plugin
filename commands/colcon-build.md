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

### 1. Find ROS 2 Workspace Root

Traverse up from the current working directory to find the ROS 2 workspace root:

- Start from current directory
- Check each parent directory for:
  - A `src/` subdirectory exists
  - No `package.xml` file in that directory (to distinguish workspace root from ROS 2 packages)
- The first directory meeting both conditions is the workspace root
- If no workspace root found, display error: "Error: Could not find ROS 2 workspace root. Make sure you're inside a ROS 2 workspace with a 'src/' directory."

### 2. Manage .ros_workspace.txt File

Once workspace root is found:

- Check if `.ros_workspace.txt` exists in the workspace root
- If it does NOT exist:
  - Create `.ros_workspace.txt` in the workspace root
  - Write the absolute path of the workspace root to this file
  - Inform user: "Created .ros_workspace.txt with workspace root: [path]"
- If it exists:
  - Inform user: ".ros_workspace.txt already exists"

### 3. Update .gitignore

Ensure `.ros_workspace.txt` is ignored by git:

- Check if `.gitignore` exists in the workspace root
- If `.gitignore` does not exist:
  - Create `.gitignore` with content: `.ros_workspace.txt`
  - Inform user: "Created .gitignore and added .ros_workspace.txt"
- If `.gitignore` exists:
  - Read the file and check if `.ros_workspace.txt` is already present
  - If NOT present:
    - Append `.ros_workspace.txt` to `.gitignore`
    - Inform user: "Added .ros_workspace.txt to .gitignore"
  - If already present:
    - Inform user: ".ros_workspace.txt already in .gitignore"

### 4. Execute Colcon Build

Run the colcon build command:

- Change to the workspace root directory
- Execute: `colcon build [user-provided-arguments]`
  - If user provided arguments (e.g., `--symlink-install`, `--packages-select pkg`), pass them to colcon
  - If no arguments provided, just run `colcon build`
- Display the output of the build process to the user
- If build fails, show the error and exit code

## Important Notes

- All file operations (creating .ros_workspace.txt, modifying .gitignore) should happen in the workspace root, not the current directory
- Use absolute paths when writing to .ros_workspace.txt
- Preserve existing .gitignore content when adding new entries
- Run colcon build from the workspace root directory
- Pass through any arguments the user provides to the colcon build command
