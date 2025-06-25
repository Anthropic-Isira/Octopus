# Error Handling in Google Apps Script

[← Back to Main Documentation](../README.md)

## Overview

Robust error handling is essential for production Apps Script applications. This guide covers error types, handling strategies, and recovery patterns.

## Error Types in Apps Script

### JavaScript Errors

```javascript
// Common JavaScript errors
function commonErrors() {
  // TypeError
  try {
    var obj = null;
    obj.property; // TypeError: Cannot read property of null
  } catch (e) {
    console.error('TypeError:', e.message);
  }
  
  // ReferenceError
  try {
    console.log(undefinedVariable); // ReferenceError
  } catch (e) {
    console.error('ReferenceError:', e.message);
  }
  
  // RangeError
  try {
    var arr = new Array(-1); // RangeError: Invalid array length
  } catch (e) {
    console.error('RangeError:', e.message);
  }
  
  // SyntaxError (caught at parse time)
  try {
    eval('var x = ;'); // SyntaxError
  } catch (e) {
    console.error('SyntaxError:', e.message);
  }
}

// Custom errors
class CustomError extends Error {
  constructor(message, code) {
    super(message);
    this.name = 'CustomError';
    this.code = code;
    this.timestamp = new Date();
  }
}

function throwCustomError() {
  throw new CustomError('Something went wrong', 'ERR_001');
}
```

### Service-Specific Errors

```javascript
// Google Service errors
function serviceErrors() {
  // Drive API errors
  try {
    var file = DriveApp.getFileById('invalid-id');
  } catch (e) {
    if (e.message.includes('not found')) {
      console.error('File not found');
    } else if (e.message.includes('permission')) {
      console.error('Permission denied');
    }
  }
  
  // Spreadsheet errors
  try {
    var sheet = SpreadsheetApp.openById('invalid-id');
  } catch (e) {
    // Exception: Unexpected error while getting the method or property openById on object SpreadsheetApp
    handleSpreadsheetError(e);
  }
  
  // Gmail quota errors
  try {
    // Send too many emails
    for (var i = 0; i < 1000; i++) {
      MailApp.sendEmail('test@example.com', 'Test', 'Body');
    }
  } catch (e) {
    if (e.message.includes('Service invoked too many times')) {
      console.error('Gmail quota exceeded');
    }
  }
}

function handleSpreadsheetError(error) {
  var errorMessage = error.toString();
  
  if (errorMessage.includes('not found')) {
    return 'Spreadsheet does not exist or you don\'t have access';
  } else if (errorMessage.includes('timeout')) {
    return 'Operation timed out - spreadsheet may be too large';
  } else if (errorMessage.includes('permission')) {
    return 'You don\'t have permission to access this spreadsheet';
  } else {
    return 'Unknown spreadsheet error: ' + errorMessage;
  }
}
```

## Basic Error Handling Patterns

### Try-Catch-Finally

```javascript
function basicErrorHandling() {
  var resource = null;
  
  try {
    // Acquire resource
    resource = acquireResource();
    
    // Risky operation
    var result = performOperation(resource);
    
    return result;
    
  } catch (error) {
    // Handle error
    console.error('Operation failed:', error);
    
    // Log error details
    logError(error);
    
    // Return default or throw
    return null;
    
  } finally {
    // Always cleanup
    if (resource) {
      releaseResource(resource);
    }
  }
}

// Nested error handling
function nestedErrorHandling() {
  try {
    // Outer operation
    var data = fetchData();
    
    try {
      // Inner operation
      return processData(data);
      
    } catch (processError) {
      console.error('Processing failed:', processError);
      
      // Try alternative processing
      return processDataAlternative(data);
    }
    
  } catch (fetchError) {
    console.error('Fetch failed:', fetchError);
    
    // Use cached data as fallback
    return getCachedData();
  }
}
```

### Error Propagation

