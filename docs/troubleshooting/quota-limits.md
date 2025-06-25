# Google Apps Script Quota Limits Guide

[← Back to Main Documentation](../README.md)

## Overview

Google Apps Script has various quotas and limits to ensure fair usage and system stability. This guide provides comprehensive information about these limits and strategies to work within them.

## Service Quotas by Account Type

### Consumer Accounts (gmail.com)

| Service | Quota | Time Period | Notes |
|---------|-------|-------------|-------|
| **Email** |
| Recipients per day | 100 | 24 hours | Includes To, Cc, Bcc |
| Email read/write | 20,000 | 24 hours | |
| **Drive** |
| Files created | 250 | 24 hours | |
| Files opened/modified | 2,500 | 24 hours | |
| **Calendar** |
| Events created | 5,000 | 24 hours | |
| Calendar read/write | 20,000 | 24 hours | |
| **Contacts** |
| Contacts created | 1,000 | 24 hours | |
| Contacts read/write | 10,000 | 24 hours | |
| **URL Fetch** |
| Requests | 20,000 | 24 hours | |
| Response size | 50MB | Per request | |

### Google Workspace Accounts

| Service | Quota | Time Period | Notes |
|---------|-------|-------------|-------|
| **Email** |
| Recipients per day | 1,500 | 24 hours | Can be higher for some plans |
| Email read/write | 50,000 | 24 hours | |
| **Drive** |
| Files created | 500 | 24 hours | |
| Files opened/modified | 10,000 | 24 hours | |
| **Calendar** |
| Events created | 10,000 | 24 hours | |
| Calendar read/write | 50,000 | 24 hours | |
| **URL Fetch** |
| Requests | 100,000 | 24 hours | |
| Response size | 50MB | Per request | |

## Script Runtime Limits

### Execution Time Limits

```javascript
// Check remaining execution time
function checkExecutionTime() {
  var startTime = new Date().getTime();
  var maxRuntime = 6 * 60 * 1000; // 6 minutes for consumer
  // var maxRuntime = 30 * 60 * 1000; // 30 minutes for Workspace
  
  function getRemainingTime() {
    var elapsed = new Date().getTime() - startTime;
    return maxRuntime - elapsed;
  }
  
  function shouldStop() {
    return getRemainingTime() < 60000; // Stop with 1 minute buffer
  }
  
  // Example usage in long-running process
  var items = getLargeDataset();
  var processed = 0;
  
  for (var i = 0; i < items.length; i++) {
    if (shouldStop()) {
      // Save progress and schedule continuation
      saveProgress(i);
      scheduleContinuation();
      break;
    }
    
    processItem(items[i]);
    processed++;
  }
  
  console.log(`Processed ${processed} items`);
}

// Create continuation trigger
function scheduleContinuation() {
  // Delete existing continuation triggers
  var triggers = ScriptApp.getProjectTriggers();
  triggers.forEach(trigger => {
    if (trigger.getHandlerFunction() === 'continueLongProcess') {
      ScriptApp.deleteTrigger(trigger);
    }
  });
  
  // Create new trigger to run in 1 minute
  ScriptApp.newTrigger('continueLongProcess')
    .timeBased()
    .after(60 * 1000)
    .create();
}
```

### Trigger Limits

| Trigger Type | Consumer Limit | Workspace Limit |
|--------------|----------------|-----------------|
| Total triggers | 20 | 20 |
| Time-based triggers | 20 | 20 |
| Simultaneous executions | 30 | 30 |

