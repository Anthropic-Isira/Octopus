# State Management in Google Apps Script

[← Back to Main Documentation](../README.md)

## Overview

State management is crucial for building robust Apps Script applications. Since scripts run in a stateless environment, you need strategies to persist data between executions.

## Properties Service

The Properties Service is the primary way to store simple key-value pairs.

### Script Properties

Shared across all users of the script.

```javascript
// Basic usage
function scriptPropertiesExample() {
  var scriptProps = PropertiesService.getScriptProperties();
  
  // Set a single property
  scriptProps.setProperty('API_KEY', 'your-api-key-here');
  
  // Set multiple properties
  scriptProps.setProperties({
    'WEBHOOK_URL': 'https://example.com/webhook',
    'BATCH_SIZE': '100',
    'LAST_RUN': new Date().toISOString()
  });
  
  // Get property
  var apiKey = scriptProps.getProperty('API_KEY');
  console.log('API Key:', apiKey);
  
  // Get all properties
  var allProps = scriptProps.getProperties();
  for (var key in allProps) {
    console.log(key + ': ' + allProps[key]);
  }
  
  // Delete property
  scriptProps.deleteProperty('TEMP_VALUE');
  
  // Delete all properties
  scriptProps.deleteAllProperties();
}

// Configuration management
class Config {
  constructor() {
    this.props = PropertiesService.getScriptProperties();
    this.cache = {};
  }
  
  get(key, defaultValue = null) {
    // Check cache first
    if (this.cache[key] !== undefined) {
      return this.cache[key];
    }
    
    var value = this.props.getProperty(key);
    if (value === null) {
      return defaultValue;
    }
    
    // Parse JSON values
    try {
      value = JSON.parse(value);
    } catch (e) {
      // Not JSON, return as string
    }
    
    this.cache[key] = value;
    return value;
  }
  
  set(key, value) {
    // Convert to string
    var stringValue = typeof value === 'string' 
      ? value 
      : JSON.stringify(value);
    
    this.props.setProperty(key, stringValue);
    this.cache[key] = value;
  }
  
  getNumber(key, defaultValue = 0) {
    var value = this.get(key);
    return value !== null ? Number(value) : defaultValue;
  }
  
  getBoolean(key, defaultValue = false) {
    var value = this.get(key);
    if (value === null) return defaultValue;
    return value === 'true' || value === true;
  }
}
```

### User Properties

Specific to each user running the script.

```javascript
function userPropertiesExample() {
  var userProps = PropertiesService.getUserProperties();
  
  // Store user preferences
  userProps.setProperties({
    'theme': 'dark',
    'language': 'en',
    'timezone': Session.getScriptTimeZone(),
    'lastLogin': new Date().toISOString()
  });
  
  // Get user preference
  var theme = userProps.getProperty('theme') || 'light';
  
  // User-specific state
  var userState = {
    currentStep: 3,
    completedTasks: ['task1', 'task2'],
    settings: {
      notifications: true,
      autoSave: false
    }
  };
  
  userProps.setProperty('state', JSON.stringify(userState));
  
  // Retrieve and parse
  var savedState = userProps.getProperty('state');
  if (savedState) {
    userState = JSON.parse(savedState);
  }
}

// User session manager
class UserSession {
  constructor() {
    this.props = PropertiesService.getUserProperties();
    this.sessionKey = 'user_session';
  }
  
  start() {
    var session = {
      id: Utilities.getUuid(),
      startTime: new Date().toISOString(),
      user: Session.getActiveUser().getEmail(),
      activities: []
    };
    
    this.props.setProperty(this.sessionKey, JSON.stringify(session));
    return session;
  }
  
  get() {
    var sessionData = this.props.getProperty(this.sessionKey);
    return sessionData ? JSON.parse(sessionData) : null;
  }
  
  addActivity(activity) {
    var session = this.get();
    if (!session) {
      session = this.start();
    }
    
    session.activities.push({
      timestamp: new Date().toISOString(),
      action: activity
    });
    
    this.props.setProperty(this.sessionKey, JSON.stringify(session));
  }
  
  end() {
    var session = this.get();
    if (session) {
      session.endTime = new Date().toISOString();
      // Optionally save to permanent storage
      this.archiveSession(session);
    }
    
    this.props.deleteProperty(this.sessionKey);
  }
  
  archiveSession(session) {
    // Save to spreadsheet or database
    console.log('Archiving session:', session.id);
  }
}
```

