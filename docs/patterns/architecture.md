# Architecture Patterns for Google Apps Script

[← Back to Main Documentation](../README.md)

## Overview

Choosing the right architecture pattern is crucial for building maintainable and scalable Apps Script projects. This guide covers various architectural approaches and when to use them.

> **Note**: Before implementing any architecture pattern, familiarize yourself with [Google Apps Script's unique behaviors and limitations](../core-concepts/gas-nuances-gotchas.md) to avoid common architectural pitfalls.

## Script Types and Organization

### Standalone Scripts

Best for utilities, APIs, and independent tools.

```javascript
/**
 * Project Structure:
 * ├── Code.gs          (Main entry point)
 * ├── Config.gs        (Configuration and constants)
 * ├── Utils.gs         (Utility functions)
 * ├── Services/        (Service integrations)
 * │   ├── DriveService.gs
 * │   ├── SheetsService.gs
 * │   └── GmailService.gs
 * ├── Models/          (Data models)
 * │   ├── User.gs
 * │   └── Document.gs
 * └── Tests/           (Test files)
 *     ├── TestUtils.gs
 *     └── TestServices.gs
 */

// Main.gs - Entry point
function doGet(e) {
  return handleRequest(e);
}

function doPost(e) {
  return handleRequest(e);
}

function handleRequest(e) {
  var router = new Router();
  return router.route(e);
}

// Config.gs
var CONFIG = {
  VERSION: '1.0.0',
  API_BASE_URL: 'https://api.example.com',
  CACHE_TTL: 3600,
  MAX_RETRIES: 3
};

// Router pattern for web apps
class Router {
  constructor() {
    this.routes = new Map();
    this.setupRoutes();
  }
  
  setupRoutes() {
    this.routes.set('GET:/users', UserController.list);
    this.routes.set('GET:/users/:id', UserController.get);
    this.routes.set('POST:/users', UserController.create);
    this.routes.set('PUT:/users/:id', UserController.update);
    this.routes.set('DELETE:/users/:id', UserController.delete);
  }
  
  route(e) {
    var method = e.parameter.method || 'GET';
    var path = e.parameter.path || '/';
    
    var handler = this.routes.get(method + ':' + path);
    
    if (handler) {
      return handler(e);
    }
    
    return this.notFound();
  }
  
  notFound() {
    return ContentService
      .createTextOutput(JSON.stringify({error: 'Not found'}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

### Container-Bound Scripts

Best for extending Google Workspace documents.

```javascript
/**
 * Spreadsheet-bound script structure:
 * ├── Code.gs          (Main functionality)
 * ├── Menu.gs          (Custom menu setup)
 * ├── Triggers.gs      (Event handlers)
 * ├── Sidebar.gs       (UI components)
 * ├── DataAccess.gs    (Sheet operations)
 * └── Utilities.gs     (Helper functions)
 */

// Menu.gs
function onOpen(e) {
  var ui = SpreadsheetApp.getUi();
  
  ui.createMenu('Custom Tools')
    .addItem('Import Data', 'importData')
    .addItem('Export Report', 'exportReport')
    .addSeparator()
    .addSubMenu(ui.createMenu('Advanced')
      .addItem('Settings', 'showSettings')
      .addItem('About', 'showAbout'))
    .addToUi();
}

// DataAccess.gs - Data layer abstraction
class SheetDatabase {
  constructor(spreadsheet) {
    this.spreadsheet = spreadsheet || SpreadsheetApp.getActiveSpreadsheet();
    this.cache = {};
  }
  
  getTable(sheetName) {
    if (!this.cache[sheetName]) {
      this.cache[sheetName] = new SheetTable(
        this.spreadsheet.getSheetByName(sheetName)
      );
    }
    return this.cache[sheetName];
  }
  
  createTable(name, columns) {
    var sheet = this.spreadsheet.insertSheet(name);
    sheet.getRange(1, 1, 1, columns.length).setValues([columns]);
    return new SheetTable(sheet);
  }
}

class SheetTable {
  constructor(sheet) {
    this.sheet = sheet;
    this.headers = this.loadHeaders();
  }
  
  loadHeaders() {
    if (this.sheet.getLastRow() === 0) return [];
    return this.sheet.getRange(1, 1, 1, this.sheet.getLastColumn())
      .getValues()[0];
  }
  
  insert(record) {
    var row = this.headers.map(header => record[header] || '');
    this.sheet.appendRow(row);
  }
  
  find(criteria) {
    var data = this.sheet.getDataRange().getValues();
    var results = [];
    
    for (var i = 1; i < data.length; i++) {
      var record = this.rowToObject(data[i]);
      
      if (this.matchesCriteria(record, criteria)) {
        results.push(record);
      }
    }
    
    return results;
  }
  
  rowToObject(row) {
    var obj = {};
    this.headers.forEach((header, index) => {
      obj[header] = row[index];
    });
    return obj;
  }
  
  matchesCriteria(record, criteria) {
    return Object.keys(criteria).every(key => 
      record[key] === criteria[key]
    );
  }
}
```

### Library Scripts

Reusable code shared across projects.

```javascript
/**
 * Library structure:
 * ├── Library.gs       (Public API)
 * ├── Internal.gs      (Private implementations)
 * └── Constants.gs     (Shared constants)
 */

// Library.gs - Public API
/**
 * MyLibrary - Utility library for Apps Script
 * Version: 1.0.0
 * 
 * Usage:
 * 1. Add library: Script ID: [YOUR_SCRIPT_ID]
 * 2. Use identifier: MyLibrary
 * 3. Call methods: MyLibrary.methodName()
 */

// Public methods
function formatDate(date, format) {
  return Internal_.formatDateInternal(date, format);
}

function parseCSV(csvString, options) {
  return Internal_.parseCSVInternal(csvString, options);
}

function retry(func, options) {
  return Internal_.retryInternal(func, options);
}

// Version info
function getVersion() {
  return '1.0.0';
}

// Internal.gs - Private implementations
var Internal_ = {
  formatDateInternal: function(date, format) {
    // Implementation hidden from library users
    if (!date) date = new Date();
    
    var timezone = Session.getScriptTimeZone();
    return Utilities.formatDate(date, timezone, format);
  },
  
  parseCSVInternal: function(csvString, options) {
    var delimiter = options?.delimiter || ',';
    var hasHeaders = options?.hasHeaders !== false;
    
    var rows = csvString.split('\n').map(row => 
      row.split(delimiter).map(cell => cell.trim())
    );
    
    if (hasHeaders) {
      var headers = rows[0];
      return rows.slice(1).map(row => {
        var obj = {};
        headers.forEach((header, index) => {
          obj[header] = row[index];
        });
        return obj;
      });
    }
    
    return rows;
  },
  
  retryInternal: function(func, options) {
    var maxAttempts = options?.maxAttempts || 3;
    var delay = options?.delay || 1000;
    
    for (var i = 0; i < maxAttempts; i++) {
      try {
        return func();
      } catch (error) {
        if (i === maxAttempts - 1) throw error;
        Utilities.sleep(delay * Math.pow(2, i));
      }
    }
  }
};
```

## Design Patterns

### Model-View-Controller (MVC)

```javascript
// Model - Data and business logic
class TaskModel {
  constructor() {
    this.sheet = SpreadsheetApp.getActiveSpreadsheet()
      .getSheetByName('Tasks') || this.createTaskSheet();
  }
  
  createTaskSheet() {
    var sheet = SpreadsheetApp.getActiveSpreadsheet()
      .insertSheet('Tasks');
    
    sheet.getRange(1, 1, 1, 5).setValues([[
      'ID', 'Title', 'Status', 'Created', 'Updated'
    ]]);
    
    return sheet;
  }
  
  getAllTasks() {
    if (this.sheet.getLastRow() <= 1) return [];
    
    var data = this.sheet.getRange(2, 1, 
      this.sheet.getLastRow() - 1, 5).getValues();
    
    return data.map(row => ({
      id: row[0],
      title: row[1],
      status: row[2],
      created: row[3],
      updated: row[4]
    }));
  }
  
  createTask(title) {
    var id = Utilities.getUuid();
    var now = new Date();
    
    this.sheet.appendRow([id, title, 'pending', now, now]);
    
    return {id, title, status: 'pending', created: now, updated: now};
  }
  
  updateTask(id, updates) {
    var data = this.sheet.getDataRange().getValues();
    
    for (var i = 1; i < data.length; i++) {
      if (data[i][0] === id) {
        if (updates.title) data[i][1] = updates.title;
        if (updates.status) data[i][2] = updates.status;
        data[i][4] = new Date(); // Updated timestamp
        
        this.sheet.getRange(i + 1, 1, 1, 5).setValues([data[i]]);
        return true;
      }
    }
    
    return false;
  }
}

// View - UI rendering
class TaskView {
  static renderTaskList(tasks) {
    var html = '<div class="task-list">';
    
    tasks.forEach(task => {
      html += `
        <div class="task-item" data-id="${task.id}">
          <h3>${task.title}</h3>
          <span class="status ${task.status}">${task.status}</span>
          <button onclick="editTask('${task.id}')">Edit</button>
          <button onclick="deleteTask('${task.id}')">Delete</button>
        </div>
      `;
    });
    
    html += '</div>';
    return html;
  }
  
  static renderTaskForm(task = null) {
    return `
      <form id="task-form">
        <input type="hidden" name="id" value="${task?.id || ''}">
        <label>
          Title:
          <input type="text" name="title" value="${task?.title || ''}" required>
        </label>
        <label>
          Status:
          <select name="status">
            <option value="pending" ${task?.status === 'pending' ? 'selected' : ''}>Pending</option>
            <option value="completed" ${task?.status === 'completed' ? 'selected' : ''}>Completed</option>
          </select>
        </label>
        <button type="submit">${task ? 'Update' : 'Create'}</button>
      </form>
    `;
  }
}

// Controller - Handles user actions
class TaskController {
  constructor() {
    this.model = new TaskModel();
  }
  
  showTaskList() {
    var tasks = this.model.getAllTasks();
    var html = TaskView.renderTaskList(tasks);
    
    var htmlOutput = HtmlService.createHtmlOutput(html)
      .setTitle('Task List')
      .setWidth(400);
    
    SpreadsheetApp.getUi().showSidebar(htmlOutput);
  }
  
  createTask(formData) {
    var task = this.model.createTask(formData.title);
    return {success: true, task: task};
  }
  
  updateTask(formData) {
    var success = this.model.updateTask(formData.id, {
      title: formData.title,
      status: formData.status
    });
    
    return {success: success};
  }
}
```

### Repository Pattern

```javascript
// Generic repository
class Repository {
  constructor(sheet) {
    this.sheet = sheet;
    this.cache = new Map();
  }
  
  find(id) {
    if (this.cache.has(id)) {
      return this.cache.get(id);
    }
    
    var data = this.sheet.getDataRange().getValues();
    
    for (var i = 1; i < data.length; i++) {
      if (data[i][0] === id) {
        var entity = this.mapRowToEntity(data[i], i + 1);
        this.cache.set(id, entity);
        return entity;
      }
    }
    
    return null;
  }
  
  findAll(criteria = {}) {
    var data = this.sheet.getDataRange().getValues();
    var headers = data[0];
    var results = [];
    
    for (var i = 1; i < data.length; i++) {
      var entity = this.mapRowToEntity(data[i], i + 1);
      
      if (this.matchesCriteria(entity, criteria)) {
        results.push(entity);
      }
    }
    
    return results;
  }
  
  save(entity) {
    if (entity._rowIndex) {
      // Update existing
      this.updateRow(entity);
    } else {
      // Insert new
      this.insertRow(entity);
    }
    
    this.cache.set(entity.id, entity);
    return entity;
  }
  
  delete(id) {
    var entity = this.find(id);
    
    if (entity && entity._rowIndex) {
      this.sheet.deleteRow(entity._rowIndex);
      this.cache.delete(id);
      return true;
    }
    
    return false;
  }
  
  mapRowToEntity(row, rowIndex) {
    // Override in subclasses
    throw new Error('mapRowToEntity must be implemented');
  }
  
  mapEntityToRow(entity) {
    // Override in subclasses
    throw new Error('mapEntityToRow must be implemented');
  }
  
  matchesCriteria(entity, criteria) {
    return Object.keys(criteria).every(key => 
      entity[key] === criteria[key]
    );
  }
  
  updateRow(entity) {
    var row = this.mapEntityToRow(entity);
    this.sheet.getRange(entity._rowIndex, 1, 1, row.length)
      .setValues([row]);
  }
  
  insertRow(entity) {
    entity.id = entity.id || Utilities.getUuid();
    var row = this.mapEntityToRow(entity);
    this.sheet.appendRow(row);
    entity._rowIndex = this.sheet.getLastRow();
  }
}

// Specific repository implementation
class UserRepository extends Repository {
  constructor() {
    var sheet = SpreadsheetApp.getActiveSpreadsheet()
      .getSheetByName('Users') || this.createSheet();
    
    super(sheet);
  }
  
  createSheet() {
    var sheet = SpreadsheetApp.getActiveSpreadsheet()
      .insertSheet('Users');
    
    sheet.getRange(1, 1, 1, 4).setValues([[
      'ID', 'Name', 'Email', 'Created'
    ]]);
    
    return sheet;
  }
  
  mapRowToEntity(row, rowIndex) {
    return {
      id: row[0],
      name: row[1],
      email: row[2],
      created: row[3],
      _rowIndex: rowIndex
    };
  }
  
  mapEntityToRow(entity) {
    return [
      entity.id,
      entity.name,
      entity.email,
      entity.created || new Date()
    ];
  }
  
  findByEmail(email) {
    return this.findAll({email: email})[0] || null;
  }
}
```

### Service Layer Pattern

```javascript
// Service layer for business logic
class UserService {
  constructor() {
    this.userRepo = new UserRepository();
    this.emailService = new EmailService();
    this.auditService = new AuditService();
  }
  
  createUser(userData) {
    // Validate
    this.validateUserData(userData);
    
    // Check if exists
    var existing = this.userRepo.findByEmail(userData.email);
    if (existing) {
      throw new Error('User with this email already exists');
    }
    
    // Create user
    var user = {
      id: Utilities.getUuid(),
      name: userData.name,
      email: userData.email,
      created: new Date()
    };
    
    // Save
    user = this.userRepo.save(user);
    
    // Send welcome email
    this.emailService.sendWelcomeEmail(user);
    
    // Audit log
    this.auditService.log('user_created', user);
    
    return user;
  }
  
  validateUserData(data) {
    if (!data.name || data.name.length < 2) {
      throw new Error('Name must be at least 2 characters');
    }
    
    if (!data.email || !this.isValidEmail(data.email)) {
      throw new Error('Valid email is required');
    }
  }
  
  isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
}

// Dependency injection container
class ServiceContainer {
  constructor() {
    this.services = new Map();
    this.singletons = new Map();
  }
  
  register(name, factory, options = {}) {
    this.services.set(name, {
      factory: factory,
      singleton: options.singleton || false
    });
  }
  
  get(name) {
    var service = this.services.get(name);
    
    if (!service) {
      throw new Error(`Service ${name} not found`);
    }
    
    if (service.singleton) {
      if (!this.singletons.has(name)) {
        this.singletons.set(name, service.factory(this));
      }
      return this.singletons.get(name);
    }
    
    return service.factory(this);
  }
}

// Setup
var container = new ServiceContainer();

container.register('userRepository', () => new UserRepository(), {singleton: true});
container.register('emailService', () => new EmailService(), {singleton: true});
container.register('auditService', () => new AuditService(), {singleton: true});
container.register('userService', (container) => new UserService(
  container.get('userRepository'),
  container.get('emailService'),
  container.get('auditService')
));
```

### Event-Driven Architecture

```javascript
// Event emitter
class EventEmitter {
  constructor() {
    this.events = new Map();
  }
  
  on(event, handler) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    
    this.events.get(event).push(handler);
  }
  
  emit(event, data) {
    if (!this.events.has(event)) return;
    
    var handlers = this.events.get(event);
    handlers.forEach(handler => {
      try {
        handler(data);
      } catch (error) {
        console.error(`Error in event handler for ${event}:`, error);
      }
    });
  }
  
  off(event, handler) {
    if (!this.events.has(event)) return;
    
    var handlers = this.events.get(event);
    var index = handlers.indexOf(handler);
    
    if (index > -1) {
      handlers.splice(index, 1);
    }
  }
}

