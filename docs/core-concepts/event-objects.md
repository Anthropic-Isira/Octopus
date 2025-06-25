# Event Objects in Google Apps Script

[← Back to Main Documentation](../README.md)

## Overview

Event objects are special parameters passed to trigger functions containing information about what triggered the function. Understanding their structure is crucial for building responsive Apps Script applications.

## Simple Trigger Events

### onOpen Event

The `onOpen` trigger fires when a user opens a spreadsheet, document, presentation, or form.

```javascript
function onOpen(e) {
  // Event object for onOpen is limited
  // e parameter exists but has minimal properties
  
  // For Spreadsheets
  var ui = SpreadsheetApp.getUi();
  ui.createMenu('Custom Menu')
    .addItem('Option 1', 'function1')
    .addSeparator()
    .addSubMenu(ui.createMenu('Sub Menu')
      .addItem('Sub Option', 'function2'))
    .addToUi();
  
  // Auth mode check (simple triggers run in LIMITED mode)
  // Cannot access services requiring authorization
}
```

### onEdit Event

The `onEdit` trigger provides the most detailed event object for simple triggers.

```javascript
function onEdit(e) {
  // Full event object structure
  var event = {
    authMode: e.authMode,         // ScriptApp.AuthMode.LIMITED
    oldValue: e.oldValue,         // Previous cell value (single cell edits only)
    range: e.range,               // Range object that was edited
    source: e.source,             // Spreadsheet object
    user: e.user,                 // User object (limited info)
    value: e.value                // New value (single cell edits only)
  };
  
  // Practical example
  if (e.range.getColumn() === 1 && e.range.getRow() > 1) {
    // Timestamp column B when column A is edited
    e.range.offset(0, 1).setValue(new Date());
  }
  
  // Multi-cell edit detection
  if (!e.value && !e.oldValue) {
    // Multiple cells were edited
    var editedRange = e.range;
    console.log('Multiple cells edited: ' + editedRange.getA1Notation());
  }
}

// Detailed onEdit handling
function onEditAdvanced(e) {
  // Get all properties safely
  var editInfo = {
    sheet: e.range.getSheet().getName(),
    range: e.range.getA1Notation(),
    row: e.range.getRow(),
    column: e.range.getColumn(),
    numRows: e.range.getNumRows(),
    numCols: e.range.getNumColumns(),
    value: e.value || 'Multiple cells or empty',
    oldValue: e.oldValue || 'No previous value',
    user: e.user ? e.user.getEmail() : 'Unknown'
  };
  
  // Log edit for audit trail
  console.log('Edit made:', JSON.stringify(editInfo));
  
  // Conditional actions based on location
  switch(editInfo.sheet) {
    case 'Data':
      handleDataSheetEdit(e);
      break;
    case 'Config':
      handleConfigSheetEdit(e);
      break;
  }
}
```

### onSelectionChange Event

Triggered when user changes selection in a spreadsheet.

```javascript
function onSelectionChange(e) {
  // Event object properties
  var selection = {
    range: e.range,              // Currently selected range
    source: e.source,            // Spreadsheet object
    user: e.user                 // User who changed selection
  };
  
  // Example: Dynamic helper text
  var sheet = e.range.getSheet();
  var helpText = {
    'A1': 'Enter the project name here',
    'B1': 'Enter the start date',
    'C1': 'Enter the budget amount'
  };
  
  var notation = e.range.getA1Notation();
  if (helpText[notation]) {
    SpreadsheetApp.getActiveSpreadsheet().toast(
      helpText[notation],
      'Help',
      3
    );
  }
}
```

## Installable Trigger Events

### Form Submit Event

Form submit events have different structures depending on the context.

