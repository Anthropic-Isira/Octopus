# Google Apps Script Nuances & Gotchas

[← Back to Main Documentation](../README.md)

## Overview

This guide documents the lesser-known behaviors, limitations, and quirks of Google Apps Script that can frustrate developers. Understanding these nuances upfront will save you hours of debugging and help you write more robust scripts.

> **Note**: These behaviors are current as of 2024 and may change as Google updates the platform.

## Table of Contents

1. [Execution Context & Scope](#execution-context--scope)
2. [Time & Performance Limitations](#time--performance-limitations)
3. [API & Service Constraints](#api--service-constraints)
4. [Event & Trigger Behaviors](#event--trigger-behaviors)
5. [Data Type Conversions](#data-type-conversions)
6. [UI Service Limitations](#ui-service-limitations)
7. [Debugging Challenges](#debugging-challenges)
8. [Deployment Pitfalls](#deployment-pitfalls)

## Execution Context & Scope

### 1. Script Container Binding Limitation

**Issue**: You cannot create a new Google Sheet/Doc and have the script run inside that container during the same execution.

```javascript
// ❌ This won't work as expected
function createAndBindScript() {
  var newSheet = SpreadsheetApp.create('New Sheet');
  
  // This will fail - the script isn't bound to the new sheet
  var ui = SpreadsheetApp.getUi();
  ui.alert('Hello from new sheet'); // Error: no UI available
}

// ✅ Workaround: Use standalone scripts or pre-create containers
function workWithExistingSheet() {
  var sheet = SpreadsheetApp.openById('SHEET_ID');
  // Work with existing sheet
}
```

### 2. Authorization Scopes Determined at Deploy Time

**Issue**: Adding new services requires reauthorization of the entire script.

```javascript
// Initial deployment - only needs Spreadsheet scope
function originalFunction() {
  SpreadsheetApp.getActiveSheet().getRange('A1').setValue('Hello');
}

// Later addition - requires new Drive scope
function addedLaterFunction() {
  // This triggers reauthorization for ALL users
  DriveApp.getFiles();
}

// ✅ Best Practice: Declare all needed scopes upfront in manifest
{
  "oauthScopes": [
    "https://www.googleapis.com/auth/spreadsheets",
    "https://www.googleapis.com/auth/drive.readonly"
  ]
}
```

### 3. Execution Context Loss with Triggers

**Issue**: `this` context behaves differently in triggers vs direct calls.

```javascript
// ❌ Problematic code
var MyApp = {
  data: 'important',
  
  processData: function() {
    console.log(this.data); // Works in direct call, undefined in trigger
  }
};

// ✅ Solution: Avoid relying on 'this' in trigger functions
var MyApp = {
  data: 'important',
  
  processData: function() {
    console.log(MyApp.data); // Always works
  }
};
```

### 4. Global Variables Don't Persist Between Executions

**Issue**: Global variables are reset with each execution.

```javascript
// ❌ This counter won't work as expected
var executionCount = 0;

function trackExecutions() {
  executionCount++; // Always 1, never increments
  console.log(executionCount);
}

// ✅ Use Properties Service for persistence
function trackExecutionsProperly() {
  var props = PropertiesService.getScriptProperties();
  var count = parseInt(props.getProperty('executionCount') || '0');
  count++;
  props.setProperty('executionCount', count.toString());
  console.log('Execution #' + count);
}
```

## Time & Performance Limitations

### 5. Six-Minute Hard Timeout

**Issue**: Scripts terminate after 6 minutes (30 for Workspace), no extensions possible.

```javascript
// ✅ Batch processing with resume capability
function processManyRows() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var scriptProps = PropertiesService.getScriptProperties();
  var startRow = parseInt(scriptProps.getProperty('lastProcessedRow') || '2');
  var maxRuntime = 5 * 60 * 1000; // 5 minutes (leave buffer)
  var startTime = new Date().getTime();
  
  var lastRow = sheet.getLastRow();
  
  for (var i = startRow; i <= lastRow; i++) {
    // Process row
    processRow(sheet, i);
    
    // Check if approaching timeout
    if (new Date().getTime() - startTime > maxRuntime) {
      // Save progress
      scriptProps.setProperty('lastProcessedRow', i.toString());
      
      // Schedule continuation
      ScriptApp.newTrigger('processManyRows')
        .timeBased()
        .after(1000)
        .create();
      
      console.log('Paused at row ' + i + ', will resume');
      return;
    }
  }
  
  // Completed - clean up
  scriptProps.deleteProperty('lastProcessedRow');
}
```

### 6. Trigger Time Precision ±15 Minutes

**Issue**: Time-based triggers are not precise.

```javascript
// ❌ Don't expect exact timing
function scheduleAt3PM() {
  ScriptApp.newTrigger('myFunction')
    .timeBased()
    .atHour(15)
    .create();
  // Might run anywhere from 2:45 PM to 3:15 PM
}

// ✅ Build tolerance into time-sensitive operations
function timeTolerантFunction() {
  var now = new Date();
  var targetHour = 15;
  var currentHour = now.getHours();
  
  // Accept execution within a window
  if (Math.abs(currentHour - targetHour) <= 1) {
    // Proceed with task
  } else {
    console.log('Execution outside acceptable window');
  }
}
```

### 7. Lock Service Maximum Wait

**Issue**: Lock service has a 300-second maximum wait time.

```javascript
// ❌ This will fail
function longLockWait() {
  var lock = LockService.getScriptLock();
  lock.waitLock(600000); // Error: exceeds maximum
}

// ✅ Use reasonable wait times with retry
function properLocking() {
  var lock = LockService.getScriptLock();
  var hasLock = false;
  var attempts = 0;
  
  while (!hasLock && attempts < 3) {
    try {
      lock.waitLock(10000); // 10 seconds
      hasLock = true;
    } catch (e) {
      attempts++;
      console.log('Lock attempt ' + attempts + ' failed');
      Utilities.sleep(1000); // Wait before retry
    }
  }
  
  if (!hasLock) {
    throw new Error('Could not obtain lock');
  }
  
  try {
    // Critical section
  } finally {
    lock.releaseLock();
  }
}
```

### 8. Quota Reset at Midnight Pacific Time

**Issue**: Quotas don't reset at user's midnight.

```javascript
// ✅ Account for Pacific Time quota resets
function quotaAwareOperation() {
  var now = new Date();
  var pacificTime = new Date(now.toLocaleString("en-US", {timeZone: "America/Los_Angeles"}));
  var pacificHour = pacificTime.getHours();
  
  // If it's late in Pacific Time, maybe wait
  if (pacificHour >= 22) {
    console.log('Close to quota reset, consider waiting');
  }
  
  // Check remaining quota
  var remainingQuota = MailApp.getRemainingDailyQuota();
  if (remainingQuota < 50) {
    console.log('Low quota, scheduling for tomorrow');
    // Schedule for after midnight Pacific
  }
}
```

## API & Service Constraints

### 9. UrlFetch 50MB Response Limit

**Issue**: Large responses fail silently or throw errors.

```javascript
// ✅ Handle large responses properly
function fetchLargeData(url) {
  try {
    var response = UrlFetchApp.fetch(url, {
      headers: {
        'Accept-Encoding': 'gzip' // Request compression
      }
    });
    
    // Check Content-Length if available
    var contentLength = response.getHeaders()['Content-Length'];
    if (contentLength && parseInt(contentLength) > 40 * 1024 * 1024) {
      throw new Error('Response too large');
    }
    
    return response.getContentText();
  } catch (e) {
    if (e.toString().includes('too large')) {
      // Implement pagination or chunking
      return fetchInChunks(url);
    }
    throw e;
  }
}
```

### 10. Properties Service Size Limits

**Issue**: 9KB per property, 500KB total storage.

```javascript
// ❌ This might fail for large data
function saveLargeData(data) {
  var props = PropertiesService.getScriptProperties();
  props.setProperty('data', JSON.stringify(data)); // May exceed 9KB
}

// ✅ Chunk large data across multiple properties
function saveLargeDataChunked(data) {
  var props = PropertiesService.getScriptProperties();
  var jsonString = JSON.stringify(data);
  var chunkSize = 8000; // Leave buffer under 9KB
  var chunks = [];
  
  for (var i = 0; i < jsonString.length; i += chunkSize) {
    chunks.push(jsonString.substr(i, chunkSize));
  }
  
  // Save chunks
  props.setProperty('dataChunkCount', chunks.length.toString());
  chunks.forEach((chunk, index) => {
    props.setProperty('dataChunk_' + index, chunk);
  });
}

function loadLargeDataChunked() {
  var props = PropertiesService.getScriptProperties();
  var chunkCount = parseInt(props.getProperty('dataChunkCount') || '0');
  var jsonString = '';
  
  for (var i = 0; i < chunkCount; i++) {
    jsonString += props.getProperty('dataChunk_' + i) || '';
  }
  
  return JSON.parse(jsonString);
}
```

### 11. Cache Service 6-Hour Maximum

**Issue**: Cache expires after 6 hours regardless of setting.

```javascript
// ❌ This won't actually cache for 24 hours
function longCache() {
  var cache = CacheService.getScriptCache();
  cache.put('key', 'value', 86400); // 24 hours requested
  // Actually expires after 6 hours
}

// ✅ Implement cache with fallback
function reliableCache(key, value) {
  var cache = CacheService.getScriptCache();
  var props = PropertiesService.getScriptProperties();
  
  if (value !== undefined) {
    // Set value
    cache.put(key, value, 21600); // 6 hours (maximum)
    props.setProperty('cache_' + key, value); // Persistent backup
    props.setProperty('cache_time_' + key, new Date().getTime().toString());
  } else {
    // Get value
    var cached = cache.get(key);
    if (cached) return cached;
    
    // Fallback to properties
    var stored = props.getProperty('cache_' + key);
    if (stored) {
      // Re-populate cache
      cache.put(key, stored, 21600);
      return stored;
    }
    
    return null;
  }
}
```

### 12. Drive "Computers" Section Inaccessible

**Issue**: Can't access files synced from desktop.

```javascript
// ❌ These files won't be found
function findComputerFiles() {
  var files = DriveApp.searchFiles('title contains "desktop file"');
  // Won't find files in Computers section
}

// ✅ Guide users to move files to My Drive
function ensureAccessibleFiles() {
  var ui = SpreadsheetApp.getUi();
  ui.alert(
    'File Access Note',
    'Please ensure files are in "My Drive", not "Computers" section.',
    ui.ButtonSet.OK
  );
}
```

## Event & Trigger Behaviors

### 13. Simple vs Installable Trigger Authorization

**Issue**: Simple triggers run with limited authorization.

```javascript
// ❌ Simple trigger can't do this
function onEdit(e) {
  MailApp.sendEmail('user@example.com', 'Edit made', 'Sheet was edited');
  // Error: No authorization
}

// ✅ Use installable trigger for full access
function installEditTrigger() {
  ScriptApp.newTrigger('onEditInstallable')
    .forSpreadsheet(SpreadsheetApp.getActive())
    .onEdit()
    .create();
}

function onEditInstallable(e) {
  // Can send emails, access other files, etc.
  MailApp.sendEmail('user@example.com', 'Edit made', 'Sheet was edited');
}
```

### 14. onChange Fires for Format Changes

**Issue**: onChange triggers for non-data changes.

```javascript
// ✅ Filter onChange events
function onChange(e) {
  // Check change type
  if (e.changeType === 'FORMAT') {
    console.log('Ignoring format change');
    return;
  }
  
  if (e.changeType === 'EDIT') {
    // Handle actual data changes
  }
}
```

### 15. Trigger Ownership and Execution

**Issue**: Triggers run as creator, not current user.

```javascript
// ✅ Be aware of execution context
function triggerFunction(e) {
  var triggerOwner = Session.getEffectiveUser().getEmail();
  var documentOwner = Session.getActiveUser().getEmail();
  
  console.log('Trigger runs as: ' + triggerOwner);
  console.log('Document owner: ' + documentOwner);
  
  // May need to handle permissions differently
  if (triggerOwner !== documentOwner) {
    // Limited access to user-specific resources
  }
}
```

### 16. Multiple Triggers Can Fire Unpredictably

**Issue**: Same event might trigger multiple functions.

```javascript
// ✅ Implement idempotency
function robustTriggerHandler(e) {
  var lock = LockService.getScriptLock();
  try {
    lock.waitLock(10000);
    
    // Check if already processed
    var cache = CacheService.getScriptCache();
    var eventId = e.range.getA1Notation() + '_' + new Date().getTime();
    
    if (cache.get(eventId)) {
      console.log('Event already processed');
      return;
    }
    
    // Process event
    processEvent(e);
    
    // Mark as processed
    cache.put(eventId, 'processed', 300); // 5 minutes
    
  } catch (e) {
    console.error('Could not obtain lock');
  } finally {
    lock.releaseLock();
  }
}
```

## Data Type Conversions

### 17. Date Objects Lose Milliseconds

**Issue**: Dates lose precision through Properties/Cache services.

```javascript
// ❌ Milliseconds are lost
function datePrecisionIssue() {
  var now = new Date();
  var props = PropertiesService.getScriptProperties();
  
  props.setProperty('timestamp', now.toString());
  var retrieved = new Date(props.getProperty('timestamp'));
  
  console.log(now.getMilliseconds()); // e.g., 123
  console.log(retrieved.getMilliseconds()); // 0
}

// ✅ Store as timestamp for precision
function preserveDatePrecision() {
  var now = new Date();
  var props = PropertiesService.getScriptProperties();
  
  props.setProperty('timestamp', now.getTime().toString());
  var retrieved = new Date(parseInt(props.getProperty('timestamp')));
  
  console.log(now.getMilliseconds() === retrieved.getMilliseconds()); // true
}
```

### 18. HTML Service Array Passing

**Issue**: Complex objects need serialization.

```javascript
// ❌ Arrays/objects don't pass directly
function showDialog() {
  var data = [{id: 1, name: 'Item 1'}, {id: 2, name: 'Item 2'}];
  var html = HtmlService.createHtmlOutput(
    '<script>var data = ' + data + ';</script>'
  ); // Results in: var data = [object Object],[object Object];
}

// ✅ Properly serialize data
function showDialogProperly() {
  var data = [{id: 1, name: 'Item 1'}, {id: 2, name: 'Item 2'}];
  var template = HtmlService.createTemplate('<script>var data = <?= data ?>;</script>');
  template.data = JSON.stringify(data);
  
  var html = template.evaluate();
  SpreadsheetApp.getUi().showModalDialog(html, 'Data Dialog');
}
```

### 19. Large Number Scientific Notation

**Issue**: Large numbers converted in sheets.

```javascript
// ❌ Large numbers lose precision
function largeNumberIssue() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var largeNumber = 12345678901234567890;
  
  sheet.getRange('A1').setValue(largeNumber);
  // Displays as: 1.23457E+19
  
  var retrieved = sheet.getRange('A1').getValue();
  console.log(retrieved === largeNumber); // false
}

// ✅ Store as string for exact values
function handleLargeNumbers() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var largeNumber = '12345678901234567890';
  
  // Force text format
  sheet.getRange('A1').setNumberFormat('@').setValue(largeNumber);
}
```

## UI Service Limitations

### 20. Dialog Modal Blocks Interaction

**Issue**: Can't interact with sheet while dialog open.

```javascript
// ✅ Use modeless dialog or sidebar for continuous interaction
function showNonBlockingUI() {
  var html = HtmlService.createHtmlOutput('<p>Non-blocking UI</p>')
    .setWidth(300)
    .setHeight(200);
  
  // Use modeless dialog
  SpreadsheetApp.getUi().showModelessDialog(html, 'Non-blocking');
  
  // Or use sidebar
  SpreadsheetApp.getUi().showSidebar(html);
}
```

### 21. Sidebar Width Maximum 300px

**Issue**: Sidebars can't be wider than 300px.

```javascript
// ✅ Design for narrow sidebar
function createResponsiveSidebar() {
  var html = HtmlService.createHtmlOutput(`
    <style>
      body { width: 280px; padding: 10px; }
      .content { max-width: 100%; overflow-x: auto; }
    </style>
    <div class="content">
      <!-- Sidebar content -->
    </div>
  `);
  
  SpreadsheetApp.getUi().showSidebar(html);
}
```

### 22. Custom Menus Disappear on Mobile

**Issue**: Custom menus not available in mobile apps.

```javascript
// ✅ Provide alternative access methods
function onOpen() {
  var ui = SpreadsheetApp.getUi();
  ui.createMenu('Custom Menu')
    .addItem('Run Function', 'myFunction')
    .addToUi();
  
  // Also add mobile-friendly alternative
  var sheet = SpreadsheetApp.getActiveSheet();
  sheet.getRange('A1').setValue('Click to run →');
  sheet.getRange('B1').setValue('RUN').setBackground('#4CAF50');
  
  // Use installable trigger for mobile
  ScriptApp.newTrigger('onMobileClick')
    .forSpreadsheet(sheet.getParent())
    .onEdit()
    .create();
}

function onMobileClick(e) {
  if (e.range.getA1Notation() === 'B1') {
    myFunction();
  }
}
```

### 23. Toast Notification Duration Limit

**Issue**: Toast notifications disappear after 5 seconds max.

```javascript
// ❌ This won't show for 10 seconds
function longToast() {
  SpreadsheetApp.getActiveSpreadsheet().toast('Message', 'Title', 10);
  // Still disappears after 5 seconds
}

// ✅ Use dialog for important messages
function importantMessage(message) {
  var ui = SpreadsheetApp.getUi();
  
  if (message.length > 50) {
    // Use dialog for longer messages
    ui.alert('Important', message, ui.ButtonSet.OK);
  } else {
    // Toast for brief messages
    SpreadsheetApp.getActiveSpreadsheet().toast(message, 'Notice', 5);
  }
}
```

## Debugging Challenges

### 24. Console.log Invisible in Triggers

**Issue**: Trigger logs only visible in Cloud Logging.

```javascript
// ✅ Implement custom logging
function customLog(message) {
  // Console for development
  console.log(message);
  
  // Also log to sheet for production
  try {
    var logSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Logs');
    if (!logSheet) {
      logSheet = SpreadsheetApp.getActiveSpreadsheet().insertSheet('Logs');
    }
    
    logSheet.appendRow([
      new Date(),
      message,
      Session.getActiveUser().getEmail()
    ]);
  } catch (e) {
    // Fallback to Properties
    var props = PropertiesService.getScriptProperties();
    var logs = JSON.parse(props.getProperty('logs') || '[]');
    logs.push({time: new Date().toISOString(), message: message});
    
    // Keep last 100 logs
    if (logs.length > 100) logs = logs.slice(-100);
    props.setProperty('logs', JSON.stringify(logs));
  }
}
```

### 25. Stack Traces Truncated

**Issue**: Error details lost in production.

```javascript
// ✅ Capture full error context
function robustErrorHandling() {
  try {
    riskyOperation();
  } catch (error) {
    var errorInfo = {
      message: error.toString(),
      stack: error.stack,
      time: new Date().toISOString(),
      user: Session.getActiveUser().getEmail(),
      context: 'Function: robustErrorHandling'
    };
    
    // Log full error
    console.error(JSON.stringify(errorInfo, null, 2));
    
    // Store for debugging
    var props = PropertiesService.getScriptProperties();
    props.setProperty('lastError', JSON.stringify(errorInfo));
    
    // User-friendly message
    throw new Error('Operation failed. Error logged for admin review.');
  }
}
```

### 26. Breakpoints Don't Work in Triggers

**Issue**: Can't debug triggers interactively.

```javascript
// ✅ Create test functions that simulate triggers
function testOnEdit() {
  // Simulate edit event
  var e = {
    range: SpreadsheetApp.getActiveRange(),
    value: SpreadsheetApp.getActiveRange().getValue(),
    oldValue: 'old',
    user: Session.getActiveUser(),
    source: SpreadsheetApp.getActiveSpreadsheet()
  };
  
  // Now you can set breakpoints here
  onEdit(e);
}

// Also use extensive logging
function onEdit(e) {
  customLog('Edit event: ' + JSON.stringify({
    range: e.range.getA1Notation(),
    value: e.value,
    oldValue: e.oldValue,
    user: e.user.getEmail()
  }));
  
  // Your code here
}
```

### 27. Testing Triggers Accurately

**Issue**: Can't perfectly simulate trigger events.

```javascript
// ✅ Comprehensive trigger testing framework
var TriggerTester = {
  // Simulate time-based trigger
  testTimeTrigger: function(functionName) {
    var e = {
      authMode: ScriptApp.AuthMode.FULL,
      triggerUid: 'test-trigger-' + Utilities.getUuid(),
      minute: new Date().getMinutes(),
      hour: new Date().getHours(),
      'day-of-month': new Date().getDate(),
      'day-of-week': new Date().getDay(),
      month: new Date().getMonth() + 1,
      year: new Date().getFullYear()
    };
    
    this[functionName](e);
  },
  
  // Simulate form submit
  testFormSubmit: function(functionName) {
    var e = {
      authMode: ScriptApp.AuthMode.FULL,
      values: ['Test User', 'test@example.com', 'Test response'],
      namedValues: {
        'Name': ['Test User'],
        'Email': ['test@example.com'],
        'Response': ['Test response']
      },
      range: SpreadsheetApp.getActiveSheet().getRange('A1:C1'),
      source: SpreadsheetApp.getActiveSpreadsheet()
    };
    
    this[functionName](e);
  },
  
  // Simulate installable onChange
  testOnChange: function(functionName) {
    var e = {
      authMode: ScriptApp.AuthMode.FULL,
      changeType: 'EDIT', // or 'INSERT_ROW', 'REMOVE_ROW', 'INSERT_COLUMN', etc.
      user: Session.getActiveUser(),
      source: SpreadsheetApp.getActiveSpreadsheet()
    };
    
    this[functionName](e);
  }
};
```

## Deployment Pitfalls

### 28. HEAD Deployment Always Latest

**Issue**: HEAD deployment can break without warning.

```javascript
// ✅ Use versioned deployments for stability
function deploymentBestPractices() {
  // In your code, check deployment version
  var deploymentId = ScriptApp.getScriptId();
  
  // Log version for debugging
  console.log('Running deployment: ' + deploymentId);
  
  // Implement feature flags for safe rollout
  var props = PropertiesService.getScriptProperties();
  var features = JSON.parse(props.getProperty('features') || '{}');
  
  if (features.newFeature) {
    // New feature code
  } else {
    // Stable code
  }
}
```

### 29. Web App URL Changes

**Issue**: URLs change with each deployment.

```javascript
// ✅ Use stable URL pattern
function getStableWebAppUrl() {
  // Deploy with "Execute as me" and specific version
  // URL format: https://script.google.com/macros/s/{DEPLOYMENT_ID}/exec
  
  // Store deployment ID
  var props = PropertiesService.getScriptProperties();
  var stableDeploymentId = props.getProperty('stableDeploymentId');
  
  if (!stableDeploymentId) {
    throw new Error('No stable deployment configured');
  }
  
  return 'https://script.google.com/macros/s/' + stableDeploymentId + '/exec';
}
```

### 30. Library Updates Don't Auto-Propagate

**Issue**: Dependent scripts don't get library updates automatically.

```javascript
// ✅ Version checking in libraries
var MyLibrary = {
  VERSION: '1.2.3',
  
  checkVersion: function(requiredVersion) {
    if (this.compareVersions(this.VERSION, requiredVersion) < 0) {
      throw new Error(
        'Library version ' + this.VERSION + 
        ' is older than required ' + requiredVersion
      );
    }
  },
  
  compareVersions: function(v1, v2) {
    var parts1 = v1.split('.').map(Number);
    var parts2 = v2.split('.').map(Number);
    
    for (var i = 0; i < 3; i++) {
      if (parts1[i] > parts2[i]) return 1;
      if (parts1[i] < parts2[i]) return -1;
    }
    return 0;
  }
};

// In dependent script
function useLibrary() {
  MyLibrary.checkVersion('1.2.0'); // Ensure minimum version
  // Use library
}
```

### 31. Manifest Scope Reduction Breaks Auth

**Issue**: Can't reduce scopes without breaking existing users.

```javascript
// ✅ Plan scopes carefully from the start
{
  "oauthScopes": [
    // Start with minimal scopes
    "https://www.googleapis.com/auth/spreadsheets.currentonly",
    
    // DON'T start with broad scopes you might want to reduce later
    // "https://www.googleapis.com/auth/spreadsheets"
  ]
}

// If you must change scopes, version your deployment
function handleScopeTransition() {
  try {
    // Try new limited scope operation
    limitedScopeOperation();
  } catch (e) {
    if (e.toString().includes('authorization')) {
      // Guide user through reauthorization
      var ui = SpreadsheetApp.getUi();
      ui.alert(
        'Reauthorization Required',
        'This script needs to be reauthorized with updated permissions. ' +
        'Please run the script again and accept the new permissions.',
        ui.ButtonSet.OK
      );
    }
    throw e;
  }
}
```

## Best Practices Summary

1. **Always test trigger functions** with simulated events
2. **Implement proper error handling** with detailed logging
3. **Plan authorization scopes** carefully from the start
4. **Use Properties Service** for persistence, not globals
5. **Design for mobile** limitations from the beginning
6. **Chunk large operations** to handle timeouts
7. **Version your deployments** for production stability
8. **Test with quota limits** in mind
9. **Handle all data type conversions** explicitly
10. **Document these gotchas** in your code comments

## Related Documentation

- [Common Errors Guide](../troubleshooting/common-errors.md)
- [Performance Optimization](../troubleshooting/performance-guide.md)
- [Architecture Patterns](../patterns/architecture.md)
- [Debugging Guide](../patterns/debugging.md)