// Application events
class AppEvents extends EventEmitter {
  static SPREADSHEET_UPDATED = 'spreadsheet:updated';
  static USER_CREATED = 'user:created';
  static DATA_IMPORTED = 'data:imported';
  static ERROR_OCCURRED = 'error:occurred';
}

// Event-driven application
class Application {
  constructor() {
    this.events = new AppEvents();
    this.setupEventHandlers();
  }
  
  setupEventHandlers() {
    // Log all events
    this.events.on(AppEvents.SPREADSHEET_UPDATED, (data) => {
      console.log('Spreadsheet updated:', data);
    });
    
    // Send notifications
    this.events.on(AppEvents.USER_CREATED, (user) => {
      NotificationService.notifyAdmins('New user: ' + user.email);
    });
    
    // Update cache
    this.events.on(AppEvents.DATA_IMPORTED, (data) => {
      CacheService.getScriptCache().remove('data_summary');
    });
    
    // Error handling
    this.events.on(AppEvents.ERROR_OCCURRED, (error) => {
      ErrorLogger.log(error);
    });
  }
  
  importData(source) {
    try {
      var data = DataImporter.import(source);
      
      this.events.emit(AppEvents.DATA_IMPORTED, {
        source: source,
        recordCount: data.length
      });
      
      return data;
      
    } catch (error) {
      this.events.emit(AppEvents.ERROR_OCCURRED, error);
      throw error;
    }
  }
}
```

## Add-on Architecture

### Add-on Structure

```javascript
/**
 * Add-on project structure:
 * ├── Code.gs          (Main add-on code)
 * ├── Config.gs        (Configuration)
 * ├── Auth.gs          (Authorization handling)
 * ├── UI.gs            (User interface)
 * ├── Cards.gs         (Card-based UI for Gmail/Calendar)
 * └── Manifest.gs      (Add-on manifest)
 */

