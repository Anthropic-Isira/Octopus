# Common Errors in Google Apps Script

[← Back to Main Documentation](../README.md)

## Overview

This guide covers the most common errors you'll encounter in Google Apps Script development, their causes, and solutions.

> **Important**: For lesser-known behaviors and platform quirks that can cause unexpected issues, see the [Google Apps Script Nuances & Gotchas Guide](../core-concepts/gas-nuances-gotchas.md).

## Authorization Errors

### "Authorization is required to perform that action"

**Cause**: Script needs permission to access Google services or user data.

**Solutions**:
```javascript
// Check authorization mode
function checkAuth(e) {
  if (e && e.authMode === ScriptApp.AuthMode.NONE) {
    // Running without authorization
    return false;
  }
  return true;
}

// Request authorization
function showAuthPrompt() {
  var authInfo = ScriptApp.getAuthorizationInfo(ScriptApp.AuthMode.FULL);
  
  if (authInfo.getAuthorizationStatus() == 
      ScriptApp.AuthorizationStatus.NOT_REQUIRED) {
    // Already authorized
    return;
  }
  
  // Show authorization URL
  var template = HtmlService.createTemplate(
    'Please <a href="<?= authUrl ?>" target="_blank">authorize</a> this script.'
  );
  template.authUrl = authInfo.getAuthorizationUrl();
  
  SpreadsheetApp.getUi().showModalDialog(
    template.evaluate(),
    'Authorization Required'
  );
}
```

### "You do not have permission to access this resource"

**Cause**: Lack of access to specific file, folder, or resource.

**Solutions**:
```javascript
// Safe file access with permission check
function safeGetFile(fileId) {
  try {
    var file = DriveApp.getFileById(fileId);
    // Test access by trying to read
    file.getName();
    return file;
  } catch (e) {
    if (e.message.includes('not have permission')) {
      // Request access
      var url = 'https://drive.google.com/file/d/' + fileId;
      throw new Error('No access to file. Request access at: ' + url);
    }
    throw e;
  }
}

// Check folder access before operations
function checkFolderAccess(folderId) {
  try {
    var folder = DriveApp.getFolderById(folderId);
    // Try to list files to verify read access
    folder.getFiles().hasNext();
    
    // Try to create a test file to verify write access
    var testFile = folder.createFile('test', 'test');
    DriveApp.getFileById(testFile.getId()).setTrashed(true);
    
    return {read: true, write: true};
  } catch (e) {
    return {read: false, write: false, error: e.message};
  }
}
```

## Service Quota Errors

### "Service invoked too many times"

**Cause**: Exceeded daily quota for service (email, URL fetch, etc.)

**Solutions**:
```javascript
// Email quota management
class EmailQuotaManager {
  constructor() {
    this.props = PropertiesService.getScriptProperties();
  }
  
  canSendEmail(count = 1) {
    var remaining = MailApp.getRemainingDailyQuota();
    return remaining >= count;
  }
  
  sendEmailSafely(to, subject, body) {
    if (!this.canSendEmail()) {
      // Queue for later
      this.queueEmail(to, subject, body);
      throw new Error('Email quota exhausted. Email queued for tomorrow.');
    }
    
    try {
      MailApp.sendEmail(to, subject, body);
      this.updateSentCount();
    } catch (e) {
      if (e.message.includes('too many times')) {
        this.queueEmail(to, subject, body);
        throw new Error('Email quota exceeded. Email queued.');
      }
      throw e;
    }
  }
  
  queueEmail(to, subject, body) {
    var queue = JSON.parse(this.props.getProperty('emailQueue') || '[]');
    queue.push({
      to: to,
      subject: subject,
      body: body,
      queued: new Date().toISOString()
    });
    this.props.setProperty('emailQueue', JSON.stringify(queue));
  }
  
  processQueue() {
    var queue = JSON.parse(this.props.getProperty('emailQueue') || '[]');
    var sent = 0;
    
    while (queue.length > 0 && this.canSendEmail()) {
      var email = queue.shift();
      try {
        MailApp.sendEmail(email.to, email.subject, email.body);
        sent++;
      } catch (e) {
        // Re-queue on failure
        queue.unshift(email);
        break;
      }
    }
    
    this.props.setProperty('emailQueue', JSON.stringify(queue));
    return sent;
  }
}

// URL Fetch quota management
function fetchWithQuotaCheck(url, params = {}) {
  var cache = CacheService.getScriptCache();
  var quotaKey = 'urlFetchCount_' + new Date().toDateString();
  var count = parseInt(cache.get(quotaKey) || '0');
  
  // Approximate daily limit
  var URL_FETCH_DAILY_LIMIT = 20000;
  
  if (count >= URL_FETCH_DAILY_LIMIT) {
    throw new Error('URL Fetch quota likely exceeded');
  }
  
  try {
    var response = UrlFetchApp.fetch(url, params);
    cache.put(quotaKey, (count + 1).toString(), 86400);
    return response;
  } catch (e) {
    if (e.message.includes('too many times')) {
      cache.put(quotaKey, URL_FETCH_DAILY_LIMIT.toString(), 86400);
      throw new Error('URL Fetch quota exceeded');
    }
    throw e;
  }
}
```

