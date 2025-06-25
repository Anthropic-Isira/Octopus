# Google Apps Script API Reference for MCP Server

This documentation provides a comprehensive reference for using Google Apps Script services through the MCP (Model Context Protocol) server. It enables Claude to interact with Google Workspace applications including Calendar, Docs, Drive, Gmail, Sheets, and Slides.

## Quick Navigation

| Service | Description | Documentation | Examples |
|---------|-------------|---------------|----------|
| [Calendar](services/calendar.md) | Access and modify Google Calendar | [API Reference](services/calendar.md) | [Examples](examples/calendar-examples.md) |
| [Docs](services/docs.md) | Create and edit Google Docs | [API Reference](services/docs.md) | [Examples](examples/docs-examples.md) |
| [Drive](services/drive.md) | Manage files and folders in Google Drive | [API Reference](services/drive.md) | [Examples](examples/drive-examples.md) |
| [Gmail](services/gmail.md) | Access Gmail threads, messages, and labels | [API Reference](services/gmail.md) | [Examples](examples/gmail-examples.md) |
| [Sheets](services/sheets.md) | Create and modify Google Sheets | [API Reference](services/sheets.md) | [Examples](examples/sheets-examples.md) |
| [Slides](services/slides.md) | Create and edit Google Slides presentations | [API Reference](services/slides.md) | [Examples](examples/slides-examples.md) |

## Overview

Google Apps Script is a cloud-based JavaScript platform that provides easy ways to integrate with and automate tasks across Google products. This MCP server implementation allows Claude to:

- Create, read, update, and delete documents across all Google Workspace apps
- Automate workflows between different Google services
- Execute scripts with proper authentication and permissions
- Handle complex operations like batch updates, formatting, and data manipulation

## Common Patterns

### Service Access Pattern
All Google Apps Script services follow a consistent access pattern:
```javascript
ServiceName.methodName(parameters)
```

For example:
- `CalendarApp.createEvent(title, startTime, endTime)`
- `SpreadsheetApp.create(name)`
- `DriveApp.getFileById(id)`

### Authentication
The MCP server handles authentication automatically. Ensure you have:
1. Proper Google account permissions
2. OAuth scopes configured for the services you need
3. Access to the target documents/calendars/etc.

### Error Handling
All methods can throw errors. Common error types include:
- Permission denied
- Resource not found
- Invalid parameters
- Quota exceeded

## Quick Reference

### Most Common Operations

#### Calendar
```javascript
// Create an event
CalendarApp.createEvent('Meeting', new Date('2024-01-15 10:00'), new Date('2024-01-15 11:00'))

// Get events for a day
var events = CalendarApp.getEventsForDay(new Date())
```

#### Docs
```javascript
// Create a document
var doc = DocumentApp.create('New Document')

// Add content
doc.getBody().appendParagraph('Hello World')
```

#### Drive
```javascript
// Create a file
DriveApp.createFile('test.txt', 'File content')

// Search files
var files = DriveApp.searchFiles('title contains "Report"')
```

#### Gmail
```javascript
// Send email
GmailApp.sendEmail('recipient@example.com', 'Subject', 'Body')

// Search messages
var threads = GmailApp.search('is:unread')
```

#### Sheets
```javascript
// Create spreadsheet
var ss = SpreadsheetApp.create('New Sheet')

// Get and set values
var range = ss.getRange('A1:B10')
range.setValues([[1, 2], [3, 4]])
```

#### Slides
```javascript
// Create presentation
var pres = SlidesApp.create('New Presentation')

// Add a slide
var slide = pres.appendSlide(SlidesApp.PredefinedLayout.TITLE_AND_BODY)
```

## Best Practices

1. **Batch Operations**: Use batch methods when possible (e.g., `setValues()` instead of multiple `setValue()` calls)
2. **Error Handling**: Always include try-catch blocks for operations that might fail
3. **Performance**: Use `SpreadsheetApp.flush()` to force pending changes in Sheets
4. **Permissions**: Check user permissions before attempting operations
5. **Rate Limits**: Be aware of Google's API quotas and rate limits