// Code.gs - Main entry points
function onInstall(e) {
  onOpen(e);
}

function onOpen(e) {
  var menu = SpreadsheetApp.getUi().createAddonMenu();
  
  if (e && e.authMode == ScriptApp.AuthMode.NONE) {
    // Add-on is installed but not enabled
    menu.addItem('Start', 'showAuthorizationDialog');
  } else {
    // Add-on is enabled
    menu.addItem('Open Sidebar', 'showSidebar')
        .addItem('Settings', 'showSettings')
        .addSeparator()
        .addItem('Help', 'showHelp');
  }
  
  menu.addToUi();
}

// Auth.gs - Authorization handling
function getAuthorizationUrl() {
  return ScriptApp.getAuthorizationInfo(ScriptApp.AuthMode.FULL)
    .getAuthorizationUrl();
}

function showAuthorizationDialog() {
  var authInfo = ScriptApp.getAuthorizationInfo(ScriptApp.AuthMode.FULL);
  
  if (authInfo.getAuthorizationStatus() == 
      ScriptApp.AuthorizationStatus.NOT_REQUIRED) {
    // Already authorized
    showSidebar();
  } else {
    // Show authorization prompt
    var template = HtmlService.createTemplate(
      '<p>Please authorize this add-on to continue.</p>' +
      '<p><a href="<?= authUrl ?>" target="_blank">Authorize</a></p>'
    );
    
    template.authUrl = authInfo.getAuthorizationUrl();
    
    var html = template.evaluate()
      .setWidth(300)
      .setHeight(200);
    
    SpreadsheetApp.getUi().showModalDialog(html, 'Authorization Required');
  }
}