```javascript
// Manage triggers within limits
class TriggerManager {
  static getTriggerCount() {
    return ScriptApp.getProjectTriggers().length;
  }
  
  static canCreateTrigger() {
    return this.getTriggerCount() < 20;
  }
  
  static createManagedTrigger(functionName, type, options) {
    if (!this.canCreateTrigger()) {
      // Remove oldest trigger of same type
      this.removeOldestTrigger(type);
    }
    
    var trigger;
    
    switch(type) {
      case 'time':
        trigger = ScriptApp.newTrigger(functionName)
          .timeBased()
          .everyMinutes(options.minutes || 1)
          .create();
        break;
        
      case 'spreadsheet':
        trigger = ScriptApp.newTrigger(functionName)
          .forSpreadsheet(options.spreadsheetId)
          .onEdit()
          .create();
        break;
        
      case 'form':
        trigger = ScriptApp.newTrigger(functionName)
          .forForm(options.formId)
          .onFormSubmit()
          .create();
        break;
    }
    
    // Store trigger metadata
    var props = PropertiesService.getScriptProperties();
    props.setProperty('trigger_' + trigger.getUniqueId(), JSON.stringify({
      created: new Date().toISOString(),
      type: type,
      function: functionName
    }));
    
    return trigger;
  }
  
  static removeOldestTrigger(type) {
    var triggers = ScriptApp.getProjectTriggers();
    var props = PropertiesService.getScriptProperties();
    
    // Find oldest trigger of specified type
    var oldestTrigger = null;
    var oldestDate = new Date();
    
    triggers.forEach(trigger => {
      var metadata = props.getProperty('trigger_' + trigger.getUniqueId());
      if (metadata) {
        var data = JSON.parse(metadata);
        if (data.type === type) {
          var created = new Date(data.created);
          if (created < oldestDate) {
            oldestDate = created;
            oldestTrigger = trigger;
          }
        }
      }
    });
    
    if (oldestTrigger) {
      ScriptApp.deleteTrigger(oldestTrigger);
      props.deleteProperty('trigger_' + oldestTrigger.getUniqueId());
    }
  }
}
```

## Properties Service Limits

| Property Type | Size Limit | Total Limit | Key Limit |
|---------------|------------|-------------|-----------|
| Script Properties | 9KB/value | 500KB | 500 keys |
| User Properties | 9KB/value | 500KB | 500 keys |
| Document Properties | 9KB/value | 500KB | 500 keys |

```javascript
// Properties storage manager
class PropertiesManager {
  constructor(type = 'script') {
    switch(type) {
      case 'user':
        this.properties = PropertiesService.getUserProperties();
        break;
      case 'document':
        this.properties = PropertiesService.getDocumentProperties();
        break;
      default:
        this.properties = PropertiesService.getScriptProperties();
    }
    
    this.prefix = type + '_';
    this.maxValueSize = 9 * 1024; // 9KB
    this.maxTotalSize = 500 * 1024; // 500KB
  }
  
  setLargeValue(key, value) {
    var stringValue = JSON.stringify(value);
    
    if (stringValue.length <= this.maxValueSize) {
      // Fits in single property
      this.properties.setProperty(this.prefix + key, stringValue);
      return true;
    }
    
    // Split into chunks
    var chunks = this.splitIntoChunks(stringValue, this.maxValueSize - 100);
    
    // Store chunk count
    this.properties.setProperty(this.prefix + key + '_chunks', chunks.length.toString());
    
    // Store each chunk
    chunks.forEach((chunk, index) => {
      this.properties.setProperty(this.prefix + key + '_' + index, chunk);
    });
    
    return true;
  }
  
  getLargeValue(key) {
    // Check if it's a multi-chunk value
    var chunkCount = this.properties.getProperty(this.prefix + key + '_chunks');
    
    if (!chunkCount) {
      // Single value
      var value = this.properties.getProperty(this.prefix + key);
      return value ? JSON.parse(value) : null;
    }
    
    // Reassemble chunks
    var chunks = [];
    for (var i = 0; i < parseInt(chunkCount); i++) {
      var chunk = this.properties.getProperty(this.prefix + key + '_' + i);
      if (chunk) chunks.push(chunk);
    }
    
    return chunks.length > 0 ? JSON.parse(chunks.join('')) : null;
  }
  
  splitIntoChunks(str, chunkSize) {
    var chunks = [];
    for (var i = 0; i < str.length; i += chunkSize) {
      chunks.push(str.substring(i, i + chunkSize));
    }
    return chunks;
  }
  
  getStorageInfo() {
    var allProps = this.properties.getProperties();
    var totalSize = 0;
    var keyCount = 0;
    
    Object.entries(allProps).forEach(([key, value]) => {
      totalSize += key.length + value.length;
      keyCount++;
    });
    
    return {
      keyCount: keyCount,
      totalSize: totalSize,
      usagePercent: ((totalSize / this.maxTotalSize) * 100).toFixed(2),
      remaining: this.maxTotalSize - totalSize
    };
  }
  
  cleanup(daysOld = 30) {
    var cutoff = new Date();
    cutoff.setDate(cutoff.getDate() - daysOld);
    
    var allProps = this.properties.getProperties();
    var deleted = 0;
    
    Object.keys(allProps).forEach(key => {
      if (key.includes('_timestamp_')) {
        var timestamp = parseInt(allProps[key]);
        if (timestamp < cutoff.getTime()) {
          var baseKey = key.replace('_timestamp_', '');
          this.deleteValue(baseKey);
          deleted++;
        }
      }
    });
    
    return deleted;
  }
}
```

