# Changelog - DevLocal Branch

Local development changes not yet merged to main.

## [Unreleased] - 2025-12-23

### Added - install.py (root)

- **`--prefer-uv` command-line argument** - Users can now explicitly prefer uv package manager over pip when both are available
- **`_get_installer_cmd()` helper function** - Utility function that returns the best available installer command based on availability and user preferences
- **`PREFER_UV` global flag** - Respected throughout the installation process when `--prefer-uv` is passed
- **Improved uv detection messaging** - Clearer console output about which installer is being used and why

### Changed - install.py (root)

- **`_setup_installer_command()`** - Now respects the PREFER_UV flag and provides clearer messaging about installer selection
- **`_install_with_onnx()`** - Now uses the passed `installer_cmd` instead of hardcoded pip for dependency installation

### Fixed - install.py (root)

- Fixed all Pylance type errors (13+ issues resolved)

### Fixed - scripts/installation/install.py

- Fixed all Pylance type errors (20+ issues resolved)
