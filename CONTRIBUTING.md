# Contributing to Live Differ

Thank you for your interest in contributing to Live Differ! This document provides comprehensive guidelines and instructions for contributing to the project.

## Table of Contents
- [Development Setup](#development-setup)
  - [Prerequisites](#prerequisites)
  - [Setting Up Development Environment](#setting-up-development-environment)
  - [Running the Differ](#running-the-differ)
- [Project Structure](#project-structure)
- [Development Workflow](#development-workflow)
- [Release Process](#release-process)
- [Code Style Guidelines](#code-style-guidelines)
- [Testing Guidelines](#testing-guidelines)
- [Documentation Guidelines](#documentation-guidelines)
- [Building and Publishing](#building-and-publishing)
  - [Prerequisites for Publishing](#prerequisites-for-publishing)
  - [Building the Package](#building-the-package)
  - [Publishing to TestPyPI](#publishing-to-testpypi)
  - [Publishing to Production PyPI](#publishing-to-production-pypi)
  - [Version Management](#version-management)
  - [Version Numbering Guidelines](#version-numbering-guidelines)
  - [Troubleshooting Publishing](#troubleshooting-publishing)
  - [Security Notes](#security-notes)
- [Error Handling](#error-handling)
- [Logging Guidelines](#logging-guidelines)
- [Pull Request Process](#pull-request-process)

## Development Setup

### Prerequisites
- Python 3.7 or higher
- pip (Python package installer)
- Git

### Setting Up Development Environment

1. Fork the repository on GitHub

2. Clone your fork:
```bash
git clone https://github.com/YOUR_USERNAME/live_differ.git
cd live_differ
```

3. Add the upstream repository:
```bash
git remote add upstream https://github.com/manthanby/live_differ.git
```

4. Create and activate a virtual environment:
```bash
python -m venv .venv
source .venv/bin/activate  # On Unix/macOS
# or
.venv\Scripts\activate     # On Windows
```

5. Install development dependencies:
```bash
pip install -e .
pip install build twine pytest pytest-cov black isort flake8
```

### Running the Differ

There are several ways to run the Live Differ:

1. **Using the CLI command** (after installation):
   ```bash
   live-differ file1.txt file2.txt
   ```

2. **Using Python module directly**:
   ```bash
   python -m live_differ file1.txt file2.txt
   ```

3. **Using Python code**:
   ```python
   from live_differ.cli import cli
   from live_differ.modules.differ import FileDiffer
   
   # Option 1: Using the CLI programmatically
   cli(["file1.txt", "file2.txt"])
   
   # Option 2: Using the Differ class directly
   differ = FileDiffer("file1.txt", "file2.txt")
   diff_data = differ.get_diff()
   ```

4. **Additional Options**:
   ```bash
   # Custom host and port
   live-differ file1.txt file2.txt --host 0.0.0.0 --port 8000
   
   # Debug mode
   live-differ file1.txt file2.txt --debug
   
   # Using environment variables
   FLASK_HOST=0.0.0.0 FLASK_PORT=8000 live-differ file1.txt file2.txt
   ```

## Project Structure

```
live_differ/
├── live_differ/          # Main package directory
│   ├── __init__.py      # Package initialization
│   ├── __main__.py      # Entry point for running as module
│   ├── cli.py           # Command-line interface
│   ├── core.py          # Core application logic
│   ├── modules/         # Core modules
│   │   ├── __init__.py
│   │   ├── differ.py    # File diffing logic
│   │   └── watcher.py   # File change monitoring
│   ├── static/          # Static web assets
│   │   ├── css/        # Stylesheets
│   │   └── js/         # JavaScript files
│   ├── templates/       # HTML templates
│   └── assets/          # Project assets
│       └── images/      # Images for documentation
├── tests/               # Test directory
│   ├── __init__.py
│   ├── test_differ.py
│   └── test_watcher.py
├── pyproject.toml       # Project configuration
├── MANIFEST.in         # Package manifest
├── README.md          # User documentation
├── CONTRIBUTING.md    # Contributor documentation
└── LICENSE           # License information
```

### Key Components

1. **Core Application (core.py)**
   - Flask application setup
   - Route definitions
   - WebSocket handling
   - Error handling

2. **Command Line Interface (cli.py)**
   - Command-line argument parsing
   - Environment variable handling
   - Application entry point

3. **Differ Module (modules/differ.py)**
   - File comparison logic
   - Difference calculation
   - Result formatting

4. **Watcher Module (modules/watcher.py)**
   - File system monitoring
   - Change detection
   - Event handling

## Release Process

### Overview
Live Differ uses GitHub Actions for automated releases. When you push a tag starting with 'v', it automatically:
1. Builds the Python package
2. Generates a changelog
3. Creates a GitHub release
4. Publishes to PyPI

### Prerequisites for Release
1. Ensure all tests pass: `pytest tests/`

### Using the Release Script

The project includes a release script (`scripts/release.py`) to help automate the version bumping process. The script:
- Checks if there are any pending git changes
- Verifies if your branch is up to date with remote
- Updates the version in pyproject.toml
- Provides the necessary git commands to complete the release

To use the release script:

1. Ensure you're in the project root directory
2. Run the script with one of the following commands:
   ```bash
   # For a patch version bump (0.0.X)
   python scripts/release.py

   # For a minor version bump (0.X.0)
   python scripts/release.py --bump minor

   # For a major version bump (X.0.0)
   python scripts/release.py --bump major

   # To see what would happen without making changes
   python scripts/release.py --dry-run
   ```

3. If the script runs successfully, it will output a series of git commands. Execute these commands to complete the release process.

Note: The script will fail if you have uncommitted changes or if your branch is not up to date with the remote repository.

### Release Process

The complete release process involves:
1. Running the release script to bump version numbers
2. Pushing the changes and tags to GitHub
3. Update version in `pyproject.toml`:
   ```toml
   [project]
   version = "X.Y.Z"  # Update this version
   ```

4. Commit your changes:
   ```bash
   git add pyproject.toml
   git commit -m "chore: bump version to vX.Y.Z"
   ```

5. Create and push a tag:
   ```bash
   git tag -a vX.Y.Z -m "Release vX.Y.Z"
   git push origin vX.Y.Z
   ```

6. Monitor the release:
   - Check GitHub Actions for the release workflow status
   - Verify the release appears on GitHub Releases
   - Confirm the package is available on PyPI

### Version Numbering
We follow Semantic Versioning (SemVer):
- MAJOR version (X) for incompatible API changes
- MINOR version (Y) for new functionality in a backward compatible manner
- PATCH version (Z) for backward compatible bug fixes

### Commit Convention
Use conventional commits to get better changelogs:
- `feat:` for new features
- `fix:` for bug fixes
- `docs:` for documentation changes
- `test:` for test changes
- `chore:` for maintenance tasks

### Troubleshooting Releases
- If the release workflow fails, check the GitHub Actions logs
- For PyPI upload issues, verify your API token is correctly set
- If you need to redo a release, delete the tag locally and remotely:
  ```bash
  git tag -d vX.Y.Z
  git push --delete origin vX.Y.Z
  ```

## Development Workflow

### 1. Creating a New Feature

1. Update your main branch:
```bash
git checkout main
git pull upstream main
```

2. Create a feature branch:
```bash
git checkout -b feature/your-feature-name
```

3. Make your changes following our coding standards

4. Test your changes:
```bash
pytest tests/
```

### 2. Code Quality Checks

Run these before committing:

```bash
# Format code
black live_differ tests
isort live_differ tests

# Check style
flake8 live_differ tests

# Run tests with coverage
pytest --cov=live_differ tests/
```

### 3. Committing Changes

Follow these commit message guidelines:
- Use present tense ("Add feature" not "Added feature")
- Use imperative mood ("Move cursor to..." not "Moves cursor to...")
- Limit first line to 72 characters
- Reference issues and pull requests

Example:
```bash
git commit -m "Add real-time update feature

- Implement WebSocket connection
- Add file change detection
- Update UI automatically
- Add error handling

Fixes #123"
```

## Code Style Guidelines

### Python Style
- Follow [PEP 8](https://pep8.org/)
- Use 4 spaces for indentation
- Maximum line length: 88 characters (Black default)
- Use meaningful variable and function names

### Documentation Style
- Use Google-style docstrings
- Document all public functions, classes, and methods
- Include type hints

Example:
```python
def compare_files(file1: str, file2: str) -> Dict[str, Any]:
    """Compare two files and return their differences.

    Args:
        file1 (str): Path to the first file
        file2 (str): Path to the second file

    Returns:
        Dict[str, Any]: Dictionary containing:
            - 'differences': List of line differences
            - 'metadata': Dict with file information

    Raises:
        FileNotFoundError: If either file doesn't exist
        PermissionError: If files can't be read
    """
```

## Testing Guidelines

### Test Structure
- Use pytest for testing
- Organize tests by module
- Use fixtures for common setup
- Include unit and integration tests

### Writing Tests
```python
def test_file_differ():
    """Test file comparison functionality."""
    # Arrange
    differ = FileDiffer("test1.txt", "test2.txt")
    
    # Act
    result = differ.compare()
    
    # Assert
    assert result is not None
    assert "differences" in result
```

### Running Tests
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_differ.py

# Run with coverage
pytest --cov=live_differ --cov-report=html
```

## Building and Publishing

### Prerequisites for Publishing
- Create accounts on [PyPI](https://pypi.org) and [TestPyPI](https://test.pypi.org)
- Install required tools:
  ```bash
  pip install build twine
  ```

### Building the Package
1. Clean previous builds:
   ```bash
   rm -rf dist/ build/ *.egg-info/
   ```

2. Build the package:
   ```bash
   python -m build
   ```
   This will create both wheel (.whl) and source (.tar.gz) distributions in the `dist` directory.

### Publishing to TestPyPI
1. First, ensure your package works on TestPyPI:
   ```bash
   python -m twine upload --repository testpypi dist/*
   ```

2. Test the installation from TestPyPI:
   ```bash
   pip install --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ live-differ
   ```

3. Verify the package works correctly:
   ```bash
   live-differ --help
   ```

### Publishing to Production PyPI
Once you've verified everything works on TestPyPI:

1. Upload to PyPI:
   ```bash
   python -m twine upload dist/*
   ```

2. Test the installation from PyPI:
   ```bash
   pip install live-differ
   ```

3. Verify the package works correctly:
   ```bash
   live-differ --help
   ```

### Version Management
1. Update version in `pyproject.toml`
2. Create a git tag:
   ```bash
   git tag v0.1.0  # Replace with your version
   git push origin v0.1.0
   ```

### Version Numbering Guidelines
1. **Production Versions**
   - Follow semantic versioning (MAJOR.MINOR.PATCH)
     - MAJOR: Breaking changes
     - MINOR: New features (backward compatible)
     - PATCH: Bug fixes (backward compatible)
   - Example: `1.2.3`

2. **Development Versions**
   - For testing on TestPyPI, use development versions:
     - Development suffix: `0.1.0.dev1`, `0.1.0.dev2`
     - Local version identifiers: `0.1.0+dev1`, `0.1.0+dev2`
   - Reset development numbers when releasing a new version

3. **Version Immutability**
   - PyPI and TestPyPI do not allow overwriting existing versions
   - Once uploaded, a version number cannot be reused
   - This is a security feature to protect against supply chain attacks
   - For TestPyPI only:
     - You can delete the entire project through the web interface
     - Must wait 24 hours before reusing deleted version numbers
     - Not recommended for production PyPI

4. **Best Practices**
   - Never delete versions from production PyPI
   - Always increment version numbers for new uploads
   - Use development versions for testing
   - Keep a changelog of version changes
   - Tag releases in git to match PyPI versions

### Troubleshooting Publishing
- If you get a version conflict error, ensure you've updated the version in `pyproject.toml`
- If you get a file exists error on PyPI, you cannot reupload the same version. You must increment the version number
- For authentication issues, use an API token from your PyPI account settings instead of password

### Security Notes
- Never commit PyPI credentials to the repository
- Use API tokens instead of passwords
- Store credentials in `~/.pypirc` or use environment variables

## Error Handling

### Guidelines
1. Use custom exceptions for specific errors
2. Provide clear error messages
3. Include context in exceptions
4. Log errors appropriately
5. Handle edge cases

Example:
```python
class DifferError(Exception):
    """Base exception for differ-related errors."""
    pass

class FileComparisonError(DifferError):
    """Raised when file comparison fails."""
    pass

def compare_files(file1: str, file2: str) -> Dict[str, Any]:
    try:
        # Comparison logic
    except OSError as e:
        raise FileComparisonError(f"Failed to compare files: {e}")
```

## Logging Guidelines

### Log Levels
- DEBUG: Detailed information for debugging
- INFO: General operational events
- WARNING: Unexpected but handled events
- ERROR: Serious issues that need attention
- CRITICAL: System-level failures

### Logging Example
```python
import logging

logger = logging.getLogger(__name__)

def process_file(filename: str) -> None:
    logger.debug("Starting file processing: %s", filename)
    try:
        # Processing logic
        logger.info("Successfully processed file: %s", filename)
    except Exception as e:
        logger.error("Failed to process file: %s", str(e))
        raise
```

## Pull Request Process

1. **Before Submitting**
   - Update your branch with main
   - Run all tests and style checks
   - Update documentation if needed
   - Add tests for new features

2. **PR Description**
   - Clearly describe the changes
   - Link related issues
   - Include screenshots for UI changes
   - List any breaking changes

3. **Review Process**
   - Address reviewer comments
   - Keep the PR focused and small
   - Be responsive to feedback

4. **After Merging**
   - Delete your feature branch
   - Update your local main branch

## Questions or Need Help?

- Open an issue for bugs or feature requests
- Join our discussions for questions
- Tag maintainers for urgent issues

## License

By contributing, you agree that your contributions will be licensed under the project's MIT License.
