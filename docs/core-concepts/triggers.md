# Google Apps Script Triggers

[← Back to Main Documentation](../README.md)

## Overview

Triggers allow Apps Script functions to run automatically when certain events occur. They're essential for automation and building responsive applications.

## Types of Triggers

### 1. Simple Triggers

Simple triggers run automatically without requiring authorization. They have limited capabilities but are easy to use.

#### onOpen()
Runs when a user opens a spreadsheet, document, presentation, or form.

```javascript
function onOpen() {
  var ui = SpreadsheetApp.getUi();
  ui.createMenu('Custom Menu')
    .addItem('Run Process', 'myFunction')
    .addToUi();
}
```

#### onEdit(e)
Runs when a user changes a value in a spreadsheet.

```javascript
function onEdit(e) {
  var range = e.range;
  var sheet = range.getSheet();
  
  // Log the edit
  console.log('Cell ' + range.getA1Notation() + ' edited');
  console.log('Old value: ' + e.oldValue);
  console.log('New value: ' + e.value);
}
```

#### onFormSubmit(e)
Runs when a user submits a Google Form.

```javascript
function onFormSubmit(e) {
  var responses = e.values;
  var timestamp = responses[0];
  var email = responses[1];
  
  // Process form submission
  console.log('Form submitted by: ' + email);
}
```

#### Simple Trigger Limitations
- Cannot access services that require authorization
- Cannot run for longer than 30 seconds
- Cannot access other files
- Cannot send emails
- Cannot make URL fetch calls

### 2. Installable Triggers

More powerful than simple triggers but require authorization.

#### Time-Based Triggers

```javascript
function installTimeTrigger() {
  ScriptApp.newTrigger('myFunction')
    .timeBased()
    .everyHours(1)
    .create();
}

// More time-based options
function installVariousTriggers() {
  // Every day at specific time
  ScriptApp.newTrigger('dailyReport')
    .timeBased()
    .atHour(9)
    .everyDays(1)
    .create();
  
  // Every week on specific day
  ScriptApp.newTrigger('weeklyBackup')
    .timeBased()
    .onWeekDay(ScriptApp.WeekDay.MONDAY)
    .atHour(2)
    .create();
  
  // Every month on specific date
  ScriptApp.newTrigger('monthlyReport')
    .timeBased()
    .onMonthDay(1)
    .atHour(6)
    .create();
  
  // At specific date/time (one-time)
  ScriptApp.newTrigger('oneTimeTask')
    .timeBased()
    .at(new Date('December 31, 2024 23:59:00'))
    .create();
}
```

#### Spreadsheet Triggers

```javascript
// Installable edit trigger
function installEditTrigger() {
  var ss = SpreadsheetApp.getActive();
  ScriptApp.newTrigger('onEditInstallable')
    .forSpreadsheet(ss)
    .onEdit()
    .create();
}

// Change trigger (detects any change including edits, inserts, deletes)
function installChangeTrigger() {
  var ss = SpreadsheetApp.getActive();
  ScriptApp.newTrigger('onChange')
    .forSpreadsheet(ss)
    .onChange()
    .create();
}

// Form submit trigger for spreadsheet
function installFormSubmitTrigger() {
  var ss = SpreadsheetApp.getActive();
  ScriptApp.newTrigger('onFormSubmitInstallable')
    .forSpreadsheet(ss)
    .onFormSubmit()
    .create();
}
```

#### Form Triggers

```javascript
function installFormTrigger() {
  var form = FormApp.openById('FORM_ID');
  ScriptApp.newTrigger('processFormSubmission')
    .forForm(form)
    .onFormSubmit()
    .create();
}
```

#### Calendar Triggers

```javascript
// Calendar event updated trigger
function installCalendarTrigger() {
  ScriptApp.newTrigger('onCalendarChange')
    .forCalendar('calendar@example.com')
    .onEventUpdated()
    .create();
}
```

## Managing Triggers

### List All Triggers

```javascript
function listTriggers() {
  var triggers = ScriptApp.getProjectTriggers();
  
  triggers.forEach(function(trigger) {
    console.log('Function: ' + trigger.getHandlerFunction());
    console.log('Type: ' + trigger.getEventType());
    console.log('Source: ' + trigger.getTriggerSource());
    console.log('ID: ' + trigger.getUniqueId());
    console.log('---');
  });
}
```

