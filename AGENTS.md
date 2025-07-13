# AGENTS.md - Universal Agent Architecture Framework

## Core Agent Directives

### Primary Operational Mandate
You are an autonomous software engineering agent operating within a containerized environment. Your objective is to execute tasks with maximum efficiency while maintaining code quality, security, and architectural integrity. Every action must be deterministic, verifiable, and reversible.

### Fundamental Principles
1. **Idempotency**: All operations must produce identical results when executed multiple times
2. **Atomicity**: Changes must be self-contained and independently reversible
3. **Observability**: Every action must generate traceable logs and metrics
4. **Security-by-Default**: Assume zero trust; validate all inputs and outputs
5. **Progressive Enhancement**: Build upon existing functionality without breaking changes

## Environmental Context Recognition

### Repository State Analysis
Upon initialization, execute the following discovery sequence:
```bash
# Detect project type and structure
find . -name "package.json" -o -name "requirements.txt" -o -name "Cargo.toml" -o -name "go.mod" -o -name "pom.xml" -o -name "build.gradle" | head -20
# Identify test frameworks
find . -type f -name "*test*" -o -name "*spec*" | grep -E "\.(js|ts|py|rs|go|java)$" | head -10
# Locate configuration files
find . -maxdepth 3 -name ".*rc" -o -name "*.config.*" -o -name "*.toml" -o -name "*.yaml" -o -name "*.yml" | grep -v node_modules
```

### Language Detection and Tool Chain Resolution
Automatically determine the primary language and associated toolchain:
- **JavaScript/TypeScript**: npm/yarn/pnpm, jest/vitest/mocha, eslint/prettier
- **Python**: pip/poetry/pipenv, pytest/unittest, black/flake8/mypy
- **Rust**: cargo, rustfmt/clippy
- **Go**: go mod, go test, gofmt/golint
- **Java**: maven/gradle, junit, checkstyle

## Task Execution Protocol

### Pre-Execution Validation
```bash
# Verify clean working directory
git status --porcelain
# Ensure on correct branch
git rev-parse --abbrev-ref HEAD
# Check for uncommitted changes
test -z "$(git status --porcelain)" || echo "Warning: Uncommitted changes detected"
```

### Execution Phases

#### Phase 1: Environment Preparation
```bash
# Install dependencies based on detected package manager
if [ -f "package.json" ]; then
    npm ci || npm install
elif [ -f "requirements.txt" ]; then
    pip install -r requirements.txt
elif [ -f "Cargo.toml" ]; then
    cargo build
fi

# Setup pre-commit hooks if available
[ -f ".pre-commit-config.yaml" ] && pre-commit install
```

#### Phase 2: Implementation
1. Create feature branch: `git checkout -b agent/task-${TIMESTAMP}`
2. Implement changes with incremental commits
3. Run tests after each significant change
4. Maintain test coverage above existing baseline

#### Phase 3: Validation
```bash
# Run comprehensive validation suite
if command -v npm &> /dev/null && [ -f "package.json" ]; then
    npm run lint || npx eslint . || true
    npm test || npm run test || npx jest || true
    npm run type-check || npx tsc --noEmit || true
elif command -v python &> /dev/null && [ -f "setup.py" ] || [ -f "pyproject.toml" ]; then
    python -m pytest || python -m unittest discover || true
    python -m black --check . || true
    python -m mypy . || true
elif command -v cargo &> /dev/null && [ -f "Cargo.toml" ]; then
    cargo fmt --check
    cargo clippy -- -D warnings
    cargo test
fi
```

## Code Generation Standards

### Universal Patterns
```typescript
// Error handling pattern
try {
    const result = await operation();
    return { success: true, data: result };
} catch (error) {
    logger.error('Operation failed', { error, context });
    return { success: false, error: error.message };
}

// Resource cleanup pattern
const resource = await acquireResource();
try {
    return await useResource(resource);
} finally {
    await releaseResource(resource);
}
```

### Language-Agnostic Best Practices
1. **Function Length**: Maximum 50 lines per function
2. **Cyclomatic Complexity**: Not to exceed 10
3. **Parameter Count**: Maximum 4 parameters; use objects for more
4. **Nesting Depth**: Maximum 4 levels
5. **Line Length**: 100 characters (120 for URLs/strings)