## Cache Service Limits

| Cache Type | Max Key Size | Max Value Size | Total Size | Expiration |
|------------|--------------|----------------|------------|------------|
| Script Cache | 250 chars | 100KB | 10MB | 6 hours max |
| User Cache | 250 chars | 100KB | 10MB | 6 hours max |
| Document Cache | 250 chars | 100KB | 10MB | 6 hours max |

```javascript
// Advanced cache management
class CacheManager {
  constructor(cacheType = 'script') {
    this.cache = this.getCache(cacheType);
    this.maxKeyLength = 250;
    this.maxValueSize = 100 * 1024; // 100KB
    this.defaultTTL = 21600; // 6 hours (maximum)
  }
  
  getCache(type) {
    switch(type) {
      case 'user':
        return CacheService.getUserCache();
      case 'document':
        return CacheService.getDocumentCache();
      default:
        return CacheService.getScriptCache();
    }
  }
  
  set(key, value, ttl = 3600) {
    // Validate key length
    if (key.length > this.maxKeyLength) {
      key = this.hashKey(key);
    }
    
    var stringValue = JSON.stringify(value);
    
    // Check value size
    if (stringValue.length > this.maxValueSize) {
      return this.setLargeValue(key, stringValue, ttl);
    }
    
    this.cache.put(key, stringValue, Math.min(ttl, this.defaultTTL));
    return true;
  }
  
  setLargeValue(key, stringValue, ttl) {
    // Compress if possible
    var compressed = this.compress(stringValue);
    
    if (compressed.length <= this.maxValueSize) {
      this.cache.put(key + '_compressed', compressed, Math.min(ttl, this.defaultTTL));
      this.cache.put(key + '_meta', 'compressed', Math.min(ttl, this.defaultTTL));
      return true;
    }
    
    // Split into multiple keys
    var chunks = Math.ceil(compressed.length / this.maxValueSize);
    
    if (chunks > 10) {
      console.warn('Value too large for cache even after compression');
      return false;
    }
    
    // Store metadata
    this.cache.put(key + '_meta', JSON.stringify({
      type: 'chunked',
      chunks: chunks,
      compressed: true
    }), Math.min(ttl, this.defaultTTL));
    
    // Store chunks
    for (var i = 0; i < chunks; i++) {
      var chunk = compressed.substring(
        i * this.maxValueSize,
        (i + 1) * this.maxValueSize
      );
      this.cache.put(key + '_chunk_' + i, chunk, Math.min(ttl, this.defaultTTL));
    }
    
    return true;
  }
  
  get(key) {
    if (key.length > this.maxKeyLength) {
      key = this.hashKey(key);
    }
    
    // Check for metadata
    var meta = this.cache.get(key + '_meta');
    
    if (!meta) {
      // Simple value
      var value = this.cache.get(key);
      return value ? JSON.parse(value) : null;
    }
    
    if (meta === 'compressed') {
      // Compressed value
      var compressed = this.cache.get(key + '_compressed');
      if (!compressed) return null;
      
      var decompressed = this.decompress(compressed);
      return JSON.parse(decompressed);
    }
    
    // Chunked value
    var metaData = JSON.parse(meta);
    if (metaData.type === 'chunked') {
      var chunks = [];
      
      for (var i = 0; i < metaData.chunks; i++) {
        var chunk = this.cache.get(key + '_chunk_' + i);
        if (!chunk) return null; // Missing chunk
        chunks.push(chunk);
      }
      
      var combined = chunks.join('');
      if (metaData.compressed) {
        combined = this.decompress(combined);
      }
      
      return JSON.parse(combined);
    }
    
    return null;
  }
  
  hashKey(key) {
    // Simple hash function for long keys
    var hash = 0;
    for (var i = 0; i < key.length; i++) {
      var char = key.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return 'hash_' + Math.abs(hash).toString(36);
  }
  
  compress(str) {
    // Simple compression using repeated string replacement
    // In real implementation, you might use a proper compression library
    return str;
  }
  
  decompress(str) {
    // Decompression
    return str;
  }
}
```

