# Performance Optimization Guide for Google Apps Script

[← Back to Main Documentation](../README.md)

## Overview

This guide provides strategies and techniques for optimizing Google Apps Script performance, addressing common bottlenecks, and improving execution speed.

## Performance Profiling

### Measuring Execution Time

```javascript
// Basic timing
function measureExecutionTime() {
  var start = new Date().getTime();
  
  // Your code here
  performOperation();
  
  var end = new Date().getTime();
  console.log('Execution time: ' + (end - start) + 'ms');
}

// Advanced profiler class
class PerformanceProfiler {
  constructor(name) {
    this.name = name;
    this.measurements = [];
    this.marks = new Map();
    this.start = new Date().getTime();
  }
  
  mark(label) {
    var now = new Date().getTime();
    this.marks.set(label, now);
    
    var elapsed = now - this.start;
    var lastMark = this.measurements.length > 0 
      ? this.measurements[this.measurements.length - 1].elapsed 
      : 0;
    
    this.measurements.push({
      label: label,
      elapsed: elapsed,
      delta: elapsed - lastMark,
      timestamp: now
    });
  }
  
  measure(label, startMark, endMark) {
    var start = this.marks.get(startMark);
    var end = this.marks.get(endMark);
    
    if (!start || !end) {
      console.error('Invalid marks for measurement');
      return;
    }
    
    this.measurements.push({
      label: label,
      elapsed: end - start,
      delta: end - start,
      timestamp: end
    });
  }
  
  getReport() {
    var report = `\n=== Performance Report: ${this.name} ===\n`;
    report += `Total time: ${new Date().getTime() - this.start}ms\n\n`;
    
    this.measurements.forEach((m, i) => {
      report += `${i + 1}. ${m.label}: ${m.delta}ms (total: ${m.elapsed}ms)\n`;
    });
    
    // Find bottlenecks
    var sorted = [...this.measurements].sort((a, b) => b.delta - a.delta);
    report += '\nTop 3 slowest operations:\n';
    sorted.slice(0, 3).forEach(m => {
      report += `- ${m.label}: ${m.delta}ms\n`;
    });
    
    return report;
  }
  
  log() {
    console.log(this.getReport());
  }
}

// Usage
function profiledFunction() {
  var profiler = new PerformanceProfiler('DataProcessing');
  
  profiler.mark('Start');
  
  var data = fetchData();
  profiler.mark('Data fetched');
  
  var processed = processData(data);
  profiler.mark('Data processed');
  
  saveData(processed);
  profiler.mark('Data saved');
  
  profiler.log();
}
```

## Spreadsheet Operations Optimization

### Batch Operations vs Individual Calls

```javascript
// SLOW: Individual cell operations
function slowSpreadsheetUpdate() {
  var sheet = SpreadsheetApp.getActiveSheet();
  
  for (var i = 1; i <= 1000; i++) {
    for (var j = 1; j <= 10; j++) {
      // Each call is a separate API request
      sheet.getRange(i, j).setValue(i * j);
    }
  }
}

// FAST: Batch operations
function fastSpreadsheetUpdate() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var data = [];
  
  // Build data array in memory
  for (var i = 1; i <= 1000; i++) {
    var row = [];
    for (var j = 1; j <= 10; j++) {
      row.push(i * j);
    }
    data.push(row);
  }
  
  // Single API call
  sheet.getRange(1, 1, data.length, data[0].length).setValues(data);
}

// Batch reading optimization
function optimizedDataReading() {
  var sheet = SpreadsheetApp.getActiveSheet();
  
  // SLOW: Multiple getRange calls
  var slowData = [];
  for (var i = 1; i <= 100; i++) {
    slowData.push(sheet.getRange(i, 1).getValue());
  }
  
  // FAST: Single batch read
  var fastData = sheet.getRange(1, 1, 100, 1).getValues()
    .map(row => row[0]);
  
  return fastData;
}
```

### Range Operations Best Practices