### Delete Triggers

```javascript
function deleteTrigger(functionName) {
  var triggers = ScriptApp.getProjectTriggers();
  
  triggers.forEach(function(trigger) {
    if (trigger.getHandlerFunction() === functionName) {
      ScriptApp.deleteTrigger(trigger);
      console.log('Deleted trigger for: ' + functionName);
    }
  });
}

// Delete all triggers
function deleteAllTriggers() {
  var triggers = ScriptApp.getProjectTriggers();
  triggers.forEach(function(trigger) {
    ScriptApp.deleteTrigger(trigger);
  });
}
```

### Update Triggers

```javascript
function updateTrigger(oldFunctionName, newFunctionName) {
  var triggers = ScriptApp.getProjectTriggers();
  
  triggers.forEach(function(trigger) {
    if (trigger.getHandlerFunction() === oldFunctionName) {
      // Delete old trigger
      ScriptApp.deleteTrigger(trigger);
      
      // Create new trigger with same settings
      if (trigger.getEventType() === ScriptApp.EventType.CLOCK) {
        ScriptApp.newTrigger(newFunctionName)
          .timeBased()
          .everyHours(1)
          .create();
      }
    }
  });
}
```

## Event Objects

Different triggers receive different event objects with useful information.

### Edit Event Object

```javascript
function onEditInstallable(e) {
  // Available properties:
  var user = e.user;           // User who made the edit
  var source = e.source;       // Spreadsheet object
  var range = e.range;         // Range that was edited
  var value = e.value;         // New value (single cell edits only)
  var oldValue = e.oldValue;   // Previous value (single cell edits only)
  
  // Example usage
  var sheet = range.getSheet();
  if (sheet.getName() === 'Data' && range.getColumn() === 1) {
    range.offset(0, 1).setValue(new Date()); // Add timestamp
  }
}
```

### Form Submit Event Object

```javascript
function onFormSubmitInstallable(e) {
  // For forms linked to sheets
  var range = e.range;
  var values = e.values;
  var namedValues = e.namedValues;
  
  // For standalone forms
  var response = e.response;
  var itemResponses = response.getItemResponses();
  
  itemResponses.forEach(function(itemResponse) {
    console.log('Question: ' + itemResponse.getItem().getTitle());
    console.log('Answer: ' + itemResponse.getResponse());
  });
}
```

### Time-based Event Object

```javascript
function timedFunction(e) {
  // Time-based triggers receive minimal event data
  var triggerUid = e.triggerUid;
  var year = e.year;
  var month = e.month;
  var dayOfMonth = e['day-of-month'];
  var dayOfWeek = e['day-of-week'];
  var hour = e.hour;
  var minute = e.minute;
  var second = e.second;
  var timezone = e.timezone;
}
```

## Debugging Triggers

### Logging and Monitoring

```javascript
function debuggableTriggerFunction(e) {
  try {
    // Log trigger execution
    console.log('Trigger executed at: ' + new Date());
    console.log('Event object: ' + JSON.stringify(e));
    
    // Your actual code here
    performTask();
    
    // Log success
    console.log('Trigger completed successfully');
    
  } catch (error) {
    // Log error details
    console.error('Trigger error: ' + error.toString());
    console.error('Stack trace: ' + error.stack);
    
    // Optional: Send email notification
    MailApp.sendEmail('admin@example.com', 
      'Trigger Error', 
      'Error in trigger: ' + error.toString());
    
    // Re-throw to ensure error is visible in executions
    throw error;
  }
}
```

### Testing Triggers Manually

```javascript
// Simulate trigger events for testing
function testEditTrigger() {
  var e = {
    range: SpreadsheetApp.getActiveRange(),
    value: 'Test Value',
    oldValue: 'Old Value',
    user: Session.getActiveUser().getEmail(),
    source: SpreadsheetApp.getActiveSpreadsheet()
  };
  
  onEditInstallable(e);
}

function testTimeTrigger() {
  var e = {
    triggerUid: 'test-trigger',
    year: new Date().getFullYear(),
    month: new Date().getMonth() + 1,
    'day-of-month': new Date().getDate(),
    hour: new Date().getHours(),
    minute: new Date().getMinutes()
  };
  
  timedFunction(e);
}
```