## URL Fetch Quotas and Limits

### Request Limits

```javascript
// URL Fetch quota tracker
class URLFetchQuotaTracker {
  constructor() {
    this.props = PropertiesService.getScriptProperties();
    this.cache = CacheService.getScriptCache();
    this.quotaKey = 'urlFetch_quota_' + new Date().toDateString();
  }
  
  canMakeRequest(count = 1) {
    var usage = this.getCurrentUsage();
    var limit = this.getDailyLimit();
    
    return (usage + count) <= limit;
  }
  
  getCurrentUsage() {
    var usage = this.cache.get(this.quotaKey);
    return usage ? parseInt(usage) : 0;
  }
  
  getDailyLimit() {
    // Check if Workspace account
    var isWorkspace = Session.getActiveUser().getEmail().includes('@');
    return isWorkspace ? 100000 : 20000;
  }
  
  recordRequest(count = 1) {
    var usage = this.getCurrentUsage();
    this.cache.put(this.quotaKey, (usage + count).toString(), 86400);
  }
  
  makeRequest(url, params = {}) {
    if (!this.canMakeRequest()) {
      throw new Error('URL Fetch daily quota exceeded');
    }
    
    try {
      var response = UrlFetchApp.fetch(url, params);
      this.recordRequest();
      return response;
      
    } catch (error) {
      if (error.message.includes('Service invoked too many times')) {
        // Update our counter to reflect actual limit
        this.cache.put(this.quotaKey, this.getDailyLimit().toString(), 86400);
      }
      throw error;
    }
  }
  
  getRemainingQuota() {
    var usage = this.getCurrentUsage();
    var limit = this.getDailyLimit();
    
    return {
      used: usage,
      limit: limit,
      remaining: limit - usage,
      percentUsed: ((usage / limit) * 100).toFixed(2)
    };
  }
}

// Batch URL fetching with quota management
class BatchURLFetcher {
  constructor(maxConcurrent = 10) {
    this.maxConcurrent = maxConcurrent;
    this.quotaTracker = new URLFetchQuotaTracker();
  }
  
  fetchAll(requests) {
    var results = [];
    var batches = this.createBatches(requests);
    
    batches.forEach((batch, batchIndex) => {
      console.log(`Processing batch ${batchIndex + 1} of ${batches.length}`);
      
      var batchResults = batch.map(request => {
        try {
          // Check quota
          if (!this.quotaTracker.canMakeRequest()) {
            return {
              url: request.url,
              error: 'Quota exceeded',
              status: 429
            };
          }
          
          var response = UrlFetchApp.fetch(request.url, request.params || {});
          this.quotaTracker.recordRequest();
          
          return {
            url: request.url,
            status: response.getResponseCode(),
            headers: response.getAllHeaders(),
            content: response.getContentText()
          };
          
        } catch (error) {
          return {
            url: request.url,
            error: error.message,
            status: 0
          };
        }
      });
      
      results = results.concat(batchResults);
      
      // Pause between batches
      if (batchIndex < batches.length - 1) {
        Utilities.sleep(1000);
      }
    });
    
    return results;
  }
  
  createBatches(requests) {
    var batches = [];
    
    for (var i = 0; i < requests.length; i += this.maxConcurrent) {
      batches.push(requests.slice(i, i + this.maxConcurrent));
    }
    
    return batches;
  }
}
```