### "Too many simultaneous invocations"

**Cause**: Too many triggers or users running script at once.

**Solutions**:
```javascript
// Concurrent execution limiter
class ExecutionLimiter {
  constructor(maxConcurrent = 20) {
    this.maxConcurrent = maxConcurrent;
    this.props = PropertiesService.getScriptProperties();
  }
  
  canExecute() {
    var lock = LockService.getScriptLock();
    
    try {
      lock.waitLock(1000);
      
      var current = parseInt(this.props.getProperty('currentExecutions') || '0');
      
      if (current >= this.maxConcurrent) {
        return false;
      }
      
      this.props.setProperty('currentExecutions', (current + 1).toString());
      return true;
      
    } catch (e) {
      return false;
    } finally {
      lock.releaseLock();
    }
  }
  
  release() {
    var lock = LockService.getScriptLock();
    
    try {
      lock.waitLock(1000);
      
      var current = parseInt(this.props.getProperty('currentExecutions') || '1');
      this.props.setProperty('currentExecutions', Math.max(0, current - 1).toString());
      
    } catch (e) {
      // Silent fail
    } finally {
      lock.releaseLock();
    }
  }
  
  execute(func) {
    if (!this.canExecute()) {
      throw new Error('Too many simultaneous executions. Please try again later.');
    }
    
    try {
      return func();
    } finally {
      this.release();
    }
  }
}

// Usage
function limitedFunction() {
  var limiter = new ExecutionLimiter(10);
  
  return limiter.execute(function() {
    // Your code here
    return processData();
  });
}
```

## Timeout Errors

### "Exceeded maximum execution time"

**Cause**: Script ran longer than 6 minutes (30 minutes for Workspace accounts).

**Solutions**:
```javascript
// Batch processing with resume
class BatchProcessor {
  constructor(jobId) {
    this.jobId = jobId;
    this.props = PropertiesService.getScriptProperties();
    this.startTime = new Date().getTime();
    this.maxRuntime = 5 * 60 * 1000; // 5 minutes (leaving buffer)
  }
  
  process(items, processFunc) {
    var state = this.loadState();
    var startIndex = state.lastProcessed + 1;
    
    console.log(`Resuming from index ${startIndex} of ${items.length}`);
    
    for (var i = startIndex; i < items.length; i++) {
      // Check time
      if (this.shouldStop()) {
        this.saveState({
          lastProcessed: i - 1,
          total: items.length,
          status: 'paused'
        });
        
        this.scheduleResume();
        console.log(`Paused at index ${i}. Will resume.`);
        return false;
      }
      
      try {
        processFunc(items[i], i);
        
        // Save progress periodically
        if (i % 10 === 0) {
          this.saveState({
            lastProcessed: i,
            total: items.length,
            status: 'processing'
          });
        }
        
      } catch (error) {
        console.error(`Error processing item ${i}:`, error);
        // Continue with next item
      }
    }
    
    // Completed
    this.saveState({
      lastProcessed: items.length - 1,
      total: items.length,
      status: 'completed'
    });
    
    return true;
  }
  
  shouldStop() {
    var elapsed = new Date().getTime() - this.startTime;
    return elapsed > this.maxRuntime;
  }
  
  loadState() {
    var stateKey = 'batch_' + this.jobId;
    var state = this.props.getProperty(stateKey);
    
    return state ? JSON.parse(state) : {
      lastProcessed: -1,
      total: 0,
      status: 'new'
    };
  }
  
  saveState(state) {
    var stateKey = 'batch_' + this.jobId;
    this.props.setProperty(stateKey, JSON.stringify(state));
  }
  
  scheduleResume() {
    // Create trigger to resume in 1 minute
    ScriptApp.newTrigger('resumeBatchJob')
      .timeBased()
      .after(60 * 1000)
      .create();
  }
}

// Resume function
function resumeBatchJob() {
  var props = PropertiesService.getScriptProperties();
  var allProps = props.getProperties();
  
  // Find paused jobs
  for (var key in allProps) {
    if (key.startsWith('batch_')) {
      var state = JSON.parse(allProps[key]);
      
      if (state.status === 'paused') {
        var jobId = key.substring(6);
        console.log('Resuming job:', jobId);
        
        // Resume processing
        continueBatchProcessing(jobId);
      }
    }
  }
}
```

