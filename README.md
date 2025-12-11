# ROS 2 Workspace Plugin

Claude Code plugin for ROS 2 workspace management with colcon build and test automation.

## Features

- **Automatic workspace detection**: Finds ROS 2 workspace root by traversing up directories
- **Workspace tracking**: Creates `.ros_workspace.txt` with workspace root path
- **Git integration**: Automatically adds generated files to `.gitignore`
- **Colcon build**: Executes `colcon build` with ccache support via `build.sh`
- **Colcon test**: Executes `colcon test` with automatic result display via `test.sh`
- **Test results**: Display detailed test results with `colcon test-result --verbose`
- **Smart package selection**: Automatically builds/tests only packages in current directory
- **Directory preservation**: Always returns to original directory after operations

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

### `/colcon-test` command

Execute colcon test with automatic workspace detection and result display:

```bash
/colcon-test
```

With options:

```bash
/colcon-test --packages-select my_package
/colcon-test --packages-up-to my_package
/colcon-test --retest-until-pass 3
```

### `/colcon-test-result` command

Display detailed test results from the workspace:

```bash
/colcon-test-result
```

With options:

```bash
/colcon-test-result --all
/colcon-test-result --test-result-base custom_results
```

**Note**: This command automatically navigates to the workspace root, displays test results, and returns to your original directory.

### Smart Package Selection

The plugin automatically determines which packages to build/test based on your current directory:

**Scenario 1: Working on specific package**
```bash
# Navigate to package directory
cd ~/ros2_ws/src/my_robot_package

# Start Claude Code
cc

# Build only this package and dependencies
/colcon-build
# Result: colcon build --packages-up-to my_robot_package

# Test only this package and dependencies
/colcon-test
# Result: colcon test --packages-up-to my_robot_package
```

**Scenario 2: Working in workspace root**
```bash
# Navigate to workspace root
cd ~/ros2_ws

# Start Claude Code
cc

# Build entire workspace
/colcon-build
# Result: colcon build

# Test entire workspace
/colcon-test
# Result: colcon test
```

**Scenario 3: Override with explicit package selection**
```bash
# From any directory, you can override
/colcon-build --packages-select specific_package
# Result: Builds only specific_package (not its dependencies)

/colcon-test --packages-select specific_package
# Result: Tests only specific_package (not its dependencies)
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

## Generated Scripts

The plugin automatically generates and updates helper scripts in your Git repository root:

### Build Script (build.sh)

Automatically generated and updated on every `/colcon-build` run with ccache support:

#### Features

- **Smart ccache integration**: Automatically detects and uses ccache if installed
- **Helpful warnings**: Displays installation instructions if ccache is not found
- **Works without ccache**: Falls back to normal gcc/g++ if ccache is unavailable
- **Workspace-aware**: Reads workspace root from `.ros_workspace.txt`
- **Statistics**: Displays ccache statistics after each build (when ccache is used)
- **Standalone usage**: Can be used independently from Claude Code plugin

#### Example usage

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

#### Benefits

- **Faster builds**: ccache caches compilation results, speeding up rebuilds significantly (when installed)
- **Works everywhere**: Falls back gracefully to normal compilation without ccache
- **User-friendly**: Clear warnings and installation instructions when ccache is missing
- **Consistent environment**: Same build settings across team members
- **Easy to use**: Simple wrapper script with helpful error messages

### Test Script (test.sh)

Automatically generated and updated on every `/colcon-test` run:

#### Features

- **Automatic test execution**: Runs `colcon test` with provided arguments
- **Result display**: Automatically shows test results with `colcon test-result --all`
- **Workspace-aware**: Reads workspace root from `.ros_workspace.txt`
- **Error handling**: Validates workspace setup before running tests
- **Standalone usage**: Can be used independently from Claude Code plugin

#### Example usage

```bash
# Auto-generated and updated on every /colcon-test run
./test.sh --packages-up-to my_package

# The script will:
# 1. Verify .ros_workspace.txt exists
# 2. Navigate to workspace root
# 3. Run: colcon test --packages-up-to my_package
# 4. Display test results with colcon test-result --all
```

Example output:
```
Testing ROS 2 workspace...
Workspace: /home/user/ros2_ws

Running: colcon test --packages-up-to my_package

[Test execution...]

Test completed!

Viewing test results...
Summary: 15 tests, 0 errors, 0 failures, 0 skipped
```

#### Benefits

- **Automatic result display**: No need to manually run `colcon test-result`
- **Clear feedback**: See test results immediately after execution
- **Consistent testing**: Same test process across team members
- **Easy to use**: Simple wrapper script with helpful error messages
- **Works everywhere**: Requires only colcon and ROS 2 installation

## Additional Commands

### `/colcon-test-result`

Display detailed test results without re-running tests:

```bash
# From any directory in your Git repository
/colcon-test-result

# The command will:
# 1. Navigate to workspace root (from .ros_workspace.txt)
# 2. Run: colcon test-result --verbose
# 3. Display detailed test results
# 4. Return to your original directory
```

**Key features:**
- **No directory change**: Returns to original directory after displaying results
- **Verbose by default**: Shows detailed test output with `--verbose` flag
- **Flexible options**: Can pass any `colcon test-result` options
- **Quick access**: View test results from anywhere in your repository

**Example output:**
```
Using workspace root from .ros_workspace.txt: /home/user/ros2_ws

build/my_package/test_results/my_package/test_my_node.gtest.xml: 5 tests, 0 errors, 0 failures, 0 skipped
build/sensor_driver/test_results/sensor_driver/test_driver.gtest.xml: 10 tests, 0 errors, 0 failures, 0 skipped

Summary: 15 tests, 0 errors, 0 failures, 0 skipped

Returned to original directory: /home/user/ros2_ws/src/my_package
```

## Requirements

- ROS 2 installed
- `colcon` build tool available in PATH
- ROS 2 workspace with standard structure (must have `src/` directory)
- Git repository initialized (command must be run from within a Git repository)
- **Optional**: `ccache` installed for faster compilation (highly recommended)
  - Install on Ubuntu: `sudo apt install ccache`

## License

MIT