### Execution Transcript

Check execution history for trigger runs:

```javascript
function checkTriggerExecutions() {
  // This must be done through the UI:
  // View -> Executions
  // Filter by function name or status
  
  // Programmatically, you can only log:
  console.log('Check Executions in UI for detailed logs');
}
```

## Best Practices

### 1. Error Handling

```javascript
function robustTriggerFunction(e) {
  var maxRetries = 3;
  var retryCount = 0;
  
  while (retryCount < maxRetries) {
    try {
      // Main logic
      processData();
      break; // Success, exit loop
      
    } catch (error) {
      retryCount++;
      
      if (retryCount >= maxRetries) {
        // Final failure
        logError(error);
        sendErrorNotification(error);
        throw error;
      }
      
      // Wait before retry
      Utilities.sleep(1000 * retryCount);
    }
  }
}
```

### 2. Prevent Duplicate Triggers

```javascript
function installTriggerSafely(functionName) {
  // Check if trigger already exists
  var triggers = ScriptApp.getProjectTriggers();
  var exists = triggers.some(function(trigger) {
    return trigger.getHandlerFunction() === functionName;
  });
  
  if (!exists) {
    ScriptApp.newTrigger(functionName)
      .timeBased()
      .everyHours(1)
      .create();
    console.log('Trigger installed for: ' + functionName);
  } else {
    console.log('Trigger already exists for: ' + functionName);
  }
}
```

### 3. Rate Limiting

```javascript
function rateLimitedTrigger(e) {
  var lock = LockService.getScriptLock();
  
  try {
    // Wait up to 10 seconds for lock
    lock.waitLock(10000);
    
    // Check last execution time
    var props = PropertiesService.getScriptProperties();
    var lastRun = props.getProperty('lastTriggerRun');
    var now = new Date().getTime();
    
    if (lastRun && (now - parseInt(lastRun) < 60000)) {
      console.log('Skipping - ran less than 1 minute ago');
      return;
    }
    
    // Process trigger
    performTask();
    
    // Update last run time
    props.setProperty('lastTriggerRun', now.toString());
    
  } catch (error) {
    console.error('Could not obtain lock or error occurred:', error);
  } finally {
    lock.releaseLock();
  }
}
```

### 4. Trigger Permissions

```javascript
function checkTriggerPermissions() {
  try {
    // Test if we can create triggers
    var testTrigger = ScriptApp.newTrigger('dummyFunction')
      .timeBased()
      .after(1)
      .create();
    
    // If successful, delete it
    ScriptApp.deleteTrigger(testTrigger);
    console.log('Trigger permissions OK');
    
  } catch (error) {
    console.error('No trigger permissions:', error);
    // Handle permission error
  }
}
```

## Common Issues and Solutions

### Issue: Trigger Not Firing
```javascript
// Solution: Check trigger installation and logs
function troubleshootTrigger() {
  // 1. Verify trigger exists
  var triggers = ScriptApp.getProjectTriggers();
  console.log('Total triggers: ' + triggers.length);
  
  // 2. Check function name matches
  triggers.forEach(function(trigger) {
    console.log('Trigger function: ' + trigger.getHandlerFunction());
  });
  
  // 3. Check for errors in Executions
  // View -> Executions in the UI
}
```

### Issue: Trigger Running Multiple Times
```javascript
// Solution: Implement idempotency
function idempotentTrigger(e) {
  var props = PropertiesService.getScriptProperties();
  var processedKey = 'processed_' + generateEventKey(e);
  
  if (props.getProperty(processedKey)) {
    console.log('Event already processed');
    return;
  }
  
  // Process event
  performTask();
  
  // Mark as processed
  props.setProperty(processedKey, 'true');
}

function generateEventKey(e) {
  // Generate unique key based on event properties
  if (e.range) {
    return e.range.getA1Notation() + '_' + new Date().getTime();
  }
  return JSON.stringify(e);
}
```