## Type Errors

### "Cannot read property 'x' of null/undefined"

**Cause**: Attempting to access property of null or undefined value.

**Solutions**:
```javascript
// Safe property access
function safeAccess(obj, path, defaultValue = null) {
  return path.split('.').reduce((current, property) => 
    current?.[property], obj) ?? defaultValue;
}

// Usage
var user = {
  profile: {
    name: 'John',
    address: null
  }
};

// Instead of: user.profile.address.street (throws error)
var street = safeAccess(user, 'profile.address.street', 'Unknown');

// Null-safe spreadsheet operations
function safeGetRange(sheet, row, col, numRows = 1, numCols = 1) {
  if (!sheet) {
    throw new Error('Sheet is null or undefined');
  }
  
  // Validate bounds
  var maxRows = sheet.getMaxRows();
  var maxCols = sheet.getMaxColumns();
  
  if (row < 1 || row > maxRows) {
    throw new Error(`Row ${row} out of bounds (1-${maxRows})`);
  }
  
  if (col < 1 || col > maxCols) {
    throw new Error(`Column ${col} out of bounds (1-${maxCols})`);
  }
  
  // Adjust size if necessary
  numRows = Math.min(numRows, maxRows - row + 1);
  numCols = Math.min(numCols, maxCols - col + 1);
  
  return sheet.getRange(row, col, numRows, numCols);
}

// Defensive array access
function safeArrayAccess(array, index, defaultValue = null) {
  if (!Array.isArray(array)) {
    console.warn('Not an array:', array);
    return defaultValue;
  }
  
  if (index < 0 || index >= array.length) {
    console.warn(`Index ${index} out of bounds for array length ${array.length}`);
    return defaultValue;
  }
  
  return array[index] ?? defaultValue;
}
```

### "Cannot convert Array to Object[][]"

**Cause**: Incorrect data structure for setValues().

**Solutions**:
```javascript
// Ensure 2D array for setValues
function ensure2DArray(data) {
  if (!Array.isArray(data)) {
    return [[data]];
  }
  
  if (data.length === 0) {
    return [[]];
  }
  
  // Check if already 2D
  if (Array.isArray(data[0])) {
    return data;
  }
  
  // Convert 1D to 2D
  return data.map(item => [item]);
}

// Safe setValues
function safeSetValues(range, values) {
  var data = ensure2DArray(values);
  
  // Validate dimensions
  var rangeRows = range.getNumRows();
  var rangeCols = range.getNumColumns();
  var dataRows = data.length;
  var dataCols = data[0]?.length || 0;
  
  if (dataRows !== rangeRows || dataCols !== rangeCols) {
    throw new Error(
      `Data dimensions (${dataRows}x${dataCols}) don't match ` +
      `range dimensions (${rangeRows}x${rangeCols})`
    );
  }
  
  range.setValues(data);
}

