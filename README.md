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
2. Creates/checks `colcon_build_options.yaml` in Git repo root (build configuration)
3. Checks for `.ros_workspace.txt` in Git repo root, or traverses up to find ROS 2 workspace root
   - Looks for `src/` directory without `package.xml` in parent
4. Creates `.ros_workspace.txt` in Git repo root (if not exists) with workspace root path
5. Adds `.ros_workspace.txt` and `colcon_build_options.yaml` to `.gitignore` (if not already present)
6. Runs `colcon list --names-only` to discover all packages in workspace
7. Updates `colcon_build_options.yaml` by adding any missing packages with empty configuration
8. Copies `colcon_build_options.yaml` to `<workspace_root>/colcon.meta`
9. Executes `colcon build` with provided arguments

## Build Configuration

The plugin manages build options through `colcon_build_options.yaml` in your Git repository root:

- **Automatic creation**: File is created with default template on first run
- **Auto-discovery**: Runs `colcon list --names-only` before each build to discover packages
- **Auto-update**: Automatically adds missing packages to the configuration file
- **Git ignored**: Automatically added to `.gitignore` (local configuration)
- **Applied before build**: Content is copied to `<workspace_root>/colcon.meta` before each build

### Example `colcon_build_options.yaml`

```yaml
# Colcon build options for this ROS 2 workspace
# This file will be copied to <workspace_root>/colcon.meta before building
# See: https://colcon.readthedocs.io/en/released/user/configuration.html

{
  "names": {
    "my_package": {
      "cmake-args": ["-DCMAKE_BUILD_TYPE=Release"]
    },
    "another_package": {
      "cmake-args": ["-DBUILD_TESTING=OFF"]
    },
    "sensor_driver": {},
    "controller_node": {}
  }
}
```

**How it works:**
1. On first run, the file is created with an empty `names` object
2. Each time you run `/colcon-build`, the plugin discovers all packages with `colcon list --names-only`
3. Any packages not in the file are automatically added with empty configuration `{}`
4. You can then customize build options for specific packages
5. Changes take effect on the next build

## Requirements

- ROS 2 installed
- `colcon` build tool available in PATH
- ROS 2 workspace with standard structure (must have `src/` directory)
- Git repository initialized (command must be run from within a Git repository)

## License

MIT