```javascript
// Rethrowing errors with context
function operationWithContext() {
  try {
    return riskyOperation();
  } catch (error) {
    // Add context and rethrow
    error.context = {
      operation: 'operationWithContext',
      timestamp: new Date(),
      user: Session.getActiveUser().getEmail()
    };
    
    throw error;
  }
}

// Wrapping errors
function wrapError(error, message) {
  var wrapped = new Error(message + ': ' + error.message);
  wrapped.originalError = error;
  wrapped.stack = error.stack;
  return wrapped;
}

// Error transformation
function transformError(error) {
  if (error.message.includes('not found')) {
    return new Error('The requested resource does not exist');
  } else if (error.message.includes('permission')) {
    return new Error('You do not have access to this resource');
  } else {
    return error;
  }
}
```

## Advanced Error Handling

### Retry Logic with Exponential Backoff

```javascript
class RetryHandler {
  constructor(options = {}) {
    this.maxRetries = options.maxRetries || 3;
    this.initialDelay = options.initialDelay || 1000;
    this.maxDelay = options.maxDelay || 60000;
    this.backoffFactor = options.backoffFactor || 2;
    this.retryableErrors = options.retryableErrors || [
      'Service unavailable',
      'Request timeout',
      'Too many requests',
      'Internal error'
    ];
  }
  
  async execute(func, context = {}) {
    var lastError;
    
    for (var attempt = 0; attempt <= this.maxRetries; attempt++) {
      try {
        return await func();
        
      } catch (error) {
        lastError = error;
        
        if (!this.isRetryable(error) || attempt === this.maxRetries) {
          throw this.enhanceError(error, attempt, context);
        }
        
        var delay = this.calculateDelay(attempt);
        console.log(`Retry ${attempt + 1}/${this.maxRetries} after ${delay}ms`);
        
        Utilities.sleep(delay);
      }
    }
    
    throw lastError;
  }
  
  isRetryable(error) {
    var errorMessage = error.toString().toLowerCase();
    return this.retryableErrors.some(msg => 
      errorMessage.includes(msg.toLowerCase())
    );
  }
  
  calculateDelay(attempt) {
    var delay = this.initialDelay * Math.pow(this.backoffFactor, attempt);
    
    // Add jitter
    delay = delay * (0.5 + Math.random() * 0.5);
    
    return Math.min(delay, this.maxDelay);
  }
  
  enhanceError(error, attempts, context) {
    error.retryAttempts = attempts;
    error.context = context;
    error.timestamp = new Date();
    return error;
  }
}

// Usage
function fetchWithRetry(url) {
  var retryHandler = new RetryHandler({
    maxRetries: 5,
    initialDelay: 2000
  });
  
  return retryHandler.execute(function() {
    var response = UrlFetchApp.fetch(url);
    
    if (response.getResponseCode() >= 500) {
      throw new Error('Service unavailable');
    }
    
    return response;
  }, {url: url});
}
```

### Circuit Breaker Pattern

```javascript
class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5;
    this.resetTimeout = options.resetTimeout || 60000; // 1 minute
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.failures = 0;
    this.lastFailureTime = null;
    this.successCount = 0;
    this.props = PropertiesService.getScriptProperties();
  }
  
  async execute(func, fallback) {
    // Load state from properties
    this.loadState();
    
    if (this.state === 'OPEN') {
      if (this.shouldAttemptReset()) {
        this.state = 'HALF_OPEN';
      } else {
        console.log('Circuit breaker is OPEN');
        return fallback ? fallback() : Promise.reject(new Error('Circuit breaker is OPEN'));
      }
    }
    
    try {
      var result = await func();
      this.onSuccess();
      return result;
      
    } catch (error) {
      this.onFailure();
      
      if (fallback) {
        return fallback();
      }
      
      throw error;
    }
  }
  
  onSuccess() {
    this.failures = 0;
    
    if (this.state === 'HALF_OPEN') {
      this.successCount++;
      
      if (this.successCount > 2) {
        this.state = 'CLOSED';
        console.log('Circuit breaker is now CLOSED');
      }
    }
    
    this.saveState();
  }
  
  onFailure() {
    this.failures++;
    this.lastFailureTime = new Date().getTime();
    
    if (this.failures >= this.failureThreshold) {
      this.state = 'OPEN';
      console.log('Circuit breaker is now OPEN');
    }
    
    this.saveState();
  }
  
  shouldAttemptReset() {
    return new Date().getTime() - this.lastFailureTime > this.resetTimeout;
  }
  
  loadState() {
    var state = this.props.getProperty('circuit_breaker_state');
    if (state) {
      var parsed = JSON.parse(state);
      Object.assign(this, parsed);
    }
  }
  
  saveState() {
    this.props.setProperty('circuit_breaker_state', JSON.stringify({
      state: this.state,
      failures: this.failures,
      lastFailureTime: this.lastFailureTime,
      successCount: this.successCount
    }));
  }
}

// Usage
var apiBreaker = new CircuitBreaker({
  failureThreshold: 3,
  resetTimeout: 30000
});

function callExternalAPI() {
  return apiBreaker.execute(
    function() {
      // Primary function
      return UrlFetchApp.fetch('https://api.example.com/data');
    },
    function() {
      // Fallback function
      return getCachedData();
    }
  );
}
```

