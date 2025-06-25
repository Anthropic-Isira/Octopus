# Performance and Limits in Google Apps Script

[← Back to Main Documentation](../README.md)

## Overview

Understanding Apps Script's performance characteristics and limits is crucial for building reliable applications. This guide covers execution limits, quotas, and optimization strategies.

## Execution Time Limits

### Maximum Execution Time

| Account Type | Single Execution | Total Daily |
|--------------|------------------|-------------|
| Consumer (gmail.com) | 6 minutes | 90 minutes |
| Google Workspace | 30 minutes | 6 hours |

```javascript
// Monitor execution time
function timeAwareFunction() {
  var startTime = new Date().getTime();
  var maxRuntime = 5 * 60 * 1000; // 5 minutes (leaving buffer)
  
  for (var i = 0; i < largeDataset.length; i++) {
    // Process item
    processItem(largeDataset[i]);
    
    // Check if approaching time limit
    if (new Date().getTime() - startTime > maxRuntime) {
      console.log('Approaching time limit. Processed ' + i + ' items.');
      
      // Save progress
      PropertiesService.getScriptProperties()
        .setProperty('lastProcessedIndex', i.toString());
      
      // Schedule continuation
      ScriptApp.newTrigger('continueProcessing')
        .timeBased()
        .after(1000)
        .create();
      
      break;
    }
  }
}

function continueProcessing() {
  var lastIndex = parseInt(
    PropertiesService.getScriptProperties()
      .getProperty('lastProcessedIndex') || '0'
  );
  
  // Resume from last position
  processDataset(lastIndex);
}
```

## Service Quotas

### Email Service Quotas

| Operation | Consumer | Google Workspace |
|-----------|----------|------------------|
| Recipients per day | 100 | 1,500 |
| Recipients per message | 50 | 50 |
| Total emails per day | 100 | 1,500 |

```javascript
// Track email quota usage
function emailWithQuotaCheck(recipients, subject, body) {
  var quotaKey = 'emails_sent_' + new Date().toDateString();
  var props = PropertiesService.getUserProperties();
  var sentToday = parseInt(props.getProperty(quotaKey) || '0');
  
  var dailyLimit = Session.getActiveUser().getEmail().includes('gmail.com') 
    ? 100 : 1500;
  
  if (sentToday + recipients.length > dailyLimit) {
    throw new Error('Email quota exceeded. Sent: ' + sentToday + 
                    ', Limit: ' + dailyLimit);
  }
  
  try {
    MailApp.sendEmail({
      to: recipients.join(','),
      subject: subject,
      body: body
    });
    
    props.setProperty(quotaKey, (sentToday + recipients.length).toString());
    
  } catch (error) {
    if (error.toString().includes('Service invoked too many times')) {
      console.error('Email quota exceeded');
    }
    throw error;
  }
}
```

### URL Fetch Quotas

| Operation | Consumer | Google Workspace |
|-----------|----------|------------------|
| URL Fetch calls per day | 20,000 | 100,000 |
| URL Fetch response size | 50 MB | 50 MB |

```javascript
// Manage URL Fetch with quotas
class UrlFetchManager {
  constructor() {
    this.dailyLimit = 20000; // Conservative estimate
    this.cache = CacheService.getScriptCache();
  }
  
  fetch(url, params) {
    // Check cache first
    var cacheKey = 'url_' + Utilities.base64Encode(url);
    var cached = this.cache.get(cacheKey);
    
    if (cached) {
      return JSON.parse(cached);
    }
    
    // Check quota
    var quotaKey = 'url_fetches_' + new Date().toDateString();
    var props = PropertiesService.getScriptProperties();
    var fetchesToday = parseInt(props.getProperty(quotaKey) || '0');
    
    if (fetchesToday >= this.dailyLimit) {
      throw new Error('URL Fetch quota exceeded');
    }
    
    try {
      var response = UrlFetchApp.fetch(url, params);
      var content = response.getContentText();
      
      // Cache successful responses
      this.cache.put(cacheKey, content, 3600); // 1 hour
      
      // Update quota counter
      props.setProperty(quotaKey, (fetchesToday + 1).toString());
      
      return content;
      
    } catch (error) {
      console.error('URL Fetch failed:', error);
      throw error;
    }
  }
}
```