## Gmail Quota Management

### Email Sending Limits

```javascript
// Gmail quota manager
class GmailQuotaManager {
  constructor() {
    this.cache = CacheService.getScriptCache();
    this.recipientKey = 'gmail_recipients_' + new Date().toDateString();
    this.readWriteKey = 'gmail_operations_' + new Date().toDateString();
  }
  
  canSendEmail(recipientCount = 1) {
    var quota = MailApp.getRemainingDailyQuota();
    return quota >= recipientCount;
  }
  
  sendEmail(options) {
    // Count recipients
    var recipientCount = 1; // to
    if (options.cc) recipientCount += options.cc.split(',').length;
    if (options.bcc) recipientCount += options.bcc.split(',').length;
    
    if (!this.canSendEmail(recipientCount)) {
      return this.queueEmail(options);
    }
    
    try {
      MailApp.sendEmail(options);
      this.recordRecipients(recipientCount);
      return {success: true};
      
    } catch (error) {
      if (error.message.includes('Service invoked too many times')) {
        return this.queueEmail(options);
      }
      throw error;
    }
  }
  
  queueEmail(options) {
    var props = PropertiesService.getScriptProperties();
    var queue = JSON.parse(props.getProperty('email_queue') || '[]');
    
    queue.push({
      ...options,
      queued: new Date().toISOString()
    });
    
    props.setProperty('email_queue', JSON.stringify(queue));
    
    // Schedule processing for next day
    this.scheduleQueueProcessing();
    
    return {
      success: false,
      queued: true,
      message: 'Email queued due to quota limits'
    };
  }
  
  scheduleQueueProcessing() {
    var triggers = ScriptApp.getProjectTriggers();
    var hasQueueTrigger = triggers.some(t => 
      t.getHandlerFunction() === 'processEmailQueue'
    );
    
    if (!hasQueueTrigger) {
      var tomorrow = new Date();
      tomorrow.setDate(tomorrow.getDate() + 1);
      tomorrow.setHours(0, 30, 0, 0); // 12:30 AM
      
      ScriptApp.newTrigger('processEmailQueue')
        .timeBased()
        .at(tomorrow)
        .create();
    }
  }
  
  recordRecipients(count) {
    var current = parseInt(this.cache.get(this.recipientKey) || '0');
    this.cache.put(this.recipientKey, (current + count).toString(), 86400);
  }
  
  recordOperation() {
    var current = parseInt(this.cache.get(this.readWriteKey) || '0');
    this.cache.put(this.readWriteKey, (current + 1).toString(), 86400);
  }
  
  getQuotaStatus() {
    var remaining = MailApp.getRemainingDailyQuota();
    var recipientsSent = parseInt(this.cache.get(this.recipientKey) || '0');
    var operations = parseInt(this.cache.get(this.readWriteKey) || '0');
    
    return {
      remainingQuota: remaining,
      recipientsSent: recipientsSent,
      operations: operations,
      maxOperations: 20000, // Consumer limit
      percentUsed: ((recipientsSent / (recipientsSent + remaining)) * 100).toFixed(2)
    };
  }
}

// Process email queue
function processEmailQueue() {
  var props = PropertiesService.getScriptProperties();
  var queue = JSON.parse(props.getProperty('email_queue') || '[]');
  
  if (queue.length === 0) return;
  
  var manager = new GmailQuotaManager();
  var processed = [];
  var remaining = [];
  
  queue.forEach(email => {
    if (manager.canSendEmail()) {
      try {
        MailApp.sendEmail(email);
        processed.push(email);
      } catch (error) {
        console.error('Failed to send queued email:', error);
        remaining.push(email);
      }
    } else {
      remaining.push(email);
    }
  });
  
  // Update queue
  props.setProperty('email_queue', JSON.stringify(remaining));
  
  console.log(`Processed ${processed.length} emails, ${remaining.length} remaining`);
}
```