```javascript
// Optimize range operations
class SpreadsheetOptimizer {
  constructor(spreadsheet) {
    this.spreadsheet = spreadsheet;
    this.cache = new Map();
  }
  
  // Cache sheet references
  getSheet(name) {
    if (!this.cache.has(name)) {
      this.cache.set(name, this.spreadsheet.getSheetByName(name));
    }
    return this.cache.get(name);
  }
  
  // Minimize range size
  getDataRange(sheet) {
    var lastRow = sheet.getLastRow();
    var lastCol = sheet.getLastColumn();
    
    if (lastRow === 0 || lastCol === 0) {
      return null;
    }
    
    return sheet.getRange(1, 1, lastRow, lastCol);
  }
  
  // Batch multiple operations
  batchUpdate(updates) {
    // Group updates by sheet
    var sheetUpdates = new Map();
    
    updates.forEach(update => {
      if (!sheetUpdates.has(update.sheet)) {
        sheetUpdates.set(update.sheet, []);
      }
      sheetUpdates.get(update.sheet).push(update);
    });
    
    // Apply updates per sheet
    sheetUpdates.forEach((updates, sheetName) => {
      var sheet = this.getSheet(sheetName);
      
      // Sort updates by range to potentially combine
      updates.sort((a, b) => a.row - b.row || a.col - b.col);
      
      // Apply updates
      updates.forEach(update => {
        sheet.getRange(update.row, update.col, 
          update.values.length, update.values[0].length)
          .setValues(update.values);
      });
    });
  }
  
  // Efficient find and replace
  findAndReplace(sheet, find, replace) {
    var data = this.getDataRange(sheet).getValues();
    var modified = false;
    
    for (var i = 0; i < data.length; i++) {
      for (var j = 0; j < data[i].length; j++) {
        if (data[i][j] === find) {
          data[i][j] = replace;
          modified = true;
        }
      }
    }
    
    if (modified) {
      sheet.getRange(1, 1, data.length, data[0].length).setValues(data);
    }
    
    return modified;
  }
}
```

## Drive Operations Optimization

### Batch File Operations

```javascript
// Optimize Drive API calls
class DriveOptimizer {
  // Batch create files
  static createFiles(folderID, files) {
    var folder = DriveApp.getFolderById(folderID);
    var created = [];
    
    // Use batch operations when possible
    files.forEach(file => {
      var blob = Utilities.newBlob(file.content, file.mimeType, file.name);
      created.push(folder.createFile(blob));
    });
    
    return created;
  }
  
  // Efficient file search
  static findFiles(query, maxResults = 100) {
    var files = [];
    var pageToken;
    
    do {
      var result = Drive.Files.list({
        q: query,
        maxResults: Math.min(maxResults - files.length, 100),
        pageToken: pageToken,
        fields: 'items(id,title,mimeType,modifiedDate),nextPageToken'
      });
      
      files = files.concat(result.items || []);
      pageToken = result.nextPageToken;
      
    } while (pageToken && files.length < maxResults);
    
    return files;
  }
  
  // Batch permission updates
  static updatePermissions(fileIds, permissions) {
    var batch = Drive.newBatch();
    
    fileIds.forEach(fileId => {
      permissions.forEach(permission => {
        var request = Drive.Permissions.insert(permission, fileId);
        batch.add(request);
      });
    });
    
    return batch.execute();
  }
  
  // Efficient folder traversal
  static* traverseFolder(folderId, maxDepth = 5) {
    var queue = [{id: folderId, depth: 0}];
    var visited = new Set();
    
    while (queue.length > 0) {
      var current = queue.shift();
      
      if (visited.has(current.id) || current.depth > maxDepth) {
        continue;
      }
      
      visited.add(current.id);
      
      // Yield current folder
      yield current;
      
      // Get subfolders
      var subfolders = Drive.Files.list({
        q: `'${current.id}' in parents and mimeType = 'application/vnd.google-apps.folder'`,
        maxResults: 100,
        fields: 'items(id,title)'
      });
      
      if (subfolders.items) {
        subfolders.items.forEach(folder => {
          queue.push({
            id: folder.id,
            depth: current.depth + 1,
            title: folder.title,
            parent: current.id
          });
        });
      }
    }
  }
}
```

## Cache Optimization

### Multi-Level Caching Strategy