### Error Logging and Monitoring

```javascript
class ErrorLogger {
  constructor(options = {}) {
    this.logSheet = options.logSheet || this.createLogSheet();
    this.maxLogs = options.maxLogs || 1000;
    this.notificationEmail = options.notificationEmail;
    this.errorThreshold = options.errorThreshold || 10;
    this.cache = CacheService.getScriptCache();
  }
  
  createLogSheet() {
    var ss = SpreadsheetApp.getActiveSpreadsheet() || 
             SpreadsheetApp.create('Error Logs');
    
    var sheet = ss.getSheetByName('Errors') || 
                ss.insertSheet('Errors');
    
    // Set headers
    if (sheet.getLastRow() === 0) {
      sheet.getRange(1, 1, 1, 8).setValues([[
        'Timestamp', 'Error Type', 'Message', 'Stack Trace', 
        'User', 'Function', 'Context', 'Resolved'
      ]]);
    }
    
    return sheet;
  }
  
  log(error, context = {}) {
    try {
      var errorData = this.parseError(error, context);
      
      // Log to sheet
      this.logSheet.appendRow([
        errorData.timestamp,
        errorData.type,
        errorData.message,
        errorData.stack,
        errorData.user,
        errorData.function,
        JSON.stringify(errorData.context),
        false
      ]);
      
      // Trim old logs
      this.trimLogs();
      
      // Check for notification
      this.checkErrorThreshold();
      
      // Also log to console
      console.error('Error logged:', errorData);
      
    } catch (logError) {
      // Fallback to console only
      console.error('Failed to log error:', logError);
      console.error('Original error:', error);
    }
  }
  
  parseError(error, context) {
    return {
      timestamp: new Date(),
      type: error.name || 'Error',
      message: error.message || error.toString(),
      stack: error.stack || 'No stack trace',
      user: Session.getActiveUser().getEmail(),
      function: context.function || 'Unknown',
      context: context
    };
  }
  
  trimLogs() {
    var rowCount = this.logSheet.getLastRow();
    
    if (rowCount > this.maxLogs + 1) {
      var rowsToDelete = rowCount - this.maxLogs;
      this.logSheet.deleteRows(2, rowsToDelete);
    }
  }
  
  checkErrorThreshold() {
    var cacheKey = 'error_count_' + new Date().toDateString();
    var count = parseInt(this.cache.get(cacheKey) || '0') + 1;
    
    this.cache.put(cacheKey, count.toString(), 86400); // 24 hours
    
    if (count === this.errorThreshold && this.notificationEmail) {
      this.sendNotification(count);
    }
  }
  
  sendNotification(errorCount) {
    try {
      MailApp.sendEmail({
        to: this.notificationEmail,
        subject: 'Error Threshold Reached',
        body: `The application has logged ${errorCount} errors today. ` +
              `Please check the error log: ${this.logSheet.getParent().getUrl()}`
      });
    } catch (e) {
      console.error('Failed to send error notification:', e);
    }
  }
  
  getRecentErrors(hours = 24) {
    var cutoff = new Date();
    cutoff.setHours(cutoff.getHours() - hours);
    
    var data = this.logSheet.getDataRange().getValues();
    var headers = data[0];
    var errors = [];
    
    for (var i = 1; i < data.length; i++) {
      var row = data[i];
      var timestamp = row[0];
      
      if (timestamp > cutoff) {
        var error = {};
        headers.forEach((header, index) => {
          error[header] = row[index];
        });
        errors.push(error);
      }
    }
    
    return errors;
  }
}

// Global error logger
var errorLogger = new ErrorLogger({
  notificationEmail: 'admin@example.com'
});

// Global error handler
function handleError(error, context) {
  errorLogger.log(error, context);
  
  // Rethrow for debugging
  if (isDebugMode()) {
    throw error;
  }
}

function isDebugMode() {
  return PropertiesService.getScriptProperties()
    .getProperty('DEBUG_MODE') === 'true';
}
```