// Convert objects to 2D array
function objectsTo2DArray(objects, headers) {
  if (!Array.isArray(objects) || objects.length === 0) {
    return [headers || []];
  }
  
  // Auto-detect headers if not provided
  if (!headers) {
    headers = Object.keys(objects[0]);
  }
  
  var data = objects.map(obj => 
    headers.map(header => obj[header] ?? '')
  );
  
  // Include headers
  return [headers, ...data];
}
```

## Range and Sheet Errors

### "Range not found"

**Cause**: Invalid range reference or coordinates.

**Solutions**:
```javascript
// Validate range references
function getValidRange(sheet, rangeA1) {
  try {
    return sheet.getRange(rangeA1);
  } catch (e) {
    if (e.message.includes('Range not found')) {
      // Try to parse and fix
      var match = rangeA1.match(/([A-Z]+)(\d+)(?::([A-Z]+)(\d+))?/);
      
      if (!match) {
        throw new Error('Invalid range notation: ' + rangeA1);
      }
      
      // Ensure within sheet bounds
      var maxRows = sheet.getMaxRows();
      var maxCols = sheet.getMaxColumns();
      
      var startRow = Math.min(parseInt(match[2]), maxRows);
      var endRow = match[4] ? Math.min(parseInt(match[4]), maxRows) : startRow;
      
      return sheet.getRange(startRow, 
        columnToNumber(match[1]), 
        endRow - startRow + 1,
        match[3] ? columnToNumber(match[3]) - columnToNumber(match[1]) + 1 : 1
      );
    }
    throw e;
  }
}

function columnToNumber(column) {
  var result = 0;
  for (var i = 0; i < column.length; i++) {
    result = result * 26 + column.charCodeAt(i) - 64;
  }
  return result;
}

// Dynamic range sizing
function getDataRange(sheet, startRow = 1, startCol = 1) {
  var lastRow = sheet.getLastRow();
  var lastCol = sheet.getLastColumn();
  
  if (lastRow < startRow || lastCol < startCol) {
    return null; // No data
  }
  
  return sheet.getRange(
    startRow, 
    startCol, 
    lastRow - startRow + 1, 
    lastCol - startCol + 1
  );
}
```

### "Sheet not found"

**Cause**: Referencing non-existent sheet.

**Solutions**:
```javascript
// Safe sheet access
function getOrCreateSheet(spreadsheet, sheetName) {
  var sheet = spreadsheet.getSheetByName(sheetName);
  
  if (!sheet) {
    console.log('Creating sheet:', sheetName);
    sheet = spreadsheet.insertSheet(sheetName);
    
    // Set up headers or initial structure
    setupSheet(sheet, sheetName);
  }
  
  return sheet;
}

function setupSheet(sheet, sheetName) {
  // Define sheet structures
  var structures = {
    'Data': ['ID', 'Name', 'Date', 'Value'],
    'Log': ['Timestamp', 'User', 'Action', 'Details'],
    'Config': ['Key', 'Value', 'Description']
  };
  
  var headers = structures[sheetName];
  if (headers) {
    sheet.getRange(1, 1, 1, headers.length).setValues([headers]);
    sheet.getRange(1, 1, 1, headers.length).setFontWeight('bold');
  }
}

// Sheet existence validator
function validateSheets(spreadsheet, requiredSheets) {
  var missing = [];
  var existing = spreadsheet.getSheets().map(s => s.getName());
  
  requiredSheets.forEach(sheetName => {
    if (!existing.includes(sheetName)) {
      missing.push(sheetName);
    }
  });
  
  if (missing.length > 0) {
    throw new Error('Missing required sheets: ' + missing.join(', '));
  }
}
```

## API and External Service Errors

### "Request failed with code 4xx/5xx"

**Cause**: External API request failed.

**Solutions**:
```javascript
// Comprehensive API error handling
class APIClient {
  constructor(baseUrl, options = {}) {
    this.baseUrl = baseUrl;
    this.headers = options.headers || {};
    this.timeout = options.timeout || 30000;
    this.retries = options.retries || 3;
  }
  