## Service-Specific Notes

### Calendar
- All times are in the script's time zone unless specified
- Recurring events use RFC 5545 RRULE format
- Event IDs are unique across all calendars

### Docs
- Document structure is hierarchical (Body > Paragraph > Text)
- Formatting is applied through attributes
- Tables, images, and other elements are treated as child elements

### Drive
- File IDs are permanent and unique
- Folders are treated as special file types
- Shared drives require additional permissions

### Gmail
- Threads contain one or more messages
- Labels are user-specific
- Draft management is separate from sent messages

### Sheets
- Cell references use A1 notation
- Formulas start with `=`
- Named ranges provide easier reference management

### Slides
- Presentations contain slides with layouts
- Elements are positioned using points (1/72 inch)
- Master slides control default formatting

## Getting Started

1. **[Read the Installation & Usage Guide](installation-usage-guide.md)** - Step-by-step instructions for running scripts
2. Review the service-specific documentation for detailed API references
3. Check the examples directory for common use cases
4. Start with simple operations and build complexity gradually
5. Use the MCP server's error responses to debug issues

## Core Concepts

Essential concepts for working with Google Apps Script:

| Topic | Description | Documentation |
|-------|-------------|---------------|
| [Triggers](core-concepts/triggers.md) | Event-driven execution (onEdit, onOpen, time-based) | [Learn More](core-concepts/triggers.md) |
| [Advanced Services](core-concepts/advanced-services.md) | Using Google REST APIs and external services | [Learn More](core-concepts/advanced-services.md) |
| [Performance Limits](core-concepts/performance-limits.md) | Execution time limits, quotas, and optimization | [Learn More](core-concepts/performance-limits.md) |
| [Data Types](core-concepts/data-types.md) | JavaScript types, dates, arrays, and conversions | [Learn More](core-concepts/data-types.md) |
| [Event Objects](core-concepts/event-objects.md) | Understanding trigger event parameters | [Learn More](core-concepts/event-objects.md) |
| [State Management](core-concepts/state-management.md) | Properties Service, Cache Service, Lock Service | [Learn More](core-concepts/state-management.md) |
| [Error Handling](core-concepts/error-handling.md) | Error types, recovery strategies, and patterns | [Learn More](core-concepts/error-handling.md) |
| **[GAS Nuances & Gotchas](core-concepts/gas-nuances-gotchas.md)** | **Critical platform quirks and limitations** | **[Must Read!](core-concepts/gas-nuances-gotchas.md)** |

## Patterns & Architecture

Best practices and design patterns:

| Pattern | Description | Documentation |
|---------|-------------|---------------|
| [Architecture Patterns](patterns/architecture.md) | MVC, Repository, Service Layer patterns | [Learn More](patterns/architecture.md) |
| [Debugging Strategies](patterns/debugging.md) | Logging, profiling, and testing approaches | [Learn More](patterns/debugging.md) |

## Troubleshooting

Common issues and solutions:

| Guide | Description | Documentation |
|-------|-------------|---------------|
| [Common Errors](troubleshooting/common-errors.md) | Authorization, quota, type errors and solutions | [Learn More](troubleshooting/common-errors.md) |
| [Performance Guide](troubleshooting/performance-guide.md) | Optimization strategies and benchmarks | [Learn More](troubleshooting/performance-guide.md) |
| [Quota Limits](troubleshooting/quota-limits.md) | Service quotas, limits, and management | [Learn More](troubleshooting/quota-limits.md) |

## Additional Resources

- [Official Google Apps Script Documentation](https://developers.google.com/apps-script)
- [Google Workspace Admin Settings](https://admin.google.com)
- [OAuth Scopes Reference](https://developers.google.com/identity/protocols/oauth2/scopes)