### Error Recovery Strategies

```javascript
// Automatic recovery with fallbacks
class RecoveryManager {
  constructor() {
    this.strategies = [];
  }
  
  addStrategy(condition, recovery) {
    this.strategies.push({condition, recovery});
    return this;
  }
  
  async execute(func) {
    try {
      return await func();
    } catch (error) {
      for (var strategy of this.strategies) {
        if (strategy.condition(error)) {
          console.log('Applying recovery strategy');
          
          try {
            return await strategy.recovery(error);
          } catch (recoveryError) {
            console.error('Recovery failed:', recoveryError);
          }
        }
      }
      
      throw error;
    }
  }
}

// Usage
function robustDataFetch() {
  var recovery = new RecoveryManager();
  
  recovery
    .addStrategy(
      error => error.message.includes('timeout'),
      async () => {
        console.log('Timeout detected, using cached data');
        return getCachedData();
      }
    )
    .addStrategy(
      error => error.message.includes('not found'),
      async () => {
        console.log('Creating default data');
        return createDefaultData();
      }
    )
    .addStrategy(
      error => error.message.includes('permission'),
      async () => {
        console.log('Requesting access');
        requestAccess();
        throw new Error('Access requested, please try again later');
      }
    );
  
  return recovery.execute(async () => {
    return fetchDataFromAPI();
  });
}
```

### Graceful Degradation

```javascript
// Feature flags for graceful degradation
class FeatureFlags {
  constructor() {
    this.props = PropertiesService.getScriptProperties();
  }
  
  isEnabled(feature) {
    var flag = this.props.getProperty('feature_' + feature);
    return flag === 'true';
  }
  
  withFallback(feature, primaryFunc, fallbackFunc) {
    if (this.isEnabled(feature)) {
      try {
        return primaryFunc();
      } catch (error) {
        console.error(`Feature ${feature} failed:`, error);
        
        // Disable feature
        this.props.setProperty('feature_' + feature, 'false');
        
        // Use fallback
        return fallbackFunc();
      }
    } else {
      return fallbackFunc();
    }
  }
}

// Usage
var features = new FeatureFlags();

function processData(data) {
  return features.withFallback(
    'advanced_processing',
    () => advancedDataProcessing(data),
    () => basicDataProcessing(data)
  );
}
```

## Error Handling Best Practices

### 1. Structured Error Handling

```javascript
// Error handler class
class ErrorHandler {
  static handle(error, options = {}) {
    var {
      context = {},
      notify = true,
      rethrow = false,
      fallback = null
    } = options;
    
    // Log error
    this.log(error, context);
    
    // Notify if critical
    if (notify && this.isCritical(error)) {
      this.notify(error, context);
    }
    
    // Execute fallback
    if (fallback) {
      try {
        return fallback();
      } catch (fallbackError) {
        this.log(fallbackError, {...context, isFallback: true});
      }
    }
    
    // Rethrow if requested
    if (rethrow) {
      throw error;
    }
  }
  
  static log(error, context) {
    console.error('Error:', {
      message: error.message,
      stack: error.stack,
      context: context,
      timestamp: new Date()
    });
  }
  
  static isCritical(error) {
    var criticalPatterns = [
      'quota exceeded',
      'authorization required',
      'service unavailable'
    ];
    
    var errorString = error.toString().toLowerCase();
    return criticalPatterns.some(pattern => 
      errorString.includes(pattern)
    );
  }
  
  static notify(error, context) {
    // Implement notification logic
    console.warn('Critical error notification:', error.message);
  }
}
```

