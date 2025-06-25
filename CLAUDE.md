# Google Apps Script Documentation Navigation Guide

## Documentation Structure & When to Reference

This project contains comprehensive Google Apps Script documentation organized by topic. Use this guide to know which documentation to reference for different scenarios.

### Quick Reference Map

| User Query About | Primary Documentation | Secondary References |
|-----------------|----------------------|---------------------|
| Script won't run as expected | `docs/core-concepts/gas-nuances-gotchas.md` | `docs/troubleshooting/common-errors.md` |
| Authorization/permission errors | `docs/troubleshooting/common-errors.md` | `docs/core-concepts/gas-nuances-gotchas.md` |
| Script timeout issues | `docs/troubleshooting/performance-guide.md` | `docs/troubleshooting/quota-limits.md` |
| How to install/use scripts | `docs/installation-usage-guide.md` | - |
| API quota exceeded | `docs/troubleshooting/quota-limits.md` | `docs/troubleshooting/common-errors.md` |
| Best practices/patterns | `docs/patterns/architecture.md` | `docs/patterns/debugging.md` |
| Service-specific issues | `docs/services/[service-name].md` | `docs/examples/[service]-examples.md` |
| Trigger problems | `docs/core-concepts/triggers.md` | `docs/core-concepts/gas-nuances-gotchas.md` |
| Data handling issues | `docs/core-concepts/data-types.md` | `docs/core-concepts/state-management.md` |
| Debugging help | `docs/patterns/debugging.md` | `docs/troubleshooting/common-errors.md` |

### Core Documentation Files

#### 1. **Platform Quirks & Limitations**
**File**: `docs/core-concepts/gas-nuances-gotchas.md`
**When to use**: 
- Unexpected script behavior
- Platform-specific limitations
- "Why doesn't this work?" questions
- First-time GAS developers

**Key topics**:
- Execution context issues
- Time limits (6-minute timeout)
- Authorization scope limitations
- UI constraints
- Data type conversions

#### 2. **Installation & Usage Guide**
**File**: `docs/installation-usage-guide.md`
**When to use**:
- "How do I run this script?"
- First-time setup questions
- Authorization help
- Deployment guidance

**Key topics**:
- Step-by-step installation
- Running scripts manually/automatically
- Common setup issues
- Best practices

#### 3. **Common Errors Reference**
**File**: `docs/troubleshooting/common-errors.md`
**When to use**:
- Specific error messages
- Permission denied errors
- Type errors
- API failures

**Key topics**:
- Authorization errors
- Quota errors
- Timeout handling
- Type conversions

#### 4. **Performance Optimization**
**File**: `docs/troubleshooting/performance-guide.md`
**When to use**:
- Slow script execution
- Timeout prevention
- Large data processing
- Optimization strategies

#### 5. **Service Quotas & Limits**
**File**: `docs/troubleshooting/quota-limits.md`
**When to use**:
- "Service invoked too many times"
- Rate limiting issues
- Planning high-volume operations

### Service-Specific Documentation

#### Google Services
Located in `docs/services/`:
- `calendar.md` - Calendar API operations
- `docs.md` - Document manipulation
- `drive.md` - File/folder management
- `gmail.md` - Email operations
- `sheets.md` - Spreadsheet operations
- `slides.md` - Presentation management

**When to use**: Service-specific API questions, method references, examples

#### Examples
Located in `docs/examples/`:
- Practical code examples for each service
- Common use case implementations
- Integration patterns

### Core Concepts

Located in `docs/core-concepts/`:
- `triggers.md` - Event-driven execution
- `advanced-services.md` - External API integration
- `data-types.md` - Type handling in GAS
- `event-objects.md` - Trigger event parameters
- `state-management.md` - Persistence strategies
- `error-handling.md` - Error management patterns

### Architecture & Patterns

Located in `docs/patterns/`:
- `architecture.md` - Design patterns for GAS
- `debugging.md` - Testing and debugging strategies

## Navigation Logic for Claude

### When user reports an error:
1. Check error type → `common-errors.md`
2. If behavior is unexpected → `gas-nuances-gotchas.md`
3. If performance-related → `performance-guide.md`

### When user asks "how to":
1. For installation/setup → `installation-usage-guide.md`
2. For specific service → `services/[service].md` + `examples/`
3. For best practices → `patterns/architecture.md`

### When user has timeout/quota issues:
1. Primary → `quota-limits.md`
2. Secondary → `performance-guide.md`
3. For workarounds → `gas-nuances-gotchas.md`

### When debugging:
1. Start with → `patterns/debugging.md`
2. For specific errors → `common-errors.md`
3. For platform quirks → `gas-nuances-gotchas.md`

## Key Information Locations

### Critical Warnings
- 6-minute timeout: `gas-nuances-gotchas.md` → "Time & Performance Limitations"
- Authorization scopes: `gas-nuances-gotchas.md` → "Execution Context & Scope"
- Quota limits: `quota-limits.md` → Service-specific sections

### Code Patterns
- Safe file access: `common-errors.md` → "Authorization Errors"
- Batch processing: `performance-guide.md` → "Batch Operations"
- Error handling: `error-handling.md` → "Error Recovery Patterns"

### Quick Fixes
- Type conversion issues: `gas-nuances-gotchas.md` → "Data Type Conversions"
- Range errors: `common-errors.md` → "Range and Sheet Errors"
- API retries: `common-errors.md` → "API and External Service Errors"

## Documentation Philosophy

1. **Start with the problem** - Identify what the user is trying to solve
2. **Check for platform quirks** - Many issues stem from GAS limitations
3. **Provide working code** - Include tested examples from the docs
4. **Explain the why** - Help users understand GAS's unique behavior
5. **Offer alternatives** - When hitting platform limits, suggest workarounds

Remember: Most Google Apps Script frustrations come from undocumented platform behaviors. When in doubt, check `gas-nuances-gotchas.md` first.