```javascript
// For Forms (standalone)
function onFormSubmit(e) {
  var formResponse = e.response;
  
  // Get all responses
  var itemResponses = formResponse.getItemResponses();
  
  itemResponses.forEach(function(itemResponse) {
    var question = itemResponse.getItem().getTitle();
    var answer = itemResponse.getResponse();
    
    console.log('Q: ' + question);
    console.log('A: ' + answer);
    
    // Handle different item types
    var itemType = itemResponse.getItem().getType();
    switch(itemType) {
      case FormApp.ItemType.MULTIPLE_CHOICE:
        handleMultipleChoice(itemResponse);
        break;
      case FormApp.ItemType.CHECKBOX:
        handleCheckbox(itemResponse);
        break;
      case FormApp.ItemType.FILE_UPLOAD:
        handleFileUpload(itemResponse);
        break;
    }
  });
  
  // Get respondent email if collected
  var email = formResponse.getRespondentEmail();
  
  // Get submission timestamp
  var timestamp = formResponse.getTimestamp();
}

// For Sheets (form responses)
function onFormSubmitSheet(e) {
  // Different event structure for sheet-bound forms
  var event = {
    authMode: e.authMode,
    namedValues: e.namedValues,    // Object with question: [answer] pairs
    range: e.range,                // Range where response was added
    source: e.source,              // Spreadsheet object
    triggerUid: e.triggerUid,      // Unique trigger identifier
    values: e.values               // Array of response values in order
  };
  
  // Example: Process form data
  var responses = {
    timestamp: e.values[0],
    email: e.values[1],
    name: e.namedValues['Full Name'][0],
    rating: e.namedValues['Satisfaction Rating'][0]
  };
  
  // Send confirmation email
  if (responses.email) {
    MailApp.sendEmail(
      responses.email,
      'Form Submission Received',
      'Thank you for your submission, ' + responses.name
    );
  }
}
```

### Change Event

The onChange trigger detects structural changes to a spreadsheet.

```javascript
function onChange(e) {
  // Change types
  var changeType = e.changeType;
  
  switch(changeType) {
    case 'INSERT_ROW':
      console.log('Row(s) inserted');
      handleRowInsert(e);
      break;
      
    case 'INSERT_COLUMN':
      console.log('Column(s) inserted');
      handleColumnInsert(e);
      break;
      
    case 'REMOVE_ROW':
      console.log('Row(s) removed');
      handleRowRemove(e);
      break;
      
    case 'REMOVE_COLUMN':
      console.log('Column(s) removed');
      handleColumnRemove(e);
      break;
      
    case 'INSERT_GRID':
      console.log('Sheet inserted');
      handleSheetInsert(e);
      break;
      
    case 'REMOVE_GRID':
      console.log('Sheet removed');
      handleSheetRemove(e);
      break;
      
    case 'FORMAT':
      console.log('Format changed');
      handleFormatChange(e);
      break;
      
    case 'OTHER':
      console.log('Other change');
      break;
  }
}

// Practical onChange handler
function onChangeAdvanced(e) {
  var sheet = SpreadsheetApp.getActiveSheet();
  var logSheet = e.source.getSheetByName('Change Log') || 
                 e.source.insertSheet('Change Log');
  
  // Log the change
  logSheet.appendRow([
    new Date(),
    e.changeType,
    Session.getActiveUser().getEmail(),
    sheet.getName()
  ]);
  
  // Maintain data integrity
  if (e.changeType === 'INSERT_COLUMN' || e.changeType === 'REMOVE_COLUMN') {
    // Update formulas that might be affected
    updateFormulasAfterStructureChange();
  }
}
```

### Time-Based Event

Time-based triggers receive minimal but useful timing information.

