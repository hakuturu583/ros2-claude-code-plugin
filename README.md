# ROS 2 Workspace Plugin

Claude Code plugin for ROS 2 workspace management and colcon build automation.

## Features

- **Automatic workspace detection**: Finds ROS 2 workspace root by traversing up directories
- **Workspace tracking**: Creates `.ros_workspace.txt` with workspace root path
- **Git integration**: Automatically adds `.ros_workspace.txt` to `.gitignore`
- **Colcon build**: Executes `colcon build` with optional arguments

## Installation

Copy this plugin to your Claude Code plugins directory or use it locally:

```bash
cc --plugin-dir /path/to/ros2-claude-code-plugin
```

## Usage

### `/colcon-build` command

Execute colcon build with automatic workspace detection:

```bash
/colcon-build
```

With options:

```bash
/colcon-build --symlink-install
/colcon-build --packages-select my_package
/colcon-build --symlink-install --packages-select my_package
```

**What it does:**

1. Finds Git repository root (directory containing `.git`)
2. Checks for `.ros_workspace.txt` in Git repo root, or traverses up to find ROS 2 workspace root
   - Looks for `src/` directory without `package.xml` in parent
3. Creates `.ros_workspace.txt` in Git repo root (if not exists) with workspace root path
4. Adds `.ros_workspace.txt` to `.gitignore` in Git repo root (if not already present)
5. Executes `colcon build` with provided arguments

## Requirements

- ROS 2 installed
- `colcon` build tool available in PATH
- ROS 2 workspace with standard structure (must have `src/` directory)
- Git repository initialized (command must be run from within a Git repository)

## License

MIT