### Document Properties

For container-bound scripts only.

```javascript
function documentPropertiesExample() {
  var docProps = PropertiesService.getDocumentProperties();
  
  // Store document-specific metadata
  docProps.setProperties({
    'version': '1.0.0',
    'lastModified': new Date().toISOString(),
    'author': Session.getActiveUser().getEmail(),
    'status': 'draft'
  });
  
  // Track document usage
  var viewCount = parseInt(docProps.getProperty('viewCount') || '0');
  docProps.setProperty('viewCount', (viewCount + 1).toString());
  
  // Document state machine
  var docState = {
    currentPhase: 'editing',
    reviewers: ['user1@example.com', 'user2@example.com'],
    approvals: {},
    history: []
  };
  
  docProps.setProperty('documentState', JSON.stringify(docState));
}
```

## Cache Service

For temporary storage with automatic expiration.

```javascript
// Basic cache usage
function cacheServiceExample() {
  var cache = CacheService.getScriptCache();
  
  // Store with expiration (max 21600 seconds = 6 hours)
  cache.put('tempData', 'This will expire', 3600); // 1 hour
  
  // Store multiple values
  cache.putAll({
    'key1': 'value1',
    'key2': 'value2',
    'key3': JSON.stringify({complex: 'object'})
  }, 1800); // 30 minutes
  
  // Retrieve
  var value = cache.get('tempData');
  console.log('Cached value:', value);
  
  // Get multiple
  var values = cache.getAll(['key1', 'key2', 'key3']);
  console.log('Multiple values:', values);
  
  // Remove
  cache.remove('tempData');
  cache.removeAll(['key1', 'key2']);
}

// Advanced caching patterns
class CacheManager {
  constructor(cacheType = 'script') {
    switch(cacheType) {
      case 'user':
        this.cache = CacheService.getUserCache();
        break;
      case 'document':
        this.cache = CacheService.getDocumentCache();
        break;
      default:
        this.cache = CacheService.getScriptCache();
    }
    
    this.defaultTTL = 3600; // 1 hour
  }
  
  get(key, computeFunction = null) {
    var cached = this.cache.get(key);
    
    if (cached) {
      try {
        return JSON.parse(cached);
      } catch (e) {
        return cached; // Return as string if not JSON
      }
    }
    
    if (computeFunction) {
      var value = computeFunction();
      this.set(key, value);
      return value;
    }
    
    return null;
  }
  
  set(key, value, ttl = null) {
    var stringValue = typeof value === 'string' 
      ? value 
      : JSON.stringify(value);
    
    // Check size limit (100KB per key)
    if (stringValue.length > 100000) {
      console.warn('Value too large for cache:', key);
      return false;
    }
    
    this.cache.put(key, stringValue, ttl || this.defaultTTL);
    return true;
  }
  
  invalidate(pattern) {
    // Cache doesn't support pattern matching, so track keys
    var keys = this.getKeys();
    var toRemove = keys.filter(key => key.includes(pattern));
    
    if (toRemove.length > 0) {
      this.cache.removeAll(toRemove);
    }
  }
  
  getKeys() {
    // This would need to be tracked separately
    var props = PropertiesService.getScriptProperties();
    var keysJson = props.getProperty('cache_keys') || '[]';
    return JSON.parse(keysJson);
  }
}

// Memoization decorator
function memoize(func, ttl = 3600) {
  var cache = CacheService.getScriptCache();
  
  return function() {
    var args = Array.prototype.slice.call(arguments);
    var key = 'memo_' + func.name + '_' + JSON.stringify(args);
    
    var cached = cache.get(key);
    if (cached) {
      return JSON.parse(cached);
    }
    
    var result = func.apply(this, args);
    cache.put(key, JSON.stringify(result), ttl);
    
    return result;
  };
}

// Usage
var expensiveOperation = memoize(function(param) {
  // Simulate expensive operation
  Utilities.sleep(2000);
  return 'Result for ' + param;
}, 300); // Cache for 5 minutes
```

## Lock Service

Prevents concurrent access to shared resources.