## Drive Quota Management

### File Operation Limits

```javascript
// Drive quota manager
class DriveQuotaManager {
  constructor() {
    this.cache = CacheService.getScriptCache();
    this.createKey = 'drive_creates_' + new Date().toDateString();
    this.modifyKey = 'drive_modifies_' + new Date().toDateString();
  }
  
  canCreateFile() {
    var created = parseInt(this.cache.get(this.createKey) || '0');
    var limit = this.getCreateLimit();
    return created < limit;
  }
  
  canModifyFile() {
    var modified = parseInt(this.cache.get(this.modifyKey) || '0');
    var limit = this.getModifyLimit();
    return modified < limit;
  }
  
  getCreateLimit() {
    var isWorkspace = Session.getActiveUser().getEmail().includes('@');
    return isWorkspace ? 500 : 250;
  }
  
  getModifyLimit() {
    var isWorkspace = Session.getActiveUser().getEmail().includes('@');
    return isWorkspace ? 10000 : 2500;
  }
  
  recordCreate() {
    var current = parseInt(this.cache.get(this.createKey) || '0');
    this.cache.put(this.createKey, (current + 1).toString(), 86400);
  }
  
  recordModify() {
    var current = parseInt(this.cache.get(this.modifyKey) || '0');
    this.cache.put(this.modifyKey, (current + 1).toString(), 86400);
  }
  
  createFile(name, content, mimeType) {
    if (!this.canCreateFile()) {
      throw new Error('Drive file creation quota exceeded');
    }
    
    try {
      var file = DriveApp.createFile(name, content, mimeType);
      this.recordCreate();
      return file;
      
    } catch (error) {
      if (error.message.includes('Service invoked too many times')) {
        // Update counter to reflect limit
        this.cache.put(this.createKey, this.getCreateLimit().toString(), 86400);
      }
      throw error;
    }
  }
  
  getQuotaStatus() {
    var created = parseInt(this.cache.get(this.createKey) || '0');
    var modified = parseInt(this.cache.get(this.modifyKey) || '0');
    
    return {
      filesCreated: created,
      createLimit: this.getCreateLimit(),
      createRemaining: this.getCreateLimit() - created,
      filesModified: modified,
      modifyLimit: this.getModifyLimit(),
      modifyRemaining: this.getModifyLimit() - modified
    };
  }
}
```

## Quota Monitoring Dashboard

