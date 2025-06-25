# Debugging Google Apps Script

[← Back to Main Documentation](../README.md)

## Overview

Debugging Apps Script can be challenging due to its cloud-based nature. This guide covers tools, techniques, and strategies for effective debugging.

## Logging and Console

### Basic Logging

```javascript
// console.log() - Appears in Stackdriver Logging
console.log('Simple log message');
console.log('Object:', {name: 'John', age: 30});
console.log('Multiple values:', 'Value1', 'Value2', 123);

// Different log levels
console.info('Information message');
console.warn('Warning message');
console.error('Error message');

// Timing operations
console.time('operation');
// ... code to measure ...
console.timeEnd('operation'); // Logs: operation: 123ms

// Conditional logging
var DEBUG = true;
if (DEBUG) console.log('Debug info');
```

### Advanced Logging

```javascript
// Custom logger with levels
class Logger {
  static LEVELS = {
    DEBUG: 0,
    INFO: 1,
    WARN: 2,
    ERROR: 3
  };
  
  constructor(name, level = Logger.LEVELS.INFO) {
    this.name = name;
    this.level = level;
  }
  
  debug(...args) {
    if (this.level <= Logger.LEVELS.DEBUG) {
      console.log(`[${this.name}] DEBUG:`, ...args);
    }
  }
  
  info(...args) {
    if (this.level <= Logger.LEVELS.INFO) {
      console.info(`[${this.name}] INFO:`, ...args);
    }
  }
  
  warn(...args) {
    if (this.level <= Logger.LEVELS.WARN) {
      console.warn(`[${this.name}] WARN:`, ...args);
    }
  }
  
  error(...args) {
    if (this.level <= Logger.LEVELS.ERROR) {
      console.error(`[${this.name}] ERROR:`, ...args);
    }
  }
  
  time(label) {
    console.time(`[${this.name}] ${label}`);
  }
  
  timeEnd(label) {
    console.timeEnd(`[${this.name}] ${label}`);
  }
}

// Usage
var logger = new Logger('UserService', Logger.LEVELS.DEBUG);
logger.debug('Starting user creation');
logger.info('User created:', userId);
logger.error('Failed to create user:', error);
```

### Structured Logging

```javascript
// Log with context
class ContextLogger {
  constructor(context = {}) {
    this.context = context;
  }
  
  log(level, message, data = {}) {
    var logEntry = {
      timestamp: new Date().toISOString(),
      level: level,
      message: message,
      context: this.context,
      data: data,
      stack: this.getCallStack()
    };
    
    console.log(JSON.stringify(logEntry, null, 2));
  }
  
  getCallStack() {
    try {
      throw new Error();
    } catch (e) {
      var stack = e.stack.split('\n');
      // Remove first 3 lines (Error, getCallStack, log)
      return stack.slice(3, 6).map(line => line.trim());
    }
  }
  
  withContext(additionalContext) {
    return new ContextLogger({
      ...this.context,
      ...additionalContext
    });
  }
}

// Usage
var logger = new ContextLogger({
  service: 'DataProcessor',
  version: '1.0.0'
});

function processData(data) {
  var requestLogger = logger.withContext({
    requestId: Utilities.getUuid(),
    dataSize: data.length
  });
  
  requestLogger.log('INFO', 'Starting data processing');
  
  try {
    // Process data
    requestLogger.log('INFO', 'Data processed successfully');
  } catch (error) {
    requestLogger.log('ERROR', 'Processing failed', {
      error: error.toString(),
      stack: error.stack
    });
  }
}
```

## Debugging Tools

### Script Editor Debugger

```javascript
// Using breakpoints
function debugExample() {
  var data = fetchData();
  
  // Set breakpoint here in editor
  debugger; // Or use editor UI
  
  var processed = processData(data);
  
  return processed;
}

// Debug-friendly code structure
function debugFriendlyFunction() {
  // Break complex expressions into steps
  var step1 = getInitialData();
  var step2 = transformData(step1);
  var step3 = validateData(step2);
  var result = finalizeData(step3);
  
  return result;
}
```

### Execution Transcript

```javascript
// View execution history
function checkExecutions() {
  // Access via: View > Executions in script editor
  
  // Add execution markers
  console.log('=== EXECUTION START ===');
  console.log('Function:', arguments.callee.name);
  console.log('Time:', new Date());
  console.log('User:', Session.getActiveUser().getEmail());
  
  try {
    // Your code
    console.log('=== EXECUTION SUCCESS ===');
  } catch (error) {
    console.error('=== EXECUTION FAILED ===');
    console.error(error);
    throw error;
  }
}
```

### Stack Driver Logging

