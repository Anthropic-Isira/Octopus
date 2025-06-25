# Performance Optimization Quick Reference

## Batch Operations Over Loops

### ❌ SLOW - Individual Operations
```javascript
for (var i = 0; i < 1000; i++) {
  sheet.getRange(i, 1).setValue(data[i]); // 1000 API calls
}
```

### ✅ FAST - Batch Operations
```javascript
var range = sheet.getRange(1, 1, data.length, 1);
range.setValues(data.map(d => [d])); // 1 API call
```

## Caching Strategies

### Script-Level Cache (6 hours max)
```javascript
function getCachedData(key) {
  var cache = CacheService.getScriptCache();
  var cached = cache.get(key);
  
  if (!cached) {
    var data = expensiveOperation();
    cache.put(key, JSON.stringify(data), 21600); // 6 hours
    return data;
  }
  
  return JSON.parse(cached);
}
```

### Properties for Persistence
```javascript
function getPersistentData(key) {
  var props = PropertiesService.getScriptProperties();
  var data = props.getProperty(key);
  
  if (!data) {
    data = computeExpensiveData();
    props.setProperty(key, JSON.stringify(data));
  }
  
  return JSON.parse(data);
}
```

## Timeout Prevention

### Checkpoint Pattern
```javascript
function processWithCheckpoints() {
  var checkpoint = loadCheckpoint();
  var items = getItems();
  var maxTime = 5 * 60 * 1000; // 5 minutes
  var start = Date.now();
  
  for (var i = checkpoint; i < items.length; i++) {
    processItem(items[i]);
    
    if (Date.now() - start > maxTime) {
      saveCheckpoint(i);
      scheduleResume();
      return;
    }
    
    if (i % 100 === 0) saveCheckpoint(i); // Periodic saves
  }
  
  clearCheckpoint();
}
```

## Memory Management

### Process in Chunks
```javascript
function* chunkArray(array, size) {
  for (var i = 0; i < array.length; i += size) {
    yield array.slice(i, i + size);
  }
}

function processLargeArray(data) {
  for (var chunk of chunkArray(data, 1000)) {
    processChunk(chunk);
    SpreadsheetApp.flush(); // Force write
    Utilities.sleep(100); // Allow GC
  }
}
```

## API Call Optimization

### Minimize Service Calls
```javascript
// ❌ Multiple calls
var name = file.getName();
var size = file.getSize();
var mime = file.getMimeType();

// ✅ Single call pattern
var file = DriveApp.getFileById(id);
var metadata = {
  name: file.getName(),
  size: file.getSize(),
  mime: file.getMimeType()
};
```

### Parallel Processing
```javascript
// ✅ Process independent operations together
var sheet = SpreadsheetApp.getActiveSheet();
var [values, formulas, backgrounds] = [
  sheet.getDataRange().getValues(),
  sheet.getDataRange().getFormulas(),
  sheet.getDataRange().getBackgrounds()
];
```

## Key Performance Metrics

| Operation | Typical Time | Optimization |
|-----------|-------------|--------------|
| getValue() | 0.1-0.2s | Use getValues() |
| setValue() | 0.1-0.2s | Use setValues() |
| appendRow() | 0.2-0.5s | Batch with setValues() |
| getFiles() | 0.5-2s | Use search queries |
| sendEmail() | 0.5-1s | Batch when possible |

## Performance Checklist

1. **Use batch operations** for all read/write
2. **Cache expensive calculations**
3. **Implement checkpoints** for long processes
4. **Chunk large datasets** to prevent memory issues
5. **Minimize API calls** by reading all needed data upfront
6. **Use SpreadsheetApp.flush()** strategically
7. **Profile with console.time()** to find bottlenecks
8. **Avoid nested loops** with API calls
9. **Use search queries** instead of iterating all files
10. **Schedule long operations** across multiple executions