```javascript
// Comprehensive quota monitoring
function createQuotaDashboard() {
  var ss = SpreadsheetApp.create('Apps Script Quota Dashboard');
  var sheet = ss.getActiveSheet();
  
  // Set headers
  sheet.getRange(1, 1, 1, 4).setValues([['Service', 'Used', 'Limit', '% Used']]);
  sheet.getRange(1, 1, 1, 4).setFontWeight('bold');
  
  var row = 2;
  
  // Email quota
  var emailQuota = new GmailQuotaManager().getQuotaStatus();
  sheet.getRange(row++, 1, 1, 4).setValues([[
    'Email Recipients',
    emailQuota.recipientsSent,
    emailQuota.recipientsSent + emailQuota.remainingQuota,
    emailQuota.percentUsed + '%'
  ]]);
  
  // URL Fetch quota
  var urlQuota = new URLFetchQuotaTracker().getRemainingQuota();
  sheet.getRange(row++, 1, 1, 4).setValues([[
    'URL Fetch',
    urlQuota.used,
    urlQuota.limit,
    urlQuota.percentUsed + '%'
  ]]);
  
  // Drive quota
  var driveQuota = new DriveQuotaManager().getQuotaStatus();
  sheet.getRange(row++, 1, 1, 4).setValues([[
    'Drive Creates',
    driveQuota.filesCreated,
    driveQuota.createLimit,
    ((driveQuota.filesCreated / driveQuota.createLimit) * 100).toFixed(2) + '%'
  ]]);
  
  sheet.getRange(row++, 1, 1, 4).setValues([[
    'Drive Modifies',
    driveQuota.filesModified,
    driveQuota.modifyLimit,
    ((driveQuota.filesModified / driveQuota.modifyLimit) * 100).toFixed(2) + '%'
  ]]);
  
  // Properties storage
  var propsManager = new PropertiesManager();
  var propsInfo = propsManager.getStorageInfo();
  sheet.getRange(row++, 1, 1, 4).setValues([[
    'Properties Storage',
    Math.round(propsInfo.totalSize / 1024) + ' KB',
    '500 KB',
    propsInfo.usagePercent + '%'
  ]]);
  
  // Triggers
  var triggerCount = ScriptApp.getProjectTriggers().length;
  sheet.getRange(row++, 1, 1, 4).setValues([[
    'Triggers',
    triggerCount,
    20,
    ((triggerCount / 20) * 100).toFixed(2) + '%'
  ]]);
  
  // Add conditional formatting
  var range = sheet.getRange(2, 4, row - 2, 1);
  var rule = SpreadsheetApp.newConditionalFormatRule()
    .setRanges([range])
    .whenNumberGreaterThan(80)
    .setBackground('#f4cccc')
    .build();
  
  sheet.setConditionalFormatRules([rule]);
  
  // Auto-resize columns
  sheet.autoResizeColumns(1, 4);
  
  return ss.getUrl();
}

// Set up quota monitoring
function setupQuotaMonitoring() {
  // Create daily quota report trigger
  ScriptApp.newTrigger('sendQuotaReport')
    .timeBased()
    .everyDays(1)
    .atHour(23) // 11 PM
    .create();
}

function sendQuotaReport() {
  var dashboard = createQuotaDashboard();
  
  MailApp.sendEmail({
    to: Session.getActiveUser().getEmail(),
    subject: 'Daily Apps Script Quota Report',
    body: 'Your daily quota usage report is ready: ' + dashboard
  });
}
```

## Best Practices for Quota Management

1. **Monitor usage proactively** - Don't wait for errors
2. **Implement quota tracking** - Know your limits
3. **Use batch operations** - Minimize API calls
4. **Cache aggressively** - Reduce repeated operations
5. **Queue operations** - Handle quota exceeded gracefully
6. **Set up alerts** - Get notified before hitting limits
7. **Plan for peak usage** - Design with quotas in mind
8. **Use triggers wisely** - Don't exceed trigger limits
9. **Optimize data storage** - Stay within property limits
10. **Document quota impacts** - Help users understand limits

## Quota Error Recovery

1. **Implement exponential backoff** for retries
2. **Queue failed operations** for later processing
3. **Use continuation patterns** for long-running scripts
4. **Provide user feedback** about quota status
5. **Switch to batch mode** when approaching limits
6. **Use multiple projects** if necessary for higher limits
7. **Consider paid alternatives** for production use
8. **Monitor and adjust** based on usage patterns
9. **Implement circuit breakers** to prevent cascading failures
10. **Plan maintenance windows** for quota-heavy operations