```javascript
// Basic lock usage
function lockServiceExample() {
  var lock = LockService.getScriptLock();
  
  try {
    // Wait up to 10 seconds for lock
    lock.waitLock(10000);
    
    // Critical section - only one execution at a time
    var props = PropertiesService.getScriptProperties();
    var counter = parseInt(props.getProperty('counter') || '0');
    counter++;
    props.setProperty('counter', counter.toString());
    
    console.log('Counter incremented to:', counter);
    
  } catch (e) {
    console.error('Could not obtain lock:', e);
  } finally {
    lock.releaseLock();
  }
}

// Advanced lock patterns
class ResourceLock {
  constructor(resourceId, lockType = 'script') {
    this.resourceId = resourceId;
    this.lockType = lockType;
  }
  
  acquire(timeoutMs = 10000) {
    switch(this.lockType) {
      case 'user':
        this.lock = LockService.getUserLock();
        break;
      case 'document':
        this.lock = LockService.getDocumentLock();
        break;
      default:
        this.lock = LockService.getScriptLock();
    }
    
    var acquired = this.lock.tryLock(timeoutMs);
    if (!acquired) {
      throw new Error('Failed to acquire lock for: ' + this.resourceId);
    }
    
    return this;
  }
  
  release() {
    if (this.lock && this.lock.hasLock()) {
      this.lock.releaseLock();
    }
  }
  
  executeWithLock(func, timeoutMs = 10000) {
    this.acquire(timeoutMs);
    
    try {
      return func();
    } finally {
      this.release();
    }
  }
}

// Usage
function safeIncrement() {
  var lock = new ResourceLock('counter_resource');
  
  return lock.executeWithLock(function() {
    // Protected operation
    var props = PropertiesService.getScriptProperties();
    var value = parseInt(props.getProperty('shared_counter') || '0');
    value++;
    props.setProperty('shared_counter', value.toString());
    return value;
  });
}

// Distributed lock with timeout
class DistributedLock {
  constructor(lockName) {
    this.lockName = lockName;
    this.props = PropertiesService.getScriptProperties();
    this.lockKey = 'lock_' + lockName;
    this.ownerKey = 'lock_owner_' + lockName;
  }
  
  acquire(timeoutMs = 30000) {
    var lockId = Utilities.getUuid();
    var startTime = new Date().getTime();
    
    while (new Date().getTime() - startTime < timeoutMs) {
      var currentLock = this.props.getProperty(this.lockKey);
      
      if (!currentLock || this.isLockExpired(currentLock)) {
        // Try to acquire lock
        var lockData = {
          id: lockId,
          timestamp: new Date().getTime(),
          expires: new Date().getTime() + 60000 // 1 minute TTL
        };
        
        this.props.setProperty(this.lockKey, JSON.stringify(lockData));
        this.props.setProperty(this.ownerKey, lockId);
        
        // Verify we got the lock (handle race condition)
        Utilities.sleep(100);
        var verifyOwner = this.props.getProperty(this.ownerKey);
        
        if (verifyOwner === lockId) {
          this.lockId = lockId;
          return true;
        }
      }
      
      Utilities.sleep(1000); // Wait 1 second before retry
    }
    
    return false;
  }
  
  release() {
    if (this.lockId) {
      var currentOwner = this.props.getProperty(this.ownerKey);
      
      if (currentOwner === this.lockId) {
        this.props.deleteProperty(this.lockKey);
        this.props.deleteProperty(this.ownerKey);
      }
    }
  }
  
  isLockExpired(lockData) {
    try {
      var lock = JSON.parse(lockData);
      return new Date().getTime() > lock.expires;
    } catch (e) {
      return true;
    }
  }
}
```

## State Persistence Patterns

### State Machine Implementation