### 2. Error Boundaries

```javascript
// Wrap functions with error boundaries
function withErrorBoundary(func, options = {}) {
  return function(...args) {
    try {
      return func.apply(this, args);
    } catch (error) {
      ErrorHandler.handle(error, {
        context: {
          function: func.name,
          arguments: args
        },
        ...options
      });
    }
  };
}

// Usage
var safeFunction = withErrorBoundary(riskyFunction, {
  fallback: () => 'Default value',
  notify: true
});
```

### 3. Validation and Guards

```javascript
// Input validation
function validateInput(input, schema) {
  var errors = [];
  
  for (var field in schema) {
    var rule = schema[field];
    var value = input[field];
    
    if (rule.required && !value) {
      errors.push(`${field} is required`);
    }
    
    if (value && rule.type && typeof value !== rule.type) {
      errors.push(`${field} must be ${rule.type}`);
    }
    
    if (value && rule.validator && !rule.validator(value)) {
      errors.push(`${field} is invalid`);
    }
  }
  
  if (errors.length > 0) {
    throw new ValidationError(errors);
  }
}

class ValidationError extends Error {
  constructor(errors) {
    super('Validation failed');
    this.name = 'ValidationError';
    this.errors = errors;
  }
}

// Guard clauses
function processOrder(order) {
  if (!order) {
    throw new Error('Order is required');
  }
  
  if (!order.items || order.items.length === 0) {
    throw new Error('Order must have at least one item');
  }
  
  if (order.total <= 0) {
    throw new Error('Order total must be positive');
  }
  
  // Process valid order
  return processValidOrder(order);
}
```

## Common Error Patterns and Solutions

### Quota Errors

```javascript
function handleQuotaError(operation) {
  var quotaKey = 'quota_' + operation;
  var props = PropertiesService.getScriptProperties();
  
  try {
    // Check quota before operation
    var usage = parseInt(props.getProperty(quotaKey) || '0');
    var limit = getQuotaLimit(operation);
    
    if (usage >= limit) {
      throw new Error('Quota exceeded for ' + operation);
    }
    
    // Perform operation
    var result = performOperation(operation);
    
    // Update usage
    props.setProperty(quotaKey, (usage + 1).toString());
    
    return result;
    
  } catch (error) {
    if (error.message.includes('Service invoked too many times')) {
      // Reset counter at midnight
      var tomorrow = new Date();
      tomorrow.setDate(tomorrow.getDate() + 1);
      tomorrow.setHours(0, 0, 0, 0);
      
      ScriptApp.newTrigger('resetQuota')
        .timeBased()
        .at(tomorrow)
        .create();
      
      throw new Error('Daily quota exceeded. Will reset at midnight.');
    }
    
    throw error;
  }
}
```

### Timeout Errors

```javascript
function handleTimeout(func, timeoutMs = 300000) {
  var startTime = new Date().getTime();
  
  function checkTimeout() {
    if (new Date().getTime() - startTime > timeoutMs) {
      throw new Error('Operation timed out');
    }
  }
  
  // For iterative operations
  return function(items) {
    var processed = [];
    
    for (var i = 0; i < items.length; i++) {
      checkTimeout();
      
      try {
        processed.push(func(items[i]));
      } catch (error) {
        // Save progress
        saveProgress(i, processed);
        throw error;
      }
    }
    
    return processed;
  };
}
```

## Summary

Key error handling principles:

1. **Always expect failures** - External services, quotas, timeouts
2. **Fail fast** with clear error messages
3. **Log everything** for debugging
4. **Implement retries** for transient failures
5. **Use circuit breakers** for external dependencies
6. **Provide fallbacks** for critical operations
7. **Monitor and alert** on error patterns
8. **Document error codes** and recovery procedures
9. **Test error paths** not just happy paths
10. **Learn from failures** and improve handling