// Configuration management
class AddOnConfig {
  static get(key, defaultValue = null) {
    var userProps = PropertiesService.getUserProperties();
    var value = userProps.getProperty('addon_' + key);
    
    if (value === null) return defaultValue;
    
    try {
      return JSON.parse(value);
    } catch (e) {
      return value;
    }
  }
  
  static set(key, value) {
    var userProps = PropertiesService.getUserProperties();
    var stringValue = typeof value === 'string' 
      ? value 
      : JSON.stringify(value);
    
    userProps.setProperty('addon_' + key, stringValue);
  }
  
  static getAll() {
    var userProps = PropertiesService.getUserProperties();
    var props = userProps.getProperties();
    var config = {};
    
    Object.keys(props).forEach(key => {
      if (key.startsWith('addon_')) {
        var configKey = key.substring(6);
        try {
          config[configKey] = JSON.parse(props[key]);
        } catch (e) {
          config[configKey] = props[key];
        }
      }
    });
    
    return config;
  }
}
```

## Best Practices

### 1. Separation of Concerns

```javascript
// Bad - Mixed concerns
function processData() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var data = sheet.getDataRange().getValues();
  
  var html = '<table>';
  
  for (var i = 0; i < data.length; i++) {
    // Business logic mixed with UI
    if (data[i][2] > 100) {
      html += '<tr class="highlight">';
    } else {
      html += '<tr>';
    }
    
    // Data access mixed with presentation
    html += '<td>' + data[i][0] + '</td>';
    html += '</tr>';
  }
  
  html += '</table>';
  
  // Direct UI manipulation
  var ui = HtmlService.createHtmlOutput(html);
  SpreadsheetApp.getUi().showSidebar(ui);
}