```javascript
class StateMachine {
  constructor(machineId, initialState = 'idle') {
    this.machineId = machineId;
    this.props = PropertiesService.getScriptProperties();
    this.stateKey = 'state_' + machineId;
    this.historyKey = 'history_' + machineId;
    
    // Define valid transitions
    this.transitions = {
      'idle': ['processing', 'error'],
      'processing': ['completed', 'error', 'paused'],
      'paused': ['processing', 'error'],
      'completed': ['idle'],
      'error': ['idle', 'processing']
    };
    
    // Initialize state
    if (!this.getState()) {
      this.setState(initialState);
    }
  }
  
  getState() {
    var stateData = this.props.getProperty(this.stateKey);
    return stateData ? JSON.parse(stateData) : null;
  }
  
  setState(newState, metadata = {}) {
    var currentState = this.getState();
    
    // Validate transition
    if (currentState && !this.canTransition(currentState.state, newState)) {
      throw new Error(`Invalid transition from ${currentState.state} to ${newState}`);
    }
    
    var stateData = {
      state: newState,
      timestamp: new Date().toISOString(),
      metadata: metadata
    };
    
    this.props.setProperty(this.stateKey, JSON.stringify(stateData));
    this.addToHistory(currentState, stateData);
    
    return stateData;
  }
  
  canTransition(fromState, toState) {
    var validTransitions = this.transitions[fromState] || [];
    return validTransitions.includes(toState);
  }
  
  addToHistory(oldState, newState) {
    var history = this.getHistory();
    history.push({
      from: oldState,
      to: newState,
      timestamp: new Date().toISOString()
    });
    
    // Keep only last 100 entries
    if (history.length > 100) {
      history = history.slice(-100);
    }
    
    this.props.setProperty(this.historyKey, JSON.stringify(history));
  }
  
  getHistory() {
    var historyData = this.props.getProperty(this.historyKey);
    return historyData ? JSON.parse(historyData) : [];
  }
}

// Usage
function processWorkflow() {
  var workflow = new StateMachine('order_workflow');
  
  try {
    workflow.setState('processing', {orderId: '12345'});
    
    // Do processing
    processOrder();
    
    workflow.setState('completed', {completedAt: new Date()});
    
  } catch (error) {
    workflow.setState('error', {error: error.toString()});
    throw error;
  }
}
```

### Persistent Queue

```javascript
class PersistentQueue {
  constructor(queueName) {
    this.queueName = queueName;
    this.props = PropertiesService.getScriptProperties();
    this.queueKey = 'queue_' + queueName;
    this.maxSize = 1000; // Adjust based on property size limits
  }
  
  enqueue(item) {
    var lock = LockService.getScriptLock();
    lock.waitLock(10000);
    
    try {
      var queue = this.getQueue();
      queue.push({
        id: Utilities.getUuid(),
        data: item,
        timestamp: new Date().toISOString()
      });
      
      // Enforce max size
      if (queue.length > this.maxSize) {
        queue.shift(); // Remove oldest
      }
      
      this.saveQueue(queue);
      return true;
      
    } finally {
      lock.releaseLock();
    }
  }
  
  dequeue() {
    var lock = LockService.getScriptLock();
    lock.waitLock(10000);
    
    try {
      var queue = this.getQueue();
      if (queue.length === 0) return null;
      
      var item = queue.shift();
      this.saveQueue(queue);
      
      return item;
      
    } finally {
      lock.releaseLock();
    }
  }
  
  peek() {
    var queue = this.getQueue();
    return queue.length > 0 ? queue[0] : null;
  }
  
  size() {
    return this.getQueue().length;
  }
  
  getQueue() {
    var queueData = this.props.getProperty(this.queueKey);
    return queueData ? JSON.parse(queueData) : [];
  }
  
  saveQueue(queue) {
    this.props.setProperty(this.queueKey, JSON.stringify(queue));
  }
  
  clear() {
    this.props.deleteProperty(this.queueKey);
  }
}

// Queue processor
function processQueue() {
  var queue = new PersistentQueue('email_queue');
  var processed = 0;
  var maxProcess = 50;
  
  while (processed < maxProcess) {
    var item = queue.dequeue();
    if (!item) break;
    
    try {
      // Process item
      sendEmail(item.data);
      processed++;
      
    } catch (error) {
      console.error('Failed to process item:', item.id, error);
      
      // Re-queue failed items (with retry limit)
      if (!item.retries) item.retries = 0;
      item.retries++;
      
      if (item.retries < 3) {
        queue.enqueue(item);
      }
    }
  }
  
  console.log('Processed ' + processed + ' items');
}
```

### Session Storage