### Issue: Trigger Quota Exceeded
```javascript
// Solution: Implement trigger management
function manageTriggerQuota() {
  var triggers = ScriptApp.getProjectTriggers();
  
  // Google Apps Script limits:
  // - 20 triggers per user per script
  // - 20 triggers per user across all scripts
  
  if (triggers.length >= 18) {
    console.warn('Approaching trigger limit: ' + triggers.length + '/20');
    
    // Delete oldest triggers if needed
    triggers.sort(function(a, b) {
      return a.getUniqueId().localeCompare(b.getUniqueId());
    });
    
    // Keep only newest 15 triggers
    for (var i = 0; i < triggers.length - 15; i++) {
      ScriptApp.deleteTrigger(triggers[i]);
    }
  }
}
```

## Trigger Limits and Quotas

| Limit | Consumer (gmail.com) | Google Workspace |
|-------|---------------------|------------------|
| Total triggers per script | 20 | 20 |
| Total triggers per user | 20 | 20 |
| Simultaneous executions | 30 | 30 |
| Trigger execution time | 6 min | 30 min |
| Time-based trigger frequency | 1 per hour | 1 per minute |

## Advanced Patterns

### Dynamic Trigger Management

```javascript
class TriggerManager {
  static install(config) {
    var trigger = ScriptApp.newTrigger(config.functionName);
    
    switch (config.type) {
      case 'time':
        trigger = trigger.timeBased();
        if (config.everyMinutes) trigger = trigger.everyMinutes(config.everyMinutes);
        if (config.everyHours) trigger = trigger.everyHours(config.everyHours);
        if (config.atHour) trigger = trigger.atHour(config.atHour);
        if (config.onWeekDay) trigger = trigger.onWeekDay(config.onWeekDay);
        break;
        
      case 'spreadsheet':
        trigger = trigger.forSpreadsheet(config.spreadsheetId);
        if (config.onEdit) trigger = trigger.onEdit();
        if (config.onChange) trigger = trigger.onChange();
        if (config.onFormSubmit) trigger = trigger.onFormSubmit();
        break;
        
      case 'form':
        trigger = trigger.forForm(config.formId);
        trigger = trigger.onFormSubmit();
        break;
    }
    
    return trigger.create();
  }
  
  static uninstall(functionName) {
    var triggers = ScriptApp.getProjectTriggers();
    triggers.forEach(function(trigger) {
      if (trigger.getHandlerFunction() === functionName) {
        ScriptApp.deleteTrigger(trigger);
      }
    });
  }
  
  static reinstall(config) {
    this.uninstall(config.functionName);
    return this.install(config);
  }
}

// Usage
TriggerManager.install({
  type: 'time',
  functionName: 'myFunction',
  everyHours: 2,
  atHour: 14
});
```

### Conditional Triggers

```javascript
function conditionalTrigger(e) {
  var config = {
    runOnWeekends: false,
    runBetweenHours: {start: 9, end: 17},
    runOnlyOnDays: [1, 15], // 1st and 15th of month
    skipHolidays: true
  };
  
  var now = new Date();
  
  // Check weekend
  if (!config.runOnWeekends && (now.getDay() === 0 || now.getDay() === 6)) {
    console.log('Skipping: Weekend');
    return;
  }
  
  // Check hours
  var hour = now.getHours();
  if (hour < config.runBetweenHours.start || hour >= config.runBetweenHours.end) {
    console.log('Skipping: Outside business hours');
    return;
  }
  
  // Check specific days
  if (config.runOnlyOnDays.indexOf(now.getDate()) === -1) {
    console.log('Skipping: Not a scheduled day');
    return;
  }
  
  // Check holidays (simplified)
  if (config.skipHolidays && isHoliday(now)) {
    console.log('Skipping: Holiday');
    return;
  }
  
  // Run actual task
  performScheduledTask();
}

function isHoliday(date) {
  // Implement holiday checking logic
  // Could use a calendar API or hardcoded list
  var holidays = ['2024-12-25', '2024-01-01'];
  var dateStr = Utilities.formatDate(date, 'GMT', 'yyyy-MM-dd');
  return holidays.indexOf(dateStr) !== -1;
}
```