  request(endpoint, options = {}) {
    var url = this.baseUrl + endpoint;
    var params = {
      headers: {...this.headers, ...options.headers},
      method: options.method || 'GET',
      muteHttpExceptions: true,
      timeout: this.timeout
    };
    
    if (options.payload) {
      params.payload = typeof options.payload === 'string' 
        ? options.payload 
        : JSON.stringify(options.payload);
      params.contentType = options.contentType || 'application/json';
    }
    
    var lastError;
    
    for (var i = 0; i < this.retries; i++) {
      try {
        var response = UrlFetchApp.fetch(url, params);
        var code = response.getResponseCode();
        
        if (code >= 200 && code < 300) {
          return JSON.parse(response.getContentText());
        }
        
        var error = this.handleHTTPError(code, response);
        
        if (!this.isRetryable(code)) {
          throw error;
        }
        
        lastError = error;
        
        // Exponential backoff
        if (i < this.retries - 1) {
          Utilities.sleep(Math.pow(2, i) * 1000);
        }
        
      } catch (e) {
        if (e.message.includes('DNS error')) {
          throw new Error('Network error: Cannot reach ' + this.baseUrl);
        }
        lastError = e;
      }
    }
    
    throw lastError;
  }
  
  handleHTTPError(code, response) {
    var body = response.getContentText();
    var message = `HTTP ${code}: `;
    
    try {
      var errorData = JSON.parse(body);
      message += errorData.error || errorData.message || body;
    } catch (e) {
      message += body;
    }
    
    var error = new Error(message);
    error.code = code;
    error.response = body;
    
    return error;
  }
  
  isRetryable(code) {
    return code === 429 || code === 503 || code >= 500;
  }
}

// Usage with specific error handling
function callAPI() {
  var client = new APIClient('https://api.example.com');
  
  try {
    return client.request('/data');
  } catch (error) {
    if (error.code === 401) {
      // Refresh auth token
      refreshAuthToken();
      // Retry once
      return client.request('/data');
    } else if (error.code === 429) {
      // Rate limited
      var retryAfter = error.response.headers?.['Retry-After'] || 60;
      throw new Error(`Rate limited. Retry after ${retryAfter} seconds`);
    } else if (error.code === 404) {
      // Not found
      return null;
    } else {
      throw error;
    }
  }
}
```

## Memory and Performance Errors

### "Out of memory"

**Cause**: Processing too much data at once.

**Solutions**:
```javascript
// Memory-efficient data processing
function processLargeDataset(sheet) {
  var batchSize = 1000;
  var startRow = 2; // Skip header
  var lastRow = sheet.getLastRow();
  var results = [];
  
  while (startRow <= lastRow) {
    var numRows = Math.min(batchSize, lastRow - startRow + 1);
    var batch = sheet.getRange(startRow, 1, numRows, sheet.getLastColumn())
      .getValues();
    
    // Process batch
    var batchResults = processBatch(batch);
    
    // Store results efficiently
    if (batchResults.length > 0) {
      writeResults(batchResults);
    }
    
    // Clear batch from memory
    batch = null;
    batchResults = null;
    
    startRow += batchSize;
    
    // Optional: Force garbage collection pause
    Utilities.sleep(100);
  }
}

// Streaming approach
function* streamSheetData(sheet, batchSize = 100) {
  var row = 2;
  var lastRow = sheet.getLastRow();
  
  while (row <= lastRow) {
    var numRows = Math.min(batchSize, lastRow - row + 1);
    var data = sheet.getRange(row, 1, numRows, sheet.getLastColumn())
      .getValues();
    
    yield data;
    
    row += numRows;
  }
}

// Usage
function processWithStreaming() {
  var sheet = SpreadsheetApp.getActiveSheet();
  
  for (var batch of streamSheetData(sheet)) {
    // Process each batch
    batch.forEach(row => {
      // Process row
    });
    
    // Batch is garbage collected after each iteration
  }
}
```

## Best Practices for Error Prevention

1. **Always validate inputs** before processing
2. **Use try-catch blocks** around risky operations
3. **Check service quotas** before bulk operations
4. **Implement proper retry logic** with backoff
5. **Log errors comprehensively** for debugging
6. **Test edge cases** and error conditions
7. **Monitor script performance** and quotas
8. **Use batch processing** for large datasets
9. **Implement circuit breakers** for external services
10. **Document known issues** and solutions

## Error Recovery Checklist

1. **Identify the error type** from the message
2. **Check quotas and limits** in Google Cloud Console
3. **Verify permissions** for all resources
4. **Review recent changes** that might have caused the issue
5. **Test with smaller data sets** to isolate the problem
6. **Check external service status** if applicable
7. **Review execution logs** for patterns
8. **Implement proper error handling** to prevent recurrence
9. **Add monitoring** for critical operations
10. **Document the solution** for future reference