### Drive Quotas

| Operation | Limit |
|-----------|-------|
| Files created per day | 250,000 |
| Files trashed per day | 250,000 |
| Total uploads per day | 750 GB |
| Individual file upload | 5 TB |

```javascript
// Batch Drive operations to avoid limits
function batchDriveOperations(files) {
  var batchSize = 100;
  var totalCreated = 0;
  
  for (var i = 0; i < files.length; i += batchSize) {
    var batch = files.slice(i, i + batchSize);
    
    try {
      batch.forEach(function(fileData) {
        DriveApp.createFile(fileData.name, fileData.content);
        totalCreated++;
      });
      
      // Pause between batches
      if (i + batchSize < files.length) {
        Utilities.sleep(1000);
      }
      
    } catch (error) {
      if (error.toString().includes('Quota exceeded')) {
        console.error('Drive quota exceeded after ' + totalCreated + ' files');
        break;
      }
      throw error;
    }
  }
  
  return totalCreated;
}
```

### Spreadsheet Limits

| Limit | Value |
|-------|-------|
| Cells per spreadsheet | 10 million |
| Columns per sheet | 18,278 |
| Rows per sheet | Unlimited (within cell limit) |
| Sheets per spreadsheet | 200 |
| Characters per cell | 50,000 |

```javascript
// Check spreadsheet limits before operations
function checkSpreadsheetLimits(spreadsheet) {
  var sheets = spreadsheet.getSheets();
  var totalCells = 0;
  
  sheets.forEach(function(sheet) {
    var lastRow = sheet.getMaxRows();
    var lastCol = sheet.getMaxColumns();
    totalCells += lastRow * lastCol;
  });
  
  var limits = {
    totalCells: totalCells,
    percentUsed: (totalCells / 10000000 * 100).toFixed(2),
    sheetsCount: sheets.length,
    canAddSheet: sheets.length < 200,
    estimatedCellsAvailable: 10000000 - totalCells
  };
  
  console.log('Spreadsheet usage:', limits);
  
  if (limits.percentUsed > 80) {
    console.warn('Spreadsheet is ' + limits.percentUsed + '% full');
  }
  
  return limits;
}

// Optimize sheet size to stay within limits
function optimizeSheetSize(sheet) {
  // Remove empty rows and columns
  var lastRow = sheet.getLastRow();
  var lastCol = sheet.getLastColumn();
  var maxRows = sheet.getMaxRows();
  var maxCols = sheet.getMaxColumns();
  
  if (maxRows > lastRow + 100) {
    sheet.deleteRows(lastRow + 101, maxRows - lastRow - 100);
  }
  
  if (maxCols > lastCol + 10) {
    sheet.deleteColumns(lastCol + 11, maxCols - lastCol - 10);
  }
}
```

## Memory Limits

Apps Script has undocumented memory limits around 50-100 MB.

```javascript
// Memory-efficient data processing
function processLargeDataset(spreadsheetId) {
  var sheet = SpreadsheetApp.openById(spreadsheetId).getActiveSheet();
  var totalRows = sheet.getLastRow();
  var batchSize = 1000; // Process in chunks
  
  for (var startRow = 2; startRow <= totalRows; startRow += batchSize) {
    var numRows = Math.min(batchSize, totalRows - startRow + 1);
    
    // Read only current batch
    var batch = sheet.getRange(startRow, 1, numRows, 10).getValues();
    
    // Process batch
    var results = batch.map(processRow);
    
    // Write results
    sheet.getRange(startRow, 11, results.length, 1)
      .setValues(results.map(r => [r]));
    
    // Force garbage collection by clearing variables
    batch = null;
    results = null;
    
    // Brief pause to prevent overload
    if (startRow + batchSize <= totalRows) {
      Utilities.sleep(100);
    }
  }
}

// Streaming approach for large data
function* dataStreamer(sheet, batchSize = 100) {
  var lastRow = sheet.getLastRow();
  
  for (var row = 2; row <= lastRow; row += batchSize) {
    var numRows = Math.min(batchSize, lastRow - row + 1);
    var data = sheet.getRange(row, 1, numRows, 10).getValues();
    
    for (var i = 0; i < data.length; i++) {
      yield {
        row: row + i,
        data: data[i]
      };
    }
  }
}
```