```javascript
function onTimeDriven(e) {
  // Time event properties
  var timeInfo = {
    authMode: e.authMode,
    triggerUid: e.triggerUid,
    year: e.year,
    month: e.month,              // 1-12
    'day-of-month': e['day-of-month'], // 1-31
    'day-of-week': e['day-of-week'],   // 1-7 (Sunday = 1)
    hour: e.hour,                // 0-23
    minute: e.minute,            // 0-59
    second: e.second,            // 0-59
    timezone: e.timezone,        // e.g., "America/New_York"
    'week-of-year': e['week-of-year']  // 1-52
  };
  
  // Example: Different actions based on time
  if (timeInfo.hour >= 9 && timeInfo.hour < 17) {
    // Business hours processing
    runBusinessHoursTask();
  } else {
    // After hours processing
    runAfterHoursTask();
  }
  
  // Weekly tasks
  if (timeInfo['day-of-week'] === 2 && timeInfo.hour === 9) {
    // Monday morning at 9 AM
    generateWeeklyReport();
  }
  
  // Monthly tasks
  if (timeInfo['day-of-month'] === 1) {
    generateMonthlyReport();
  }
}

// Advanced time-based processing
function processTimeEvent(e) {
  // Create a proper Date object from event
  var eventDate = new Date(
    e.year,
    e.month - 1,  // JavaScript months are 0-indexed
    e['day-of-month'],
    e.hour,
    e.minute,
    e.second
  );
  
  console.log('Trigger fired at:', eventDate);
  
  // Check if it's a business day
  var dayOfWeek = e['day-of-week'];
  var isWeekend = dayOfWeek === 1 || dayOfWeek === 7;
  
  if (!isWeekend) {
    performBusinessDayTasks();
  }
}
```

## Calendar Event Updates

Calendar triggers provide information about calendar changes.

```javascript
function onCalendarEventUpdate(e) {
  // Calendar event properties
  var calendarEvent = {
    authMode: e.authMode,
    calendarId: e.calendarId,    // ID of the calendar
    triggerUid: e.triggerUid
  };
  
  // Note: The event object doesn't include details about what changed
  // You need to track changes yourself
  
  // Example: Sync calendar changes
  var calendar = CalendarApp.getCalendarById(e.calendarId);
  var now = new Date();
  var oneWeekFromNow = new Date(now.getTime() + 7 * 24 * 60 * 60 * 1000);
  
  var events = calendar.getEvents(now, oneWeekFromNow);
  
  // Store event data for comparison
  var props = PropertiesService.getScriptProperties();
  var lastSync = props.getProperty('lastCalendarSync');
  
  if (lastSync) {
    var lastSyncData = JSON.parse(lastSync);
    // Compare and find changes
    findCalendarChanges(lastSyncData, events);
  }
  
  // Update sync data
  var currentData = events.map(function(event) {
    return {
      id: event.getId(),
      title: event.getTitle(),
      start: event.getStartTime().toISOString(),
      end: event.getEndTime().toISOString()
    };
  });
  
  props.setProperty('lastCalendarSync', JSON.stringify(currentData));
}
```

## Advanced Event Handling

### Event Validation and Safety

```javascript
function safeEventHandler(e) {
  try {
    // Validate event object exists
    if (!e) {
      console.error('No event object provided');
      return;
    }
    
    // Safely access nested properties
    var range = e.range;
    if (!range) {
      console.log('No range in event');
      return;
    }
    
    // Check for expected properties
    var hasValue = 'value' in e;
    var hasOldValue = 'oldValue' in e;
    
    // Handle based on available data
    if (hasValue && hasOldValue) {
      handleCellChange(e.oldValue, e.value, range);
    } else {
      handleMultiCellEdit(range);
    }
    
  } catch (error) {
    console.error('Error in event handler:', error);
    // Log error details for debugging
    logError(error, e);
  }
}

function logError(error, eventObj) {
  var errorSheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName('Error Log') || 
    SpreadsheetApp.getActiveSpreadsheet()
    .insertSheet('Error Log');
  
  errorSheet.appendRow([
    new Date(),
    error.toString(),
    error.stack,
    JSON.stringify(eventObj)
  ]);
}
```

### Event Object Inspection

