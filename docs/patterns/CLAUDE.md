# Google Apps Script Architecture Patterns

## Code Organization Patterns

### Modular Service Pattern
```javascript
// Services/SheetService.gs
var SheetService = {
  getSheet: function(name) {
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    return ss.getSheetByName(name) || ss.insertSheet(name);
  },
  
  writeData: function(sheetName, data) {
    var sheet = this.getSheet(sheetName);
    sheet.clear();
    sheet.getRange(1, 1, data.length, data[0].length).setValues(data);
  }
};

// Services/EmailService.gs  
var EmailService = {
  sendNotification: function(subject, body) {
    var email = Session.getActiveUser().getEmail();
    GmailApp.sendEmail(email, subject, body);
  }
};
```

### Configuration Pattern
```javascript
// Config.gs
var Config = {
  SHEET_ID: PropertiesService.getScriptProperties().getProperty('SHEET_ID'),
  EMAIL_QUOTA_BUFFER: 50,
  BATCH_SIZE: 1000,
  
  get: function(key) {
    return this[key] || PropertiesService.getScriptProperties().getProperty(key);
  },
  
  set: function(key, value) {
    PropertiesService.getScriptProperties().setProperty(key, value);
    this[key] = value;
  }
};
```

### Repository Pattern
```javascript
// Repositories/UserRepository.gs
var UserRepository = {
  sheet: null,
  
  init: function() {
    this.sheet = SpreadsheetApp.openById(Config.SHEET_ID)
      .getSheetByName('Users');
  },
  
  findById: function(id) {
    var data = this.sheet.getDataRange().getValues();
    var headers = data[0];
    var idIndex = headers.indexOf('id');
    
    for (var i = 1; i < data.length; i++) {
      if (data[i][idIndex] === id) {
        return this.rowToObject(headers, data[i]);
      }
    }
    return null;
  },
  
  save: function(user) {
    // Implementation
  },
  
  rowToObject: function(headers, row) {
    var obj = {};
    headers.forEach((header, i) => obj[header] = row[i]);
    return obj;
  }
};
```

## Error Handling Patterns

### Circuit Breaker Pattern
```javascript
var CircuitBreaker = {
  failures: {},
  threshold: 5,
  timeout: 60000, // 1 minute
  
  call: function(serviceName, func) {
    var state = this.getState(serviceName);
    
    if (state.isOpen) {
      if (Date.now() - state.lastFailure < this.timeout) {
        throw new Error('Circuit breaker is OPEN for ' + serviceName);
      }
      // Try to half-open
      state.isOpen = false;
    }
    
    try {
      var result = func();
      this.onSuccess(serviceName);
      return result;
    } catch (e) {
      this.onFailure(serviceName);
      throw e;
    }
  },
  
  onFailure: function(serviceName) {
    var state = this.failures[serviceName] || {count: 0};
    state.count++;
    state.lastFailure = Date.now();
    
    if (state.count >= this.threshold) {
      state.isOpen = true;
      console.error('Circuit breaker OPENED for ' + serviceName);
    }
    
    this.failures[serviceName] = state;
  },
  
  onSuccess: function(serviceName) {
    delete this.failures[serviceName];
  },
  
  getState: function(serviceName) {
    return this.failures[serviceName] || {count: 0, isOpen: false};
  }
};
```

### Retry Pattern
```javascript
var RetryHandler = {
  execute: function(func, options) {
    options = options || {};
    var maxRetries = options.maxRetries || 3;
    var backoff = options.backoff || 1000;
    var lastError;
    
    for (var i = 0; i < maxRetries; i++) {
      try {
        return func();
      } catch (e) {
        lastError = e;
        console.log('Attempt ' + (i + 1) + ' failed:', e.toString());
        
        if (i < maxRetries - 1) {
          Utilities.sleep(backoff * Math.pow(2, i));
        }
      }
    }
    
    throw lastError;
  }
};
```

## State Management Patterns

### State Manager
```javascript
var StateManager = {
  cache: CacheService.getScriptCache(),
  props: PropertiesService.getScriptProperties(),
  
  get: function(key) {
    // Try cache first
    var cached = this.cache.get(key);
    if (cached) return JSON.parse(cached);
    
    // Fall back to properties
    var stored = this.props.getProperty(key);
    if (stored) {
      var parsed = JSON.parse(stored);
      // Repopulate cache
      this.cache.put(key, stored, 21600);
      return parsed;
    }
    
    return null;
  },
  
  set: function(key, value) {
    var serialized = JSON.stringify(value);
    
    // Store in both cache and properties
    this.cache.put(key, serialized, 21600);
    this.props.setProperty(key, serialized);
  },
  
  delete: function(key) {
    this.cache.remove(key);
    this.props.deleteProperty(key);
  }
};
```

## Testing Patterns

### Test Framework
```javascript
var TestRunner = {
  tests: [],
  results: [],
  
  describe: function(description, testFunc) {
    this.tests.push({
      description: description,
      func: testFunc
    });
  },
  
  it: function(description, assertion) {
    return {
      description: description,
      assertion: assertion
    };
  },
  
  run: function() {
    this.results = [];
    
    this.tests.forEach(test => {
      try {
        test.func();
        this.results.push({
          description: test.description,
          status: 'PASSED'
        });
      } catch (e) {
        this.results.push({
          description: test.description,
          status: 'FAILED',
          error: e.toString()
        });
      }
    });
    
    this.report();
  },
  
  report: function() {
    console.log('Test Results:');
    this.results.forEach(result => {
      console.log(result.status + ': ' + result.description);
      if (result.error) console.log('  Error: ' + result.error);
    });
  }
};

// Usage
TestRunner.describe('UserRepository', function() {
  var user = UserRepository.findById('123');
  if (!user) throw new Error('User not found');
  if (user.id !== '123') throw new Error('Wrong user returned');
});
```

## Trigger Management Pattern
```javascript
var TriggerManager = {
  install: function() {
    this.removeAll();
    
    // Time-based trigger
    ScriptApp.newTrigger('dailyJob')
      .timeBased()
      .everyDays(1)
      .atHour(2)
      .create();
    
    // Spreadsheet trigger
    ScriptApp.newTrigger('onSheetEdit')
      .forSpreadsheet(SpreadsheetApp.getActive())
      .onEdit()
      .create();
  },
  
  removeAll: function() {
    var triggers = ScriptApp.getProjectTriggers();
    triggers.forEach(trigger => ScriptApp.deleteTrigger(trigger));
  },
  
  listAll: function() {
    return ScriptApp.getProjectTriggers().map(trigger => ({
      function: trigger.getHandlerFunction(),
      type: trigger.getEventType(),
      id: trigger.getUniqueId()
    }));
  }
};
```

## Best Practices Summary

1. **Separate concerns** - Use service/repository patterns
2. **Centralize configuration** - Single Config object
3. **Handle errors gracefully** - Circuit breakers, retries
4. **Manage state properly** - Cache + Properties hybrid
5. **Make code testable** - Dependency injection
6. **Document patterns** - Clear naming conventions
7. **Version your APIs** - For web apps/libraries
8. **Monitor performance** - Add timing logs
9. **Plan for scale** - Batch operations from start
10. **Keep it simple** - Don't over-engineer

## When to Use Each Pattern

| Pattern | Use When |
|---------|----------|
| Service Pattern | Organizing related functionality |
| Repository Pattern | Managing data access |
| Circuit Breaker | Calling unreliable external services |
| Retry Pattern | Handling transient failures |
| State Manager | Persisting data between executions |
| Test Framework | Building reliable scripts |