## Script Size Limits

| Limit | Value |
|-------|-------|
| Script files per project | 100 |
| Characters per script file | 1,024,000 |
| Total project size | 50 MB |

```javascript
// Check script size
function getScriptSize() {
  // This is an approximation
  var files = DriveApp.getFilesByName('Script Name');
  
  if (files.hasNext()) {
    var file = files.next();
    console.log('Script size: ' + (file.getSize() / 1024).toFixed(2) + ' KB');
  }
}
```

## Performance Optimization Strategies

### 1. Batch Operations

```javascript
// BAD: Individual operations
function slowUpdate(sheet) {
  for (var i = 1; i <= 1000; i++) {
    sheet.getRange(i, 1).setValue(i); // 1000 API calls!
  }
}

// GOOD: Batch operations
function fastUpdate(sheet) {
  var values = [];
  for (var i = 1; i <= 1000; i++) {
    values.push([i]);
  }
  sheet.getRange(1, 1, values.length, 1).setValues(values); // 1 API call
}

// Benchmark function
function benchmark(func, ...args) {
  var start = new Date().getTime();
  var result = func.apply(null, args);
  var end = new Date().getTime();
  
  console.log(func.name + ' took ' + (end - start) + 'ms');
  return result;
}
```

### 2. Cache Frequently Used Data

```javascript
// Cache service usage
class DataCache {
  constructor() {
    this.cache = CacheService.getScriptCache();
    this.ttl = 3600; // 1 hour
  }
  
  get(key) {
    var cached = this.cache.get(key);
    if (cached) {
      return JSON.parse(cached);
    }
    return null;
  }
  
  set(key, value) {
    try {
      this.cache.put(key, JSON.stringify(value), this.ttl);
    } catch (error) {
      if (error.toString().includes('too large')) {
        console.warn('Cache value too large for key:', key);
      }
    }
  }
  
  getOrCompute(key, computeFunction) {
    var cached = this.get(key);
    if (cached) return cached;
    
    var value = computeFunction();
    this.set(key, value);
    return value;
  }
}

// Usage
function getExpensiveData() {
  var cache = new DataCache();
  
  return cache.getOrCompute('expensive_data', function() {
    // Expensive operation
    var data = SpreadsheetApp.openById('ID')
      .getDataRange()
      .getValues();
    
    return processData(data);
  });
}
```

### 3. Minimize Service Calls

```javascript
// Reduce API calls by getting all data at once
function efficientDataAccess() {
  var sheet = SpreadsheetApp.getActiveSheet();
  
  // BAD: Multiple getRange calls
  var name = sheet.getRange('A1').getValue();
  var email = sheet.getRange('B1').getValue();
  var phone = sheet.getRange('C1').getValue();
  
  // GOOD: Single getRange call
  var data = sheet.getRange('A1:C1').getValues()[0];
  var [name, email, phone] = data;
  
  // BEST: Get all data if needed multiple times
  var allData = sheet.getDataRange().getValues();
  // Process allData in memory
}
```

### 4. Use Built-in Methods

```javascript
// Built-in methods are optimized
function useBuiltInMethods() {
  var sheet = SpreadsheetApp.getActiveSheet();
  
  // BAD: Manual search
  var data = sheet.getDataRange().getValues();
  var found = null;
  for (var i = 0; i < data.length; i++) {
    if (data[i][0] === 'SearchTerm') {
      found = data[i];
      break;
    }
  }
  
  // GOOD: Use createTextFinder
  var finder = sheet.createTextFinder('SearchTerm');
  var match = finder.findNext();
  if (match) {
    found = match.getRow();
  }
}
```

### 5. Lazy Loading

```javascript
// Load data only when needed
class LazyDataLoader {
  constructor(spreadsheetId) {
    this.spreadsheetId = spreadsheetId;
    this._spreadsheet = null;
    this._sheets = {};
  }
  
  get spreadsheet() {
    if (!this._spreadsheet) {
      this._spreadsheet = SpreadsheetApp.openById(this.spreadsheetId);
    }
    return this._spreadsheet;
  }
  
  getSheet(name) {
    if (!this._sheets[name]) {
      this._sheets[name] = this.spreadsheet.getSheetByName(name);
    }
    return this._sheets[name];
  }
  
  getData(sheetName, range) {
    var sheet = this.getSheet(sheetName);
    return sheet.getRange(range).getValues();
  }
}
```