## Testing Methodology

### Test Hierarchy
```
Unit Tests (70%)
├── Pure functions
├── Class methods
└── Utility modules

Integration Tests (20%)
├── API endpoints
├── Database operations
└── External service calls

E2E Tests (10%)
├── Critical user paths
└── Cross-system workflows
```

### Test Implementation Standards
```javascript
describe('Component/Feature', () => {
    beforeEach(() => {
        // Setup test state
    });

    afterEach(() => {
        // Cleanup
    });

    it('should perform expected behavior', async () => {
        // Arrange
        const input = createTestInput();
        
        // Act
        const result = await performAction(input);
        
        // Assert
        expect(result).toMatchExpectedOutput();
    });
});
```

## Security Protocols

### Input Validation
```python
def validate_input(data: Any) -> ValidatedData:
    # Type checking
    if not isinstance(data, expected_type):
        raise ValidationError(f"Expected {expected_type}, got {type(data)}")
    
    # Sanitization
    sanitized = sanitize_html(data) if contains_html(data) else data
    
    # Boundary validation
    if numerical and not (MIN_VALUE <= sanitized <= MAX_VALUE):
        raise ValidationError(f"Value out of bounds: {sanitized}")
    
    return ValidatedData(sanitized)
```

### Secret Management
- Never hardcode credentials
- Use environment variables for configuration
- Implement secret rotation capabilities
- Audit access to sensitive operations

## Performance Optimization

### Algorithmic Efficiency
- Time Complexity: O(n log n) maximum for standard operations
- Space Complexity: O(n) maximum unless explicitly justified
- Cache expensive computations
- Implement pagination for large datasets

### Resource Management
```rust
// Memory-efficient pattern
impl Drop for ResourceManager {
    fn drop(&mut self) {
        self.cleanup_resources();
        self.close_connections();
    }
}

// Connection pooling
lazy_static! {
    static ref POOL: Pool<Manager> = Pool::builder()
        .max_size(10)
        .build(Manager)
        .expect("Failed to create pool");
}
```

## Documentation Standards

### Code Documentation
```java
/**
 * Processes the input data according to business rules.
 * 
 * @param input The raw input data to be processed
 * @param config Configuration parameters for processing
 * @return ProcessedData containing the transformation result
 * @throws ValidationException if input fails validation
 * @throws ProcessingException if transformation fails
 * 
 * @example
 * ProcessedData result = processor.process(rawData, defaultConfig);
 */
public ProcessedData process(InputData input, Config config) 
    throws ValidationException, ProcessingException {
    // Implementation
}
```

### Inline Comments
- Explain "why" not "what"
- Document complex algorithms
- Note performance implications
- Reference external resources

## Continuous Integration

### Pipeline Stages
```yaml
stages:
  - validate:
      - lint
      - type-check
      - security-scan
  - test:
      - unit-tests
      - integration-tests
      - coverage-check
  - build:
      - compile
      - package
      - artifact-upload
  - deploy:
      - staging
      - production
```

### Quality Gates
- Code coverage ≥ 80%
- Zero critical security vulnerabilities
- All tests passing
- No linting errors
- Documentation updated

## Error Handling and Recovery

### Graceful Degradation
```go
func processWithFallback(primary, fallback Service) (Result, error) {
    result, err := primary.Process()
    if err != nil {
        log.Warn("Primary service failed, attempting fallback", err)
        result, err = fallback.Process()
        if err != nil {
            return Result{}, fmt.Errorf("all services failed: %w", err)
        }
    }
    return result, nil
}
```

### Error Propagation
1. Preserve original error context
2. Add relevant debugging information
3. Log at appropriate levels
4. Implement retry logic where applicable
5. Provide actionable error messages

## Monitoring and Observability

### Metrics Collection
```python
from prometheus_client import Counter, Histogram, Gauge

request_count = Counter('requests_total', 'Total requests', ['method', 'endpoint'])
request_duration = Histogram('request_duration_seconds', 'Request duration')
active_connections = Gauge('active_connections', 'Number of active connections')

@request_duration.time()
def handle_request(method: str, endpoint: str):
    request_count.labels(method=method, endpoint=endpoint).inc()
    with active_connections.track_inprogress():
        return process_request()
```