// Good - Separated concerns
class DataService {
  static getData() {
    var sheet = SpreadsheetApp.getActiveSheet();
    return sheet.getDataRange().getValues();
  }
  
  static getHighValueItems(threshold = 100) {
    var data = this.getData();
    return data.filter(row => row[2] > threshold);
  }
}

class DataPresenter {
  static formatAsTable(data, highlightCondition) {
    return data.map(row => ({
      values: row,
      highlight: highlightCondition(row)
    }));
  }
}

class UIController {
  static showDataTable() {
    var data = DataService.getData();
    var formatted = DataPresenter.formatAsTable(
      data,
      row => row[2] > 100
    );
    
    var html = UIBuilder.buildTable(formatted);
    UIService.showSidebar(html);
  }
}
```

### 2. Configuration Management

```javascript
// Centralized configuration
class AppConfig {
  static ENV = {
    DEVELOPMENT: 'development',
    STAGING: 'staging',
    PRODUCTION: 'production'
  };
  
  static getCurrentEnv() {
    var scriptProps = PropertiesService.getScriptProperties();
    return scriptProps.getProperty('ENVIRONMENT') || this.ENV.DEVELOPMENT;
  }
  
  static get(key) {
    var env = this.getCurrentEnv();
    var config = this.CONFIGS[env];
    
    return key.split('.').reduce((obj, k) => obj?.[k], config);
  }
  