## Monitoring Performance

### Performance Logger

```javascript
class PerformanceMonitor {
  constructor() {
    this.metrics = [];
  }
  
  start(operation) {
    return {
      operation: operation,
      startTime: new Date().getTime(),
      startMemory: this.estimateMemoryUsage()
    };
  }
  
  end(measurement) {
    var endTime = new Date().getTime();
    var duration = endTime - measurement.startTime;
    
    this.metrics.push({
      operation: measurement.operation,
      duration: duration,
      timestamp: new Date()
    });
    
    console.log(measurement.operation + ' took ' + duration + 'ms');
    
    if (duration > 5000) {
      console.warn('Slow operation detected: ' + measurement.operation);
    }
  }
  
  estimateMemoryUsage() {
    // Rough estimate based on script properties size
    try {
      var props = PropertiesService.getScriptProperties().getProperties();
      return JSON.stringify(props).length;
    } catch (e) {
      return 0;
    }
  }
  
  getReport() {
    var report = {
      totalOperations: this.metrics.length,
      totalTime: this.metrics.reduce((sum, m) => sum + m.duration, 0),
      averageTime: 0,
      slowestOperation: null
    };
    
    if (this.metrics.length > 0) {
      report.averageTime = report.totalTime / this.metrics.length;
      report.slowestOperation = this.metrics.reduce((max, m) => 
        m.duration > max.duration ? m : max
      );
    }
    
    return report;
  }
}

// Usage
var monitor = new PerformanceMonitor();

function monitoredFunction() {
  var measure = monitor.start('Database Query');
  
  // Perform operation
  var data = SpreadsheetApp.openById('ID').getDataRange().getValues();
  
  monitor.end(measure);
  
  return data;
}
```

### Execution Time Distribution

```javascript
function analyzeExecutionTime() {
  // This would need to be collected over multiple runs
  var timings = [
    {operation: 'Read Sheet', time: 1200},
    {operation: 'Process Data', time: 3400},
    {operation: 'Write Results', time: 800},
    {operation: 'Send Email', time: 500}
  ];
  
  var total = timings.reduce((sum, t) => sum + t.time, 0);
  
  console.log('Execution Time Breakdown:');
  timings.forEach(function(timing) {
    var percentage = (timing.time / total * 100).toFixed(1);
    console.log(timing.operation + ': ' + timing.time + 'ms (' + percentage + '%)');
  });
  
  console.log('Total: ' + total + 'ms');
}
```

## Handling Quota Errors

```javascript
function handleQuotaErrors(func) {
  try {
    return func();
  } catch (error) {
    var errorString = error.toString();
    
    if (errorString.includes('Service invoked too many times')) {
      console.error('Quota exceeded: Service called too frequently');
      
      // Implement backoff
      Utilities.sleep(60000); // Wait 1 minute
      
      try {
        return func(); // Retry once
      } catch (retryError) {
        // Schedule for later
        ScriptApp.newTrigger('retryFunction')
          .timeBased()
          .after(60 * 60 * 1000) // 1 hour
          .create();
        
        throw new Error('Quota exceeded. Scheduled retry.');
      }
    }
    
    if (errorString.includes('Exceeded maximum execution time')) {
      console.error('Execution time limit reached');
      // Handle timeout
    }
    
    throw error;
  }
}
```

## Best Practices Summary

1. **Always batch operations** when working with Google services
2. **Monitor execution time** and implement resume logic for long operations
3. **Cache frequently accessed data** to reduce API calls
4. **Use streaming/chunking** for large datasets
5. **Implement proper error handling** for quota/limit errors
6. **Clean up resources** (close files, clear large variables)
7. **Use Properties Service** to track quotas and state
8. **Profile your code** to identify bottlenecks
9. **Minimize API calls** by getting all needed data at once
10. **Schedule work** across multiple executions for large jobs