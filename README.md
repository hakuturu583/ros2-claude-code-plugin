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

### Smart Package Selection

The plugin automatically determines which packages to build based on your current directory:

**Scenario 1: Working on specific package**
```bash
# Navigate to package directory
cd ~/ros2_ws/src/my_robot_package

# Start Claude Code and run build
cc
/colcon-build

# Result: Only builds my_robot_package and its dependencies
# Command executed: colcon build --packages-up-to my_robot_package
```

**Scenario 2: Working in workspace root**
```bash
# Navigate to workspace root
cd ~/ros2_ws

# Start Claude Code and run build
cc
/colcon-build

# Result: Builds entire workspace
# Command executed: colcon build
```

**Scenario 3: Override with explicit package selection**
```bash
# From any directory, you can override
/colcon-build --packages-select specific_package

# Result: Builds only specific_package (not its dependencies)
```

**What it does:**

1. Finds Git repository root (directory containing `.git`)
2. Creates/checks `colcon_build_options.yaml` in Git repo root (build configuration)
3. Checks for `.ros_workspace.txt` in Git repo root, or traverses up to find ROS 2 workspace root
   - Looks for `src/` directory without `package.xml` in parent
4. Creates `.ros_workspace.txt` and `build.sh` script in Git repo root (if not exists)
   - `.ros_workspace.txt`: stores workspace root path
   - `build.sh`: build script with ccache support for faster compilation
5. Adds generated files to `.gitignore` (if not already present)
6. **Intelligent package discovery**:
   - Runs `colcon list --names-only` in current directory to find packages you're working on
   - Runs `colcon list --names-only` in workspace root to discover all packages
7. Updates `colcon_build_options.yaml` by adding any missing packages with empty configuration
8. Copies `colcon_build_options.yaml` to `<workspace_root>/colcon.meta`
9. **Smart build execution via build.sh**:
   - Executes build through `build.sh` script with ccache enabled
   - If packages found in current directory: builds only those + dependencies with `--packages-up-to`
   - If no packages in current directory: builds entire workspace
   - User can override with explicit package selection arguments
   - Displays ccache statistics after build completion

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

## Build Script (build.sh)

The plugin automatically generates and updates a `build.sh` script on every run in your Git repository root with ccache support:

### Features

- **Smart ccache integration**: Automatically detects and uses ccache if installed
- **Helpful warnings**: Displays installation instructions if ccache is not found
- **Works without ccache**: Falls back to normal gcc/g++ if ccache is unavailable
- **Workspace-aware**: Reads workspace root from `.ros_workspace.txt`
- **Statistics**: Displays ccache statistics after each build (when ccache is used)
- **Standalone usage**: Can be used independently from Claude Code plugin

### Example usage

```bash
# Auto-generated and updated on every /colcon-build run
./build.sh --packages-up-to my_package --symlink-install

# The script will:
# 1. Check if ccache is installed (shows warning if not found)
# 2. Enable ccache if available (CC="ccache gcc", CXX="ccache g++")
# 3. Navigate to workspace root
# 4. Run: colcon build --packages-up-to my_package --symlink-install
# 5. Show ccache statistics (if ccache is used)
```

**Without ccache:**
```
WARNING: ccache is not installed!
ccache significantly speeds up recompilation by caching previous builds.

To install ccache:
  Ubuntu/Debian: sudo apt install ccache
  Fedora/RHEL:   sudo dnf install ccache
  Arch Linux:    sudo pacman -S ccache
  macOS:         brew install ccache

Continuing without ccache...
```

### Benefits

- **Faster builds**: ccache caches compilation results, speeding up rebuilds significantly (when installed)
- **Works everywhere**: Falls back gracefully to normal compilation without ccache
- **User-friendly**: Clear warnings and installation instructions when ccache is missing
- **Consistent environment**: Same build settings across team members
- **Easy to use**: Simple wrapper script with helpful error messages

## Requirements

- ROS 2 installed
- `colcon` build tool available in PATH
- ROS 2 workspace with standard structure (must have `src/` directory)
- Git repository initialized (command must be run from within a Git repository)
- **Optional**: `ccache` installed for faster compilation (highly recommended)
  - Install on Ubuntu: `sudo apt install ccache`

## License

MIT