```javascript
class SessionStorage {
  constructor(sessionId = null) {
    this.sessionId = sessionId || this.createSessionId();
    this.cache = CacheService.getUserCache();
    this.ttl = 3600; // 1 hour
  }
  
  createSessionId() {
    return 'session_' + Utilities.getUuid();
  }
  
  get(key) {
    var fullKey = this.sessionId + '_' + key;
    var value = this.cache.get(fullKey);
    
    try {
      return value ? JSON.parse(value) : null;
    } catch (e) {
      return value;
    }
  }
  
  set(key, value) {
    var fullKey = this.sessionId + '_' + key;
    var stringValue = typeof value === 'string' 
      ? value 
      : JSON.stringify(value);
    
    this.cache.put(fullKey, stringValue, this.ttl);
  }
  
  remove(key) {
    var fullKey = this.sessionId + '_' + key;
    this.cache.remove(fullKey);
  }
  
  clear() {
    // Note: Can't clear just session keys without tracking them
    console.warn('Session clear not fully implemented');
  }
  
  extend() {
    // Extend session by updating TTL
    // Would need to re-set all values
  }
}

// Multi-step form example
function handleFormStep1() {
  var session = new SessionStorage();
  
  session.set('formData', {
    step: 1,
    name: 'John Doe',
    email: 'john@example.com'
  });
  
  return session.sessionId;
}

function handleFormStep2(sessionId) {
  var session = new SessionStorage(sessionId);
  var formData = session.get('formData');
  
  formData.step = 2;
  formData.address = '123 Main St';
  
  session.set('formData', formData);
}
```

## Data Synchronization

### Cross-Execution State Sync

```javascript
class StateSync {
  constructor(syncId) {
    this.syncId = syncId;
    this.props = PropertiesService.getScriptProperties();
    this.cache = CacheService.getScriptCache();
  }
  
  sync(key, computeFunction) {
    // Try cache first
    var cacheKey = 'sync_' + this.syncId + '_' + key;
    var cached = this.cache.get(cacheKey);
    
    if (cached) {
      return JSON.parse(cached);
    }
    
    // Try properties
    var propKey = 'sync_' + this.syncId + '_' + key;
    var stored = this.props.getProperty(propKey);
    
    if (stored) {
      var data = JSON.parse(stored);
      
      // Check if expired
      if (data.expires && new Date().getTime() > data.expires) {
        // Recompute
        data = this.compute(key, computeFunction);
      }
      
      // Update cache
      this.cache.put(cacheKey, JSON.stringify(data.value), 300);
      return data.value;
    }
    
    // Compute and store
    var result = this.compute(key, computeFunction);
    return result.value;
  }
  
  compute(key, computeFunction) {
    var lock = LockService.getScriptLock();
    lock.waitLock(10000);
    
    try {
      // Double-check after acquiring lock
      var propKey = 'sync_' + this.syncId + '_' + key;
      var stored = this.props.getProperty(propKey);
      
      if (stored) {
        var data = JSON.parse(stored);
        if (!data.expires || new Date().getTime() <= data.expires) {
          return data;
        }
      }
      
      // Compute value
      var value = computeFunction();
      
      var data = {
        value: value,
        computed: new Date().getTime(),
        expires: new Date().getTime() + 3600000 // 1 hour
      };
      
      this.props.setProperty(propKey, JSON.stringify(data));
      
      return data;
      
    } finally {
      lock.releaseLock();
    }
  }
  
  invalidate(key) {
    var propKey = 'sync_' + this.syncId + '_' + key;
    var cacheKey = 'sync_' + this.syncId + '_' + key;
    
    this.props.deleteProperty(propKey);
    this.cache.remove(cacheKey);
  }
}
```

## Best Practices

1. **Use appropriate storage type**:
   - Properties: Configuration, state
   - Cache: Temporary data, expensive computations
   - Lock: Protecting critical sections

2. **Handle size limits**:
   - Properties: 9KB per value, 500KB total
   - Cache: 100KB per key, 10MB total

3. **Implement expiration** for cached data

4. **Use locks** for concurrent access to shared state

5. **Structure data** to fit within limits

6. **Monitor usage** to avoid quota issues

7. **Clean up** old data regularly

8. **Use compression** for large data if needed

9. **Implement fallbacks** when storage fails

10. **Document your state management** strategy