```javascript
function inspectEventObject(e) {
  // Generic event inspector
  console.log('=== Event Object Inspection ===');
  console.log('Event type:', typeof e);
  
  if (e === null || e === undefined) {
    console.log('Event is null or undefined');
    return;
  }
  
  // List all properties
  for (var key in e) {
    try {
      var value = e[key];
      var type = typeof value;
      
      if (type === 'function') {
        console.log(key + ': [Function]');
      } else if (type === 'object') {
        if (value === null) {
          console.log(key + ': null');
        } else if (Array.isArray(value)) {
          console.log(key + ': [Array length=' + value.length + ']');
        } else {
          console.log(key + ': [Object]');
        }
      } else {
        console.log(key + ':', value, '(' + type + ')');
      }
    } catch (error) {
      console.log(key + ': [Error accessing property]');
    }
  }
  
  console.log('=== End Inspection ===');
}
```

### Custom Event Emulation

```javascript
// Create mock events for testing
function createMockEditEvent(range, value, oldValue) {
  return {
    authMode: ScriptApp.AuthMode.FULL,
    range: range,
    source: SpreadsheetApp.getActiveSpreadsheet(),
    user: Session.getActiveUser(),
    value: value,
    oldValue: oldValue
  };
}

function createMockFormEvent(responses) {
  return {
    authMode: ScriptApp.AuthMode.FULL,
    values: responses,
    namedValues: responses.reduce(function(obj, val, idx) {
      obj['Question ' + (idx + 1)] = [val];
      return obj;
    }, {}),
    range: SpreadsheetApp.getActiveRange(),
    source: SpreadsheetApp.getActiveSpreadsheet()
  };
}

// Test triggers without waiting for actual events
function testEditTrigger() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var testRange = sheet.getRange('A1');
  
  var mockEvent = createMockEditEvent(
    testRange,
    'New Value',
    'Old Value'
  );
  
  onEdit(mockEvent);
}
```

### Event-Driven State Machine

```javascript
// Complex event handling with state
class EventStateMachine {
  constructor() {
    this.state = this.loadState();
  }
  
  loadState() {
    var props = PropertiesService.getScriptProperties();
    var stateJson = props.getProperty('eventState');
    return stateJson ? JSON.parse(stateJson) : {
      lastEdit: null,
      editCount: 0,
      lastUser: null
    };
  }
  
  saveState() {
    PropertiesService.getScriptProperties()
      .setProperty('eventState', JSON.stringify(this.state));
  }
  
  handleEdit(e) {
    // Update state based on event
    this.state.lastEdit = new Date().toISOString();
    this.state.editCount++;
    this.state.lastUser = e.user ? e.user.getEmail() : 'Unknown';
    
    // Apply business rules based on state
    if (this.state.editCount > 100) {
      this.performMaintenance();
      this.state.editCount = 0;
    }
    
    this.saveState();
  }
  
  performMaintenance() {
    // Cleanup or optimization tasks
    console.log('Performing maintenance after 100 edits');
  }
}

// Usage
function onEditWithState(e) {
  var stateMachine = new EventStateMachine();
  stateMachine.handleEdit(e);
}
```

## Event Object Reference

### Quick Reference Table

| Trigger Type | Key Properties | Notes |
|-------------|---------------|-------|
| onOpen | authMode | Limited properties |
| onEdit | range, value, oldValue, user | Single cell edits only for value/oldValue |
| onSelectionChange | range, user | Current selection |
| onFormSubmit (Form) | response, source | Full form response object |
| onFormSubmit (Sheet) | values, namedValues, range | Array of responses |
| onChange | changeType | Structural changes only |
| onTime | year, month, day, hour, etc. | Time components |
| onCalendarUpdate | calendarId | Minimal info |

## Best Practices

1. **Always validate event objects** - Check for null/undefined
2. **Use try-catch blocks** - Events can have unexpected structures
3. **Log event data** - Helpful for debugging trigger issues
4. **Test with mock events** - Don't wait for real triggers
5. **Handle missing properties gracefully** - Not all properties are always present
6. **Consider event timing** - Events may fire in rapid succession
7. **Implement idempotency** - Same event might fire multiple times
8. **Monitor performance** - Event handlers should be fast
9. **Use installable triggers** for full event data when needed
10. **Document expected event structures** in your code