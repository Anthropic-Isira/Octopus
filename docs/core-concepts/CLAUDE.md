# Google Apps Script Platform Quirks & Limitations

## Critical Execution Context Issues

### Script Container Binding
- **CANNOT** create a new Sheet/Doc and run script inside it in same execution
- Scripts must be pre-bound or use standalone approach

### Global Variables Don't Persist
```javascript
// ❌ WRONG
var counter = 0;
function increment() { counter++; } // Always 1

// ✅ CORRECT
function increment() {
  var props = PropertiesService.getScriptProperties();
  var counter = parseInt(props.getProperty('counter') || '0');
  props.setProperty('counter', (counter + 1).toString());
}
```

### Authorization Scopes Lock at Deploy
- Adding new services = full reauthorization for ALL users
- Plan all needed scopes upfront in manifest

### This Context Lost in Triggers
```javascript
// ❌ BREAKS in triggers
obj.method = function() { console.log(this.data); }

// ✅ WORKS everywhere  
obj.method = function() { console.log(obj.data); }
```

## Time Limits & Performance

### 6-Minute Execution Timeout
- Hard limit: 6 min (30 for Workspace)
- Implement batch processing with resume for long operations
- Save progress to Properties Service before timeout

### Trigger Time ±15 Minutes
- Time triggers are NOT precise
- Build tolerance into time-sensitive operations
- Don't expect exact scheduling

### Lock Service Max 300 Seconds
```javascript
// ❌ FAILS
lock.waitLock(600000);

// ✅ Use reasonable waits with retry
lock.waitLock(10000); // 10 seconds max
```

## API Service Limits

### UrlFetch 50MB Response Cap
- Responses over 50MB fail
- Implement pagination or chunking
- Request compression when possible

### Properties Service: 9KB per property
```javascript
// ❌ May exceed limit
props.setProperty('data', hugeJsonString);

// ✅ Chunk large data
function saveChunked(key, data) {
  var str = JSON.stringify(data);
  var chunks = str.match(/.{1,8000}/g);
  props.setProperty(key + '_count', chunks.length);
  chunks.forEach((chunk, i) => props.setProperty(key + '_' + i, chunk));
}
```

### Cache Service 6-Hour Max
- Expires after 6 hours regardless of setting
- Use Properties Service for longer persistence

### No Drive "Computers" Access
- Can't access desktop-synced files
- Only "My Drive" and shared drives work

## Event & Trigger Gotchas

### Simple vs Installable Triggers
- Simple triggers: LIMITED auth (no email, external services)
- Installable triggers: FULL auth but run as creator

### onChange Fires for Formatting
- Triggers on non-data changes too
- Filter by changeType

### Triggers Run as Creator
- Not as current user
- Plan permissions accordingly

## Data Type Issues

### Dates Lose Milliseconds
```javascript
// Store as timestamp for precision
props.setProperty('time', new Date().getTime().toString());
```

### Large Numbers → Scientific Notation
```javascript
// Force text format for exact values
range.setNumberFormat('@').setValue('12345678901234567890');
```

### HTML Service Needs JSON
```javascript
// ❌ WRONG
'<script>var data = ' + objectArray + '</script>'

// ✅ CORRECT
template.data = JSON.stringify(objectArray);
```

## UI Constraints

### Modal Dialogs Block Everything
- User can't interact with sheet
- Use modeless or sidebar for continuous interaction

### Sidebars Max 300px Wide
- Design for narrow layouts
- No way to increase width

### No Custom Menus on Mobile
- Provide alternative triggers
- Use buttons in sheets for mobile

### Toast Notifications 5 Seconds Max
- Won't display longer regardless of setting
- Use dialogs for important messages

## Debugging Limitations

### Console.log Invisible in Triggers
- Only in Cloud Logging
- Implement custom logging to sheet/properties

### No Breakpoints in Triggers
- Create test functions that simulate events
- Use extensive logging

### Stack Traces Truncated
- Capture full error context
- Log to Properties for debugging

## Deployment Pitfalls

### HEAD Always Latest Code
- Can break without warning
- Use versioned deployments for stability

### Web App URLs Change
- Each deployment = new URL
- Use stable deployment IDs

### Library Updates Manual
- Dependent scripts don't auto-update
- Implement version checking

### Can't Reduce Scopes
- Would break existing authorizations
- Plan minimal scopes from start

## Key Patterns to Always Use

### Safe Resource Access
```javascript
try {
  var file = DriveApp.getFileById(id);
  file.getName(); // Test access
  return file;
} catch (e) {
  return null; // Handle gracefully
}
```

### Batch Processing
```javascript
// Process in chunks, save progress
if (timeApproaching()) {
  saveProgress();
  scheduleContinuation();
  return;
}
```

### Proper Error Handling
```javascript
try {
  riskyOperation();
} catch (e) {
  console.error('Context:', {error: e.toString(), stack: e.stack});
  // User-friendly message
  throw new Error('Operation failed. Error logged.');
}
```