```javascript
// Efficient caching system
class CacheSystem {
  constructor() {
    this.memoryCache = new Map();
    this.scriptCache = CacheService.getScriptCache();
    this.cacheStats = {
      hits: 0,
      misses: 0,
      memoryHits: 0,
      scriptHits: 0
    };
  }
  
  get(key, computeFunction = null, ttl = 3600) {
    // Level 1: Memory cache
    if (this.memoryCache.has(key)) {
      this.cacheStats.hits++;
      this.cacheStats.memoryHits++;
      return this.memoryCache.get(key);
    }
    
    // Level 2: Script cache
    var cached = this.scriptCache.get(key);
    if (cached) {
      this.cacheStats.hits++;
      this.cacheStats.scriptHits++;
      
      var value = JSON.parse(cached);
      this.memoryCache.set(key, value);
      return value;
    }
    
    // Cache miss
    this.cacheStats.misses++;
    
    if (computeFunction) {
      var value = computeFunction();
      this.set(key, value, ttl);
      return value;
    }
    
    return null;
  }
  
  set(key, value, ttl = 3600) {
    // Store in both caches
    this.memoryCache.set(key, value);
    
    var serialized = JSON.stringify(value);
    if (serialized.length <= 100000) { // Script cache limit
      this.scriptCache.put(key, serialized, Math.min(ttl, 21600));
    }
  }
  
  // Batch get with efficient loading
  getBatch(keys) {
    var results = {};
    var missingKeys = [];
    
    // Check memory cache first
    keys.forEach(key => {
      if (this.memoryCache.has(key)) {
        results[key] = this.memoryCache.get(key);
      } else {
        missingKeys.push(key);
      }
    });
    
    // Check script cache for missing
    if (missingKeys.length > 0) {
      var scriptResults = this.scriptCache.getAll(missingKeys);
      
      Object.entries(scriptResults).forEach(([key, value]) => {
        if (value) {
          var parsed = JSON.parse(value);
          results[key] = parsed;
          this.memoryCache.set(key, parsed);
        }
      });
    }
    
    return results;
  }
  
  getStats() {
    var hitRate = this.cacheStats.hits / 
      (this.cacheStats.hits + this.cacheStats.misses) || 0;
    
    return {
      ...this.cacheStats,
      hitRate: (hitRate * 100).toFixed(2) + '%'
    };
  }
}

// Usage
var cache = new CacheSystem();

function getCachedData(id) {
  return cache.get(`data_${id}`, () => {
    // Expensive computation
    return fetchDataFromAPI(id);
  }, 3600);
}
```

## Algorithm Optimization

### Data Structure Selection

```javascript
// Choose efficient data structures
class DataStructureOptimizer {
  // Use Map for frequent lookups
  static createLookupMap(data, keyField) {
    var map = new Map();
    
    data.forEach(item => {
      map.set(item[keyField], item);
    });
    
    return map;
  }
  
  // Use Set for unique values
  static getUniqueValues(data, field) {
    var uniqueSet = new Set();
    
    data.forEach(item => {
      if (item[field] !== undefined) {
        uniqueSet.add(item[field]);
      }
    });
    
    return Array.from(uniqueSet);
  }
  
  // Efficient array operations
  static filterAndMap(data, filterFn, mapFn) {
    var result = [];
    
    for (var i = 0; i < data.length; i++) {
      if (filterFn(data[i])) {
        result.push(mapFn(data[i]));
      }
    }
    
    return result;
  }
  
  // Binary search for sorted data
  static binarySearch(sortedArray, target, compareFn) {
    var left = 0;
    var right = sortedArray.length - 1;
    
    while (left <= right) {
      var mid = Math.floor((left + right) / 2);
      var comparison = compareFn(sortedArray[mid], target);
      
      if (comparison === 0) {
        return mid;
      } else if (comparison < 0) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
    
    return -1;
  }
}

// Example: Optimized data processing
function processLargeDataset(data) {
  var profiler = new PerformanceProfiler('DataProcessing');
  
  profiler.mark('Start');
  
  // Create lookup maps for O(1) access
  var userMap = DataStructureOptimizer.createLookupMap(data.users, 'id');
  var roleMap = DataStructureOptimizer.createLookupMap(data.roles, 'id');
  
  profiler.mark('Lookup maps created');
  
  // Process with efficient algorithms
  var results = data.records.map(record => {
    var user = userMap.get(record.userId);
    var role = roleMap.get(record.roleId);
    
    return {
      ...record,
      userName: user ? user.name : 'Unknown',
      roleName: role ? role.name : 'Unknown'
    };
  });
  
  profiler.mark('Processing complete');
  profiler.log();
  
  return results;
}
```

