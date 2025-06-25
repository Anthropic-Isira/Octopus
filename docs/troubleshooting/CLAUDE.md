# Google Apps Script Error Handling Guide

## Common Authorization Errors

### "Authorization is required to perform that action"
- Script needs permission to access Google services
- Solution: Run script manually first to trigger auth dialog
- For triggers: Use installable triggers with proper auth mode

### "You do not have permission to access this resource"
```javascript
// Always check access before operations
function safeFileAccess(fileId) {
  try {
    var file = DriveApp.getFileById(fileId);
    file.getName(); // Test access
    return file;
  } catch (e) {
    if (e.message.includes('not have permission')) {
      var url = 'https://drive.google.com/file/d/' + fileId;
      throw new Error('No access. Request at: ' + url);
    }
    throw e;
  }
}
```

## Service Quota Errors

### "Service invoked too many times"
```javascript
// Check quotas before operations
if (!MailApp.getRemainingDailyQuota() >= neededEmails) {
  queueForTomorrow();
  return;
}

// Implement quota tracking
var URL_FETCH_DAILY_LIMIT = 20000;
var count = cache.get('urlFetchCount') || 0;
if (count >= URL_FETCH_DAILY_LIMIT) {
  throw new Error('URL Fetch quota exceeded');
}
```

### "Too many simultaneous invocations"
- Limit: ~20-30 concurrent executions
- Use Lock Service to control concurrency
- Implement execution limiting:
```javascript
if (!canExecute()) {
  throw new Error('Too many simultaneous executions');
}
```

## Timeout Errors

### "Exceeded maximum execution time"
```javascript
// Batch processing pattern
class BatchProcessor {
  constructor(jobId) {
    this.maxRuntime = 5 * 60 * 1000; // 5 min buffer
    this.startTime = new Date().getTime();
  }
  
  shouldStop() {
    return (new Date().getTime() - this.startTime) > this.maxRuntime;
  }
  
  process(items) {
    var state = this.loadState();
    for (var i = state.lastProcessed + 1; i < items.length; i++) {
      if (this.shouldStop()) {
        this.saveState({lastProcessed: i - 1});
        this.scheduleResume();
        return false;
      }
      processItem(items[i]);
    }
    return true;
  }
}
```

## Type Errors

### "Cannot read property 'x' of null/undefined"
```javascript
// Safe property access
function safeAccess(obj, path, defaultValue = null) {
  return path.split('.').reduce((curr, prop) => 
    curr?.[prop], obj) ?? defaultValue;
}

// Defensive array access  
function safeArrayAccess(array, index, defaultValue = null) {
  if (!Array.isArray(array) || index < 0 || index >= array.length) {
    return defaultValue;
  }
  return array[index] ?? defaultValue;
}
```

### "Cannot convert Array to Object[][]"
```javascript
// Ensure 2D array for setValues
function ensure2DArray(data) {
  if (!Array.isArray(data)) return [[data]];
  if (data.length === 0) return [[]];
  if (Array.isArray(data[0])) return data;
  return data.map(item => [item]);
}
```

## Range & Sheet Errors

### "Range not found"
```javascript
// Validate ranges before use
function getValidRange(sheet, rangeA1) {
  try {
    return sheet.getRange(rangeA1);
  } catch (e) {
    // Adjust to sheet bounds
    var maxRows = sheet.getMaxRows();
    var maxCols = sheet.getMaxColumns();
    // Parse and fix range...
  }
}
```

### "Sheet not found"
```javascript
// Get or create pattern
function getOrCreateSheet(spreadsheet, sheetName) {
  var sheet = spreadsheet.getSheetByName(sheetName);
  if (!sheet) {
    sheet = spreadsheet.insertSheet(sheetName);
    setupSheetHeaders(sheet);
  }
  return sheet;
}
```

## API Request Errors

### HTTP 4xx/5xx Errors
```javascript
class APIClient {
  request(endpoint, options = {}) {
    var params = {
      muteHttpExceptions: true,
      ...options
    };
    
    for (var i = 0; i < this.retries; i++) {
      var response = UrlFetchApp.fetch(url, params);
      var code = response.getResponseCode();
      
      if (code >= 200 && code < 300) {
        return JSON.parse(response.getContentText());
      }
      
      if (!this.isRetryable(code)) {
        throw new Error(`HTTP ${code}: ${response.getContentText()}`);
      }
      
      // Exponential backoff
      Utilities.sleep(Math.pow(2, i) * 1000);
    }
  }
  
  isRetryable(code) {
    return code === 429 || code === 503 || code >= 500;
  }
}
```

## Memory Errors

### "Out of memory"
```javascript
// Process in chunks
function processLargeDataset(sheet) {
  var batchSize = 1000;
  var row = 2;
  
  while (row <= sheet.getLastRow()) {
    var batch = sheet.getRange(row, 1, batchSize, sheet.getLastColumn())
      .getValues();
    
    processBatch(batch);
    
    // Clear memory
    batch = null;
    Utilities.sleep(100);
    
    row += batchSize;
  }
}
```

## Error Prevention Checklist

1. **Validate all inputs** before processing
2. **Check service quotas** before bulk operations  
3. **Implement retry logic** with exponential backoff
4. **Use batch processing** for large datasets
5. **Add circuit breakers** for external services
6. **Log errors comprehensively** with context
7. **Test edge cases** thoroughly
8. **Monitor execution time** continuously
9. **Handle null/undefined** defensively
10. **Document known issues** in code

## Related Documentation

- `gas-nuances-gotchas.md` - Platform limitations causing errors
- `performance-guide.md` - Optimization to prevent timeouts
- `quota-limits.md` - Service limits and management