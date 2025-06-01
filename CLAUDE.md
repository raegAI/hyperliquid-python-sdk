# Claude Code Configuration for Hyperliquid Python SDK

## Project Overview
The Hyperliquid Python SDK is a Python library for interacting with the Hyperliquid decentralized exchange API. It provides a comprehensive interface for trading operations, including order management, account information retrieval, and WebSocket-based real-time data streaming.

## Code Style Guidelines
- **Python Version**: Use Python 3.9+ for runtime compatibility, Python 3.10 for development
- **Type Hints**: All functions and methods MUST have complete type annotations using the typing module
- **Import Style**: Use absolute imports and group them according to isort configuration (FUTURE, TYPING, STDLIB, THIRDPARTY, FIRSTPARTY, LOCALFOLDER)
- **Line Length**: Maximum 120 characters per line
- **Formatting**: Code is formatted with Black (line-length=120)
- **Docstrings**: While not required everywhere, add clear docstrings for all public API methods
- **TypedDict Usage**: Use TypedDict for complex data structures (as seen in utils/types.py)
- **Constants**: Define API URLs and other constants in utils/constants.py

## Development Workflow
- **Setup**: Run `make install` to install all dependencies via Poetry
- **Testing**: Run `make test` to execute the test suite with pytest
- **Linting**: Run `make lint` or `make pre-commit` to run all linters and formatters
- **Type Checking**: Use mypy with strict settings as defined in pyproject.toml
- **Safety Checks**: Run `make check-safety` to verify dependency security
- **Lock File Updates**: Use `make lockfile-update` for incremental updates, `make lockfile-update-full` for full regeneration

## Review Criteria
- **API Compatibility**: Never break existing public API methods or signatures
- **Type Safety**: Ensure all new code has proper type hints that pass mypy checks
- **Error Handling**: All API calls must have proper error handling using the custom ClientError and ServerError exceptions
- **Security**: Never log or expose private keys, API secrets, or sensitive user data
- **Rate Limiting**: Consider rate limits when implementing features that make multiple API calls
- **WebSocket Management**: Properly handle WebSocket lifecycle (connection, reconnection, cleanup)
- **Decimal Precision**: Use appropriate decimal handling for financial calculations

## Project-Specific Rules
- **API Structure**: The codebase follows a clear separation: `Info` class for read operations, `Exchange` class for write operations
- **Signing Pattern**: All trading operations require proper message signing using eth_account
- **Configuration**: Examples use config.json for credentials - never commit actual credentials
- **Network Support**: Support both MAINNET and TESTNET with proper URL configuration
- **Agent/API Wallet**: Support both direct wallet usage and agent wallet delegation
- **Error Classes**: Use ClientError for 4xx errors and ServerError for 5xx errors
- **Constants Usage**: Always use constants from utils/constants.py for API URLs

## Preferred Patterns
- **Type Definitions**: Define complex types in utils/types.py using TypedDict
- **WebSocket Subscriptions**: Use the Subscription union type pattern for different subscription types
- **Order Types**: Use the OrderRequest, OrderWire pattern for order serialization
- **Async Handling**: WebSocket manager runs in a separate thread with proper synchronization
- **Example Structure**: Examples should use example_utils.setup() for consistent initialization
- **Error Response**: Return {"status": "err", "response": error_details} for errors
- **Success Response**: Return {"status": "ok", "response": result} for successful operations

## Testing Requirements
- **Test Framework**: Use pytest with VCR.py for recording HTTP interactions
- **Test Data**: Use recorded cassettes in tests/cassettes/ for reproducible tests
- **Test Coverage**: Run tests with coverage reporting (`--cov=hyperliquid`)
- **Mocking**: Use pytest.mark.vcr() decorator for tests that make API calls
- **Test Constants**: Define test-specific constants (TEST_META, TEST_SPOT_META) at module level
- **Assertions**: Use specific assertions on response structure and values
- **No Network Calls**: Tests should not make actual network calls in CI

## Documentation Standards
- **README**: Keep README.md updated with installation instructions and basic usage
- **Examples**: Provide working examples in the examples/ directory
- **Type Information**: Let type hints serve as inline documentation
- **API Changes**: Document any breaking changes in release notes
- **Config Example**: Maintain config.json.example with clear instructions
- **Error Messages**: Provide clear, actionable error messages

## Integration Guidelines
- **Poetry**: This project uses Poetry for dependency management - always use Poetry commands
- **Pre-commit Hooks**: The project uses pre-commit hooks - ensure they pass before commits
- **Semantic Versioning**: Follow semver for version updates (poetry version major/minor/patch)
- **Release Process**: Use GitHub releases with proper categorization (see README labels)
- **CI/CD**: Ensure all tests and linters pass in the CI pipeline

## Common Patterns to Follow
```python
# Type imports at the top
from hyperliquid.utils.types import Any, Optional, TypedDict, Union

# Class initialization pattern
class MyClass(API):
    def __init__(self, base_url: Optional[str] = None):
        super().__init__(base_url)
        
# Error handling pattern
try:
    response = self.session.post(url, json=payload)
    self._handle_exception(response)
    return response.json()
except ValueError:
    return {"error": f"Could not parse JSON: {response.text}"}

# TypedDict pattern for complex types
MyType = TypedDict("MyType", {"field1": str, "field2": int, "field3": Optional[float]})
```

## Commands to Run
When making changes, always run these commands:
1. `make install` - Install dependencies
2. `make lint` - Run all linters and formatters
3. `make test` - Run the test suite
4. `make check-safety` - Check for security vulnerabilities

## Important Files and Directories
- `hyperliquid/` - Main package directory
  - `api.py` - Base API class with request handling
  - `info.py` - Read-only operations (market data, user info)
  - `exchange.py` - Write operations (orders, transfers)
  - `websocket_manager.py` - WebSocket subscription handling
  - `utils/` - Utility modules
    - `constants.py` - API URLs and constants
    - `types.py` - TypedDict definitions
    - `signing.py` - Message signing utilities
    - `error.py` - Custom exception classes
- `examples/` - Usage examples (must remain functional)
- `tests/` - Test suite with VCR cassettes