## API Call Optimization

### Request Batching and Pooling

```javascript
// API request optimizer
class APIOptimizer {
  constructor(options = {}) {
    this.batchSize = options.batchSize || 50;
    this.batchDelay = options.batchDelay || 100;
    this.maxConcurrent = options.maxConcurrent || 5;
    this.queue = [];
    this.processing = false;
  }
  
  // Batch multiple requests
  async batchRequest(request) {
    return new Promise((resolve, reject) => {
      this.queue.push({request, resolve, reject});
      
      if (!this.processing) {
        this.processQueue();
      }
    });
  }
  
  async processQueue() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      var batch = this.queue.splice(0, this.batchSize);
      
      try {
        var results = await this.executeBatch(batch.map(item => item.request));
        
        batch.forEach((item, index) => {
          item.resolve(results[index]);
        });
        
      } catch (error) {
        batch.forEach(item => {
          item.reject(error);
        });
      }
      
      if (this.queue.length > 0) {
        Utilities.sleep(this.batchDelay);
      }
    }
    
    this.processing = false;
  }
  
  async executeBatch(requests) {
    // Implement actual batch execution
    return Promise.all(requests);
  }
}

// Connection pooling for external APIs
class ConnectionPool {
  constructor(maxConnections = 10) {
    this.maxConnections = maxConnections;
    this.activeConnections = 0;
    this.waitQueue = [];
  }
  
  async acquire() {
    if (this.activeConnections < this.maxConnections) {
      this.activeConnections++;
      return true;
    }
    
    // Wait for available connection
    return new Promise(resolve => {
      this.waitQueue.push(resolve);
    });
  }
  
  release() {
    this.activeConnections--;
    
    if (this.waitQueue.length > 0) {
      var next = this.waitQueue.shift();
      this.activeConnections++;
      next(true);
    }
  }
  
  async execute(fn) {
    await this.acquire();
    
    try {
      return await fn();
    } finally {
      this.release();
    }
  }
}
```

## Memory Management

### Efficient Memory Usage

```javascript
// Memory-efficient operations
class MemoryManager {
  // Process large data in chunks
  static processInChunks(data, chunkSize, processor) {
    var results = [];
    
    for (var i = 0; i < data.length; i += chunkSize) {
      var chunk = data.slice(i, i + chunkSize);
      var chunkResults = processor(chunk);
      
      // Append results
      results = results.concat(chunkResults);
      
      // Clear references
      chunk = null;
      chunkResults = null;
      
      // Allow garbage collection
      if (i % (chunkSize * 10) === 0) {
        Utilities.sleep(10);
      }
    }
    
    return results;
  }
  
  // Stream processing for very large datasets
  static* streamProcess(getDataFn, batchSize = 1000) {
    var offset = 0;
    var hasMore = true;
    
    while (hasMore) {
      var batch = getDataFn(offset, batchSize);
      
      if (!batch || batch.length === 0) {
        hasMore = false;
      } else {
        yield batch;
        offset += batch.length;
        
        if (batch.length < batchSize) {
          hasMore = false;
        }
      }
    }
  }
  
  // Clear large objects from memory
  static cleanup(...objects) {
    objects.forEach(obj => {
      if (Array.isArray(obj)) {
        obj.length = 0;
      } else if (obj && typeof obj === 'object') {
        Object.keys(obj).forEach(key => {
          delete obj[key];
        });
      }
    });
  }
}

// Example usage
function processLargeSpreadsheet() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var rowCount = sheet.getLastRow();
  var results = [];
  
  // Process in manageable chunks
  for (var batch of MemoryManager.streamProcess(
    (offset, size) => {
      if (offset >= rowCount) return null;
      
      var numRows = Math.min(size, rowCount - offset);
      return sheet.getRange(offset + 1, 1, numRows, 10).getValues();
    },
    500
  )) {
    // Process batch
    var processed = batch.map(row => processRow(row));
    
    // Save results immediately instead of accumulating
    saveResults(processed);
    
    // Clear batch from memory
    MemoryManager.cleanup(batch, processed);
  }
}
```