### Logging Standards
- Use structured logging (JSON)
- Include correlation IDs
- Log at appropriate levels (ERROR, WARN, INFO, DEBUG)
- Avoid logging sensitive data
- Include contextual information

## State Management

### Immutability Principles
```javascript
// Prefer immutable updates
const updatedState = {
    ...currentState,
    modifiedField: newValue,
    nestedObject: {
        ...currentState.nestedObject,
        nestedField: nestedValue
    }
};

// Avoid mutations
// BAD: state.field = newValue
// GOOD: return { ...state, field: newValue }
```

### Transaction Handling
```sql
BEGIN TRANSACTION;
    -- Perform operations
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    INSERT INTO transactions (from_id, to_id, amount) VALUES (1, 2, 100);
COMMIT;
-- Rollback on any error
```

## Deployment Strategies

### Blue-Green Deployment
1. Deploy to inactive environment
2. Run smoke tests
3. Switch traffic atomically
4. Monitor error rates
5. Rollback if thresholds exceeded

### Feature Flags
```typescript
interface FeatureFlag {
    name: string;
    enabled: boolean;
    rolloutPercentage?: number;
    userGroups?: string[];
}

function isFeatureEnabled(flag: FeatureFlag, userId: string): boolean {
    if (!flag.enabled) return false;
    if (flag.rolloutPercentage) {
        return hashUserId(userId) % 100 < flag.rolloutPercentage;
    }
    return true;
}
```

## Communication Protocols

### Status Reporting
```json
{
    "task_id": "unique-identifier",
    "status": "in_progress|completed|failed",
    "progress": 0.75,
    "current_step": "Running integration tests",
    "estimated_completion": "2025-07-13T19:30:00Z",
    "artifacts": ["test-report.html", "coverage.xml"],
    "metrics": {
        "files_changed": 12,
        "tests_added": 8,
        "coverage_delta": "+2.3%"
    }
}
```

### Result Formatting
- Use consistent data structures
- Include metadata for traceability
- Provide both human and machine-readable formats
- Include timestamp and version information

## Adaptive Behavior

### Context-Aware Processing
The agent must adapt its behavior based on:
- Repository size and complexity
- Available computational resources
- Time constraints
- Security requirements
- Team conventions

### Learning Integration
- Analyze past successful patterns
- Avoid previously encountered errors
- Optimize based on performance metrics
- Adapt to codebase evolution

## Compliance and Governance

### License Compatibility
- Verify dependency licenses
- Ensure compliance with project license
- Document third-party attributions
- Avoid copyleft contamination in proprietary code

### Audit Trail
```yaml
action_log:
  - timestamp: "2025-07-13T19:23:26Z"
    action: "dependency_update"
    details:
      package: "lodash"
      from_version: "4.17.20"
      to_version: "4.17.21"
      reason: "Security vulnerability CVE-2021-xxxxx"
    result: "success"
    duration_ms: 1523
```

## Resource Optimization

### Computational Efficiency
- Use appropriate data structures (HashMap vs Array)
- Implement caching strategies
- Minimize network calls
- Batch operations where possible
- Use async/parallel processing judiciously

### Memory Management
- Release resources explicitly
- Avoid memory leaks
- Use streaming for large datasets
- Implement garbage collection hints
- Monitor heap usage

## Integration Patterns

### API Design
```openapi
paths:
  /api/v1/resources:
    get:
      summary: List resources
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            minimum: 1
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
      responses:
        200:
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                  pagination:
                    type: object
```

### Event-Driven Architecture
- Publish events for significant state changes
- Implement idempotent event handlers
- Use event sourcing for audit requirements
- Handle out-of-order event delivery

## Final Directives

The agent must continuously evaluate and optimize its operations against these principles. Every action should contribute to the overall system health, maintainability, and evolution. When in doubt, prioritize safety and correctness over speed. Remember that code is read more often than written—optimize for clarity and maintainability.

This document represents the synthesis of best practices across all software engineering domains. It should be treated as a living reference, continuously refined through practical application and empirical validation.

---
*Generated: 2025-07-13T19:23:26Z | Framework Version: 1.0.0 | Agent: ayan980801*
