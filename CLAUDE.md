# Claude Code Configuration

This file contains project-specific instructions and guidelines for Claude Code automation.

## Overview

This is a Python SDK for the Hyperliquid API trading platform. Claude Code should follow the existing patterns and maintain the high quality standards of this codebase.

For detailed information about using Claude Code automation, see our [Automation Guide](docs/AUTOMATION_GUIDE.md).

## Project Setup

When working on this project, Claude should:

1. Use Poetry for dependency management (Python 3.10 required)
2. Run `make install` to install dependencies
3. Run `make test` for testing
4. Run `make lint` for code quality checks
5. Test examples to ensure backward compatibility

## Code Style Guidelines

### Python Code
- Follow PEP 8 and use Black for formatting
- Use type hints for all function signatures
- Maintain existing code patterns and conventions
- Write docstrings for all public functions and classes
- Keep functions focused and single-purpose

### Testing
- Write tests for all new functionality
- Maintain or improve test coverage (minimum 80%)
- Use pytest for testing
- Follow existing test patterns in the `tests/` directory
- Test edge cases and error conditions

### Documentation
- Update docstrings when modifying functions
- Update README.md for significant changes
- Add examples for new features in `examples/` directory
- Keep API documentation up to date

## Development Guidelines

### When implementing features:
1. Study existing code patterns first
2. Maintain backward compatibility
3. Follow the existing module structure
4. Use appropriate error handling from `hyperliquid.utils.error`
5. Add comprehensive tests

### When fixing bugs:
1. Write a test that reproduces the bug first
2. Implement the minimal fix
3. Ensure no regressions in existing functionality
4. Update relevant documentation

### Security Considerations
- Never commit API keys or secrets
- Use `config.json.example` as reference
- Validate all user inputs
- Follow secure coding practices

## File Structure

```
hyperliquid/
├── __init__.py          # Main package exports
├── api.py               # Base API functionality
├── exchange.py          # Exchange operations
├── info.py              # Information queries
├── utils/               # Utility modules
│   ├── constants.py     # API constants
│   ├── error.py         # Error handling
│   ├── signing.py       # Cryptographic signing
│   └── types.py         # Type definitions
└── websocket_manager.py # WebSocket functionality
```

## Restrictions

Claude should NOT:
- Modify the core signing logic without explicit approval
- Change the API interface in breaking ways
- Add dependencies without justification
- Modify GitHub Actions workflows
- Access or modify user secrets

## Testing Commands

Always run these before marking work as complete:
- `make lint` - Run all linters and formatters
- `make test` - Run the test suite
- `poetry run python examples/basic_order.py` - Test a basic example

## Common Patterns

### API Calls
```python
from hyperliquid.info import Info
from hyperliquid.utils import constants

info = Info(constants.TESTNET_API_URL, skip_ws=True)
```

### Error Handling
```python
from hyperliquid.utils.error import HyperliquidException

try:
    # API operation
except HyperliquidException as e:
    # Handle specific error
```

### Type Definitions
```python
from typing import Dict, List, Optional
from hyperliquid.utils.types import UserState
```

## Pull Request Guidelines

When Claude creates a PR, it should:
1. Have a clear, descriptive title
2. Include a summary of changes
3. Reference the related issue
4. List any breaking changes
5. Show test results summary

## Additional Resources

- [Hyperliquid API Documentation](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api)
- [Project README](README.md)
- [Automation Guide](docs/AUTOMATION_GUIDE.md)
- [Example Scripts](examples/)

---

*For questions about Claude Code automation, refer to the [Automation Guide](docs/AUTOMATION_GUIDE.md).*