```javascript
// Access advanced logs
function setupStackdriverLogging() {
  // Enable in Google Cloud Console
  // View logs at: console.cloud.google.com/logs
  
  // Log with severity
  console.log({
    message: 'Custom log entry',
    severity: 'WARNING',
    labels: {
      function: 'processData',
      version: '1.0.0'
    }
  });
}

// Query logs programmatically
function queryLogs() {
  // Requires Cloud Logging API
  var projectId = 'your-project-id';
  var filter = 'resource.type="app_script_function"';
  
  // Use UrlFetchApp with OAuth2
}
```

## Debugging Strategies

### State Inspection

```javascript
// Dump application state
class StateInspector {
  static inspect() {
    var state = {
      properties: this.getProperties(),
      cache: this.getCache(),
      triggers: this.getTriggers(),
      user: this.getUserInfo(),
      quotas: this.getQuotas()
    };
    
    console.log('=== APPLICATION STATE ===');
    console.log(JSON.stringify(state, null, 2));
    
    return state;
  }
  
  static getProperties() {
    return {
      script: PropertiesService.getScriptProperties().getProperties(),
      user: PropertiesService.getUserProperties().getProperties(),
      document: PropertiesService.getDocumentProperties()?.getProperties()
    };
  }
  
  static getCache() {
    // Cache cannot be inspected directly
    return {
      scriptCache: 'Active',
      userCache: 'Active'
    };
  }
  
  static getTriggers() {
    return ScriptApp.getProjectTriggers().map(trigger => ({
      function: trigger.getHandlerFunction(),
      type: trigger.getEventType(),
      id: trigger.getUniqueId()
    }));
  }
  
  static getUserInfo() {
    return {
      email: Session.getActiveUser().getEmail(),
      timezone: Session.getScriptTimeZone(),
      locale: Session.getActiveUserLocale()
    };
  }
  
  static getQuotas() {
    return {
      emailsRemaining: MailApp.getRemainingDailyQuota(),
      // Add other quotas as needed
    };
  }
}

// Usage
function debugState() {
  StateInspector.inspect();
}
```

### Debug Mode

```javascript
// Global debug configuration
class Debug {
  static get enabled() {
    var props = PropertiesService.getScriptProperties();
    return props.getProperty('DEBUG_MODE') === 'true';
  }
  
  static set enabled(value) {
    var props = PropertiesService.getScriptProperties();
    props.setProperty('DEBUG_MODE', value.toString());
  }
  
  static log(...args) {
    if (this.enabled) {
      console.log('[DEBUG]', ...args);
    }
  }
  
  static time(label) {
    if (this.enabled) {
      console.time(`[DEBUG] ${label}`);
    }
  }
  
  static timeEnd(label) {
    if (this.enabled) {
      console.timeEnd(`[DEBUG] ${label}`);
    }
  }
  
  static assert(condition, message) {
    if (this.enabled && !condition) {
      throw new Error(`Assertion failed: ${message}`);
    }
  }
  
  static trace() {
    if (this.enabled) {
      console.trace('[DEBUG] Stack trace');
    }
  }
  
  static group(label) {
    if (this.enabled) {
      console.log(`[DEBUG GROUP] ${label}`);
    }
  }
  
  static table(data) {
    if (this.enabled) {
      console.log('[DEBUG] Table data:');
      console.log(JSON.stringify(data, null, 2));
    }
  }
}

// Usage
Debug.enabled = true;

function debuggableFunction() {
  Debug.group('Function Start');
  Debug.log('Input parameters:', arguments);
  
  Debug.time('Processing');
  
  var result = processData();
  Debug.assert(result != null, 'Result should not be null');
  
  Debug.timeEnd('Processing');
  Debug.log('Result:', result);
  
  return result;
}
```

### Performance Profiling

```javascript
// Performance profiler
class Profiler {
  constructor(name) {
    this.name = name;
    this.metrics = [];
    this.startTime = new Date().getTime();
  }
  
  mark(label) {
    var now = new Date().getTime();
    var elapsed = now - this.startTime;
    
    this.metrics.push({
      label: label,
      timestamp: now,
      elapsed: elapsed,
      delta: this.metrics.length > 0 
        ? elapsed - this.metrics[this.metrics.length - 1].elapsed 
        : elapsed
    });
    
    Debug.log(`${this.name} - ${label}: ${elapsed}ms`);
  }
  
  report() {
    console.log(`\n=== Performance Report: ${this.name} ===`);
    console.log(`Total time: ${new Date().getTime() - this.startTime}ms\n`);
    
    this.metrics.forEach((metric, index) => {
      console.log(`${index + 1}. ${metric.label}:`);
      console.log(`   Elapsed: ${metric.elapsed}ms`);
      console.log(`   Delta: ${metric.delta}ms`);
    });
    
    // Find bottlenecks
    var sorted = [...this.metrics].sort((a, b) => b.delta - a.delta);
    console.log('\nSlowest operations:');
    sorted.slice(0, 3).forEach(metric => {
      console.log(`- ${metric.label}: ${metric.delta}ms`);
    });
  }
}

// Usage
function profiledOperation() {
  var profiler = new Profiler('DataImport');
  
  profiler.mark('Start');
  
  var data = fetchData();
  profiler.mark('Data fetched');
  
  var validated = validateData(data);
  profiler.mark('Data validated');
  
  var processed = processData(validated);
  profiler.mark('Data processed');
  
  saveData(processed);
  profiler.mark('Data saved');
  
  profiler.report();
}
```