## Parallel Processing

### Simulating Parallel Execution

```javascript
// Parallel processing simulation
class ParallelProcessor {
  static async processInParallel(items, processor, maxConcurrent = 5) {
    var results = new Array(items.length);
    var queue = items.map((item, index) => ({item, index}));
    var active = [];
    
    while (queue.length > 0 || active.length > 0) {
      // Start new processes up to limit
      while (active.length < maxConcurrent && queue.length > 0) {
        var task = queue.shift();
        
        var promise = processor(task.item)
          .then(result => {
            results[task.index] = result;
            // Remove from active
            var idx = active.findIndex(p => p === promise);
            if (idx > -1) active.splice(idx, 1);
          })
          .catch(error => {
            results[task.index] = {error: error.message};
            // Remove from active
            var idx = active.findIndex(p => p === promise);
            if (idx > -1) active.splice(idx, 1);
          });
        
        active.push(promise);
      }
      
      // Wait for at least one to complete
      if (active.length > 0) {
        await Promise.race(active);
      }
    }
    
    return results;
  }
  
  // Divide work among triggers
  static divideWork(totalItems, workers) {
    var itemsPerWorker = Math.ceil(totalItems / workers);
    var ranges = [];
    
    for (var i = 0; i < workers; i++) {
      var start = i * itemsPerWorker;
      var end = Math.min(start + itemsPerWorker, totalItems);
      
      if (start < totalItems) {
        ranges.push({
          worker: i,
          start: start,
          end: end,
          count: end - start
        });
      }
    }
    
    return ranges;
  }
}

// Example: Parallel URL fetching
async function fetchMultipleURLs(urls) {
  return ParallelProcessor.processInParallel(
    urls,
    async (url) => {
      try {
        var response = UrlFetchApp.fetch(url);
        return {
          url: url,
          status: response.getResponseCode(),
          content: response.getContentText()
        };
      } catch (error) {
        return {
          url: url,
          error: error.message
        };
      }
    },
    10 // Max 10 concurrent requests
  );
}
```

## Script Optimization Checklist

### Before Optimization
1. **Profile current performance** - Identify bottlenecks
2. **Set performance goals** - Define acceptable execution time
3. **Identify critical paths** - Focus on high-impact areas

### Optimization Strategies
1. **Batch API calls** - Reduce round trips
2. **Use appropriate data structures** - Maps, Sets for lookups
3. **Implement caching** - Multi-level cache strategy
4. **Optimize algorithms** - Better time complexity
5. **Manage memory** - Process in chunks, clear references
6. **Minimize I/O operations** - Batch reads/writes
7. **Use built-in methods** - Often optimized in C++
8. **Avoid repeated calculations** - Memoization
9. **Parallelize when possible** - Multiple triggers
10. **Monitor and measure** - Continuous improvement

### After Optimization
1. **Verify correctness** - Ensure functionality unchanged
2. **Measure improvement** - Document performance gains
3. **Monitor in production** - Watch for edge cases
4. **Document changes** - Help future maintenance

## Performance Benchmarks

### Expected Performance Metrics

| Operation | Records | Expected Time | Optimized Time |
|-----------|---------|--------------|----------------|
| Sheet Read | 1,000 | 2-3s | <1s |
| Sheet Write | 1,000 | 3-4s | <1s |
| Drive Search | 100 files | 5-10s | 2-3s |
| API Calls | 100 | 30-60s | 5-10s |
| Data Processing | 10,000 | 10-20s | 2-5s |

## Best Practices

1. **Always batch operations** when possible
2. **Cache expensive computations** aggressively
3. **Use efficient data structures** for your use case
4. **Monitor execution time** in production
5. **Set up alerts** for performance degradation
6. **Document performance** requirements
7. **Test with realistic data** volumes
8. **Plan for growth** - Design for 10x current load
9. **Review regularly** - Performance can degrade over time
10. **Share knowledge** - Document optimizations for team