  static CONFIGS = {
    development: {
      api: {
        baseUrl: 'http://localhost:3000',
        timeout: 5000
      },
      cache: {
        ttl: 300
      },
      debug: true
    },
    
    production: {
      api: {
        baseUrl: 'https://api.example.com',
        timeout: 30000
      },
      cache: {
        ttl: 3600
      },
      debug: false
    }
  };
}

// Usage
var apiUrl = AppConfig.get('api.baseUrl');
var isDebug = AppConfig.get('debug');
```

### 3. Testing Strategy

```javascript
// Test framework
class TestRunner {
  constructor() {
    this.tests = [];
    this.results = [];
  }
  
  describe(description, testFunc) {
    this.currentSuite = description;
    testFunc();
  }
  
  it(description, testFunc) {
    this.tests.push({
      suite: this.currentSuite,
      description: description,
      func: testFunc
    });
  }
  
  expect(actual) {
    return {
      toBe: (expected) => {
        if (actual !== expected) {
          throw new Error(`Expected ${actual} to be ${expected}`);
        }
      },
      
      toEqual: (expected) => {
        if (JSON.stringify(actual) !== JSON.stringify(expected)) {
          throw new Error(`Expected ${actual} to equal ${expected}`);
        }
      },
      
      toThrow: () => {
        try {
          actual();
          throw new Error('Expected function to throw');
        } catch (e) {
          // Expected
        }
      }
    };
  }
  
  run() {
    this.tests.forEach(test => {
      try {
        test.func();
        this.results.push({
          ...test,
          passed: true
        });
      } catch (error) {
        this.results.push({
          ...test,
          passed: false,
          error: error.message
        });
      }
    });
    
    this.report();
  }
  
  report() {
    var passed = this.results.filter(r => r.passed).length;
    var failed = this.results.filter(r => !r.passed).length;
    
    console.log(`Tests: ${passed} passed, ${failed} failed`);
    
    this.results.filter(r => !r.passed).forEach(result => {
      console.error(`FAIL: ${result.suite} - ${result.description}`);
      console.error(`  ${result.error}`);
    });
  }
}

// Test examples
function runTests() {
  var runner = new TestRunner();
  
  runner.describe('UserService', function() {
    runner.it('should create a user', function() {
      var user = UserService.create({
        name: 'Test User',
        email: 'test@example.com'
      });
      
      runner.expect(user.name).toBe('Test User');
      runner.expect(user.id).toBeTruthy();
    });
    
    runner.it('should validate email', function() {
      runner.expect(() => {
        UserService.create({
          name: 'Test',
          email: 'invalid-email'
        });
      }).toThrow();
    });
  });
  
  runner.run();
}
```

## Summary

Key architectural principles:

1. **Choose the right script type** for your use case
2. **Organize code logically** with clear separation
3. **Use design patterns** appropriately
4. **Abstract data access** from business logic
5. **Implement proper error handling** at all layers
6. **Make code testable** with dependency injection
7. **Use configuration** for environment-specific values
8. **Document your architecture** decisions
9. **Plan for scalability** from the start
10. **Keep it simple** - don't over-engineer