## Error Diagnosis

### Enhanced Error Information

```javascript
// Error wrapper with context
class DiagnosticError extends Error {
  constructor(message, code, context = {}) {
    super(message);
    this.name = 'DiagnosticError';
    this.code = code;
    this.context = context;
    this.timestamp = new Date();
    this.scriptId = ScriptApp.getScriptId();
    
    // Capture environment
    this.environment = {
      user: Session.getActiveUser().getEmail(),
      timezone: Session.getScriptTimeZone(),
      trigger: this.getTriggerInfo()
    };
  }
  
  getTriggerInfo() {
    try {
      // Attempt to determine if running from trigger
      var triggers = ScriptApp.getProjectTriggers();
      return {
        count: triggers.length,
        types: triggers.map(t => t.getEventType())
      };
    } catch (e) {
      return null;
    }
  }
  
  toJSON() {
    return {
      name: this.name,
      message: this.message,
      code: this.code,
      context: this.context,
      stack: this.stack,
      timestamp: this.timestamp,
      scriptId: this.scriptId,
      environment: this.environment
    };
  }
}

// Usage
function operationWithDiagnostics() {
  try {
    var result = riskyOperation();
    return result;
  } catch (error) {
    throw new DiagnosticError(
      'Operation failed: ' + error.message,
      'OP_001',
      {
        operation: 'riskyOperation',
        input: arguments,
        originalError: error.toString()
      }
    );
  }
}
```

### Debug Utilities

```javascript
// Debug utility functions
var DebugUtils = {
  // Pretty print objects
  pp: function(obj, indent = 2) {
    console.log(JSON.stringify(obj, null, indent));
  },
  
  // Type checking with logging
  assertType: function(value, expectedType, name) {
    var actualType = Array.isArray(value) ? 'array' : typeof value;
    
    if (actualType !== expectedType) {
      var error = `Type assertion failed for ${name}: expected ${expectedType}, got ${actualType}`;
      console.error(error);
      console.error('Value:', value);
      throw new Error(error);
    }
  },
  
  // Trace function calls
  trace: function(func, name) {
    return function() {
      console.log(`→ Entering ${name || func.name}`, arguments);
      var start = new Date().getTime();
      
      try {
        var result = func.apply(this, arguments);
        var duration = new Date().getTime() - start;
        console.log(`← Exiting ${name || func.name} (${duration}ms)`, result);
        return result;
      } catch (error) {
        console.error(`✗ Error in ${name || func.name}:`, error);
        throw error;
      }
    };
  },
  
  // Memory estimation
  sizeof: function(obj) {
    var size = 0;
    
    function calculate(obj) {
      if (obj === null) return 4;
      
      switch (typeof obj) {
        case 'number':
          return 8;
        case 'string':
          return obj.length * 2;
        case 'boolean':
          return 4;
        case 'object':
          if (Array.isArray(obj)) {
            return obj.reduce((sum, item) => sum + calculate(item), 0);
          } else {
            return Object.keys(obj).reduce((sum, key) => 
              sum + calculate(key) + calculate(obj[key]), 0
            );
          }
        default:
          return 0;
      }
    }
    
    size = calculate(obj);
    return {
      bytes: size,
      kb: (size / 1024).toFixed(2),
      mb: (size / 1024 / 1024).toFixed(2)
    };
  },
  
  // Watch object changes
  watch: function(obj, onChange) {
    return new Proxy(obj, {
      set: function(target, property, value) {
        var oldValue = target[property];
        target[property] = value;
        onChange(property, oldValue, value);
        return true;
      }
    });
  }
};

// Usage
var tracedFunction = DebugUtils.trace(processData, 'processData');
DebugUtils.assertType(data, 'array', 'data');
DebugUtils.pp(complexObject);
```

## Testing and Mocking

### Mock Objects

```javascript
// Mock Google Services for testing
class MockSpreadsheetApp {
  static sheets = new Map();
  
  static create(name) {
    var sheet = new MockSpreadsheet(name);
    this.sheets.set(sheet.getId(), sheet);
    return sheet;
  }
  
  static openById(id) {
    if (!this.sheets.has(id)) {
      throw new Error('Spreadsheet not found');
    }
    return this.sheets.get(id);
  }
}

class MockSpreadsheet {
  constructor(name) {
    this.id = Utilities.getUuid();
    this.name = name;
    this.sheets = [new MockSheet('Sheet1')];
  }
  
  getId() {
    return this.id;
  }
  
  getName() {
    return this.name;
  }
  
  getActiveSheet() {
    return this.sheets[0];
  }
}

class MockSheet {
  constructor(name) {
    this.name = name;
    this.data = [];
  }
  
  getRange(row, col, numRows, numCols) {
    return new MockRange(this, row, col, numRows, numCols);
  }
  
  appendRow(values) {
    this.data.push(values);
  }
}

class MockRange {
  constructor(sheet, row, col, numRows, numCols) {
    this.sheet = sheet;
    this.row = row;
    this.col = col;
    this.numRows = numRows;
    this.numCols = numCols;
  }
  
  getValues() {
    // Return mock data
    return Array(this.numRows).fill(null).map(() => 
      Array(this.numCols).fill('mock')
    );
  }
  
  setValues(values) {
    // Store mock data
    console.log('Mock setValues called:', values);
  }
}

// Test with mocks
function testWithMocks() {
  // Replace real service with mock
  var originalSpreadsheetApp = SpreadsheetApp;
  SpreadsheetApp = MockSpreadsheetApp;
  
  try {
    // Run tests
    var ss = SpreadsheetApp.create('Test');
    var sheet = ss.getActiveSheet();
    sheet.appendRow(['test', 'data']);
    
    // Assertions
    console.assert(ss.getName() === 'Test');
    
  } finally {
    // Restore original
    SpreadsheetApp = originalSpreadsheetApp;
  }
}
```

## Common Debugging Scenarios

### Debugging Triggers

```javascript
// Debug trigger execution
function debugTrigger(e) {
  // Log trigger event
  console.log('=== TRIGGER DEBUG ===');
  console.log('Trigger Type:', e?.triggerUid ? 'Installable' : 'Simple');
  console.log('Auth Mode:', e?.authMode);
  console.log('User:', e?.user);
  console.log('Event Object:', JSON.stringify(e, null, 2));
  
  // Create debug sheet
  var debugSheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName('Debug_Log') || 
    SpreadsheetApp.getActiveSpreadsheet()
    .insertSheet('Debug_Log');
  
  debugSheet.appendRow([
    new Date(),
    'Trigger',
    JSON.stringify(e),
    Session.getActiveUser().getEmail()
  ]);
}

// Test trigger without waiting
function testTriggerLocally() {
  var mockEvent = {
    authMode: ScriptApp.AuthMode.FULL,
    triggerUid: 'test-trigger',
    user: Session.getActiveUser()
  };
  
  debugTrigger(mockEvent);
}
```

### Debugging API Calls

```javascript
// API call debugger
class APIDebugger {
  static fetch(url, params = {}) {
    console.log('=== API REQUEST ===');
    console.log('URL:', url);
    console.log('Method:', params.method || 'GET');
    console.log('Headers:', params.headers);
    console.log('Payload:', params.payload);
    
    var startTime = new Date().getTime();
    
    try {
      var response = UrlFetchApp.fetch(url, params);
      var duration = new Date().getTime() - startTime;
      
      console.log('=== API RESPONSE ===');
      console.log('Status:', response.getResponseCode());
      console.log('Duration:', duration + 'ms');
      console.log('Headers:', response.getAllHeaders());
      
      var content = response.getContentText();
      
      try {
        console.log('Body:', JSON.parse(content));
      } catch (e) {
        console.log('Body (text):', content.substring(0, 200) + '...');
      }
      
      return response;
      
    } catch (error) {
      var duration = new Date().getTime() - startTime;
      
      console.error('=== API ERROR ===');
      console.error('Duration:', duration + 'ms');
      console.error('Error:', error.toString());
      
      throw error;
    }
  }
}

// Usage
var response = APIDebugger.fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  payload: JSON.stringify({key: 'value'})
});
```

## Best Practices

1. **Use descriptive log messages** with context
2. **Log at appropriate levels** (debug, info, warn, error)
3. **Include timestamps** in log entries
4. **Use structured logging** for complex data
5. **Clean up debug code** before production
6. **Set up error notifications** for production
7. **Use debug flags** to control output
8. **Profile performance** bottlenecks
9. **Test with mock data** when possible
10. **Document debugging procedures** for your team