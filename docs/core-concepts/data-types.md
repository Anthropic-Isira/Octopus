# Data Types in Google Apps Script

[← Back to Main Documentation](../README.md)

## Overview

Google Apps Script is based on JavaScript, but working with Google services introduces unique data type challenges. This guide covers type handling, conversions, and common pitfalls.

## JavaScript Types in Apps Script

### Basic Types

```javascript
// Apps Script supports standard JavaScript types
var string = "Hello World";
var number = 42;
var boolean = true;
var nullValue = null;
var undefinedValue = undefined;
var object = {key: "value"};
var array = [1, 2, 3];
var func = function() { return "function"; };

// Type checking
console.log(typeof string);      // "string"
console.log(typeof number);      // "number"
console.log(typeof boolean);     // "boolean"
console.log(typeof nullValue);   // "object" (JavaScript quirk)
console.log(typeof undefinedValue); // "undefined"
console.log(Array.isArray(array)); // true
```

## Date and Time Handling

### Working with Dates

```javascript
// Creating dates
var now = new Date();
var specific = new Date('2024-12-25');
var fromParts = new Date(2024, 11, 25); // Month is 0-indexed!

// Common date operations
function dateOperations() {
  var date = new Date();
  
  // Get components
  var year = date.getFullYear();
  var month = date.getMonth(); // 0-11
  var day = date.getDate();    // 1-31
  var hours = date.getHours();  // 0-23
  var minutes = date.getMinutes();
  
  // Set components
  date.setHours(0, 0, 0, 0); // Start of day
  
  // Add days
  date.setDate(date.getDate() + 7); // Add 7 days
  
  // Format date
  var formatted = Utilities.formatDate(date, 'GMT', 'yyyy-MM-dd HH:mm:ss');
  
  return formatted;
}
```

### Time Zone Handling

```javascript
// Time zones are critical in Apps Script
function timeZoneHandling() {
  var date = new Date();
  
  // Get script time zone
  var scriptTimeZone = Session.getScriptTimeZone();
  console.log('Script timezone:', scriptTimeZone);
  
  // Format in specific time zone
  var nyTime = Utilities.formatDate(date, 'America/New_York', 'yyyy-MM-dd HH:mm:ss z');
  var utcTime = Utilities.formatDate(date, 'UTC', 'yyyy-MM-dd HH:mm:ss z');
  
  console.log('NY Time:', nyTime);
  console.log('UTC Time:', utcTime);
  
  // Convert between time zones
  var localDate = new Date();
  var utcDate = new Date(localDate.getTime() + localDate.getTimezoneOffset() * 60000);
  
  return {
    local: localDate,
    utc: utcDate
  };
}

// Calendar dates vs JavaScript dates
function calendarDateHandling() {
  // Calendar API returns dates in ISO format with timezone
  var event = CalendarApp.getDefaultCalendar()
    .getEvents(new Date(), new Date())[0];
  
  if (event) {
    var startTime = event.getStartTime(); // JavaScript Date object
    var endTime = event.getEndTime();
    
    // All-day events
    if (event.isAllDayEvent()) {
      // Dates are at midnight in calendar's timezone
      console.log('All-day event date:', startTime);
    }
  }
}
```

### Date Comparison and Math

```javascript
function dateMath() {
  var start = new Date('2024-01-01');
  var end = new Date('2024-12-31');
  
  // Compare dates
  if (end > start) {
    console.log('End is after start');
  }
  
  // Calculate difference
  var diffMs = end - start;
  var diffDays = diffMs / (1000 * 60 * 60 * 24);
  console.log('Difference in days:', diffDays);
  
  // Check if date is between
  function isDateBetween(date, start, end) {
    return date >= start && date <= end;
  }
  
  // Business days calculation
  function addBusinessDays(startDate, daysToAdd) {
    var date = new Date(startDate);
    var added = 0;
    
    while (added < daysToAdd) {
      date.setDate(date.getDate() + 1);
      
      // Skip weekends
      if (date.getDay() !== 0 && date.getDay() !== 6) {
        added++;
      }
    }
    
    return date;
  }
  
  return {
    daysBetween: diffDays,
    businessDaysFromNow: addBusinessDays(new Date(), 10)
  };
}
```

## Spreadsheet Data Types

### Range Values

```javascript
// Spreadsheet values have special behaviors
function spreadsheetDataTypes() {
  var sheet = SpreadsheetApp.getActiveSheet();
  
  // Getting values always returns 2D array
  var singleCell = sheet.getRange('A1').getValue();        // Single value
  var singleCellArray = sheet.getRange('A1').getValues(); // [[value]]
  var rangeValues = sheet.getRange('A1:C3').getValues();  // 2D array
  
  // Type conversions in sheets
  var data = [
    ['String', 'Number', 'Date', 'Boolean', 'Formula'],
    ['Hello', 123, new Date(), true, '=A2&B2'],
    ['World', 456.78, '2024-12-25', 'TRUE', '=SUM(B2:B3)']
  ];
  
  sheet.getRange(1, 1, data.length, data[0].length).setValues(data);
  
  // Reading back - types may change!
  var retrieved = sheet.getRange(1, 1, 3, 5).getValues();
  
  retrieved.forEach(function(row, i) {
    row.forEach(function(cell, j) {
      console.log('Cell [' + i + ',' + j + '] type:', typeof cell, 'value:', cell);
    });
  });
}

// Handle empty cells and errors
function handleSpreadsheetSpecialValues() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var values = sheet.getRange('A1:A10').getValues();
  
  values.forEach(function(row, i) {
    var value = row[0];
    
    // Empty cells return empty string
    if (value === '') {
      console.log('Row ' + (i+1) + ' is empty');
    }
    
    // Error values
    if (value instanceof Error || value === '#ERROR!') {
      console.log('Row ' + (i+1) + ' has an error');
    }
    
    // Check for specific error types
    if (typeof value === 'string' && value.startsWith('#')) {
      console.log('Row ' + (i+1) + ' has error:', value);
    }
  });
}
```

### Number Formatting

```javascript
function numberHandling() {
  var sheet = SpreadsheetApp.getActiveSheet();
  
  // Numbers in JavaScript vs Sheets
  var jsNumber = 123.456789;
  var range = sheet.getRange('A1');
  
  // Set value
  range.setValue(jsNumber);
  
  // Apply number format
  range.setNumberFormat('#,##0.00'); // Shows as 123.46
  
  // Get value - returns full precision
  var retrieved = range.getValue(); // 123.456789
  
  // Get display value
  var displayed = range.getDisplayValue(); // "123.46"
  
  // Currency handling
  var currency = 1234.56;
  sheet.getRange('B1').setValue(currency).setNumberFormat('$#,##0.00');
  
  // Percentage
  var percent = 0.1534;
  sheet.getRange('C1').setValue(percent).setNumberFormat('0.00%'); // Shows as 15.34%
  
  // Scientific notation
  var scientific = 1234567890;
  sheet.getRange('D1').setValue(scientific).setNumberFormat('0.00E+00');
  
  return {
    original: jsNumber,
    retrieved: retrieved,
    displayed: displayed
  };
}

// Precision issues
function precisionIssues() {
  // JavaScript floating point issues
  console.log(0.1 + 0.2); // 0.30000000000000004
  
  // Rounding solutions
  function round(value, decimals) {
    return Number(Math.round(value + 'e' + decimals) + 'e-' + decimals);
  }
  
  console.log(round(0.1 + 0.2, 2)); // 0.3
  
  // Money calculations
  function moneyCalc(amount1, amount2) {
    // Convert to cents for calculation
    var cents1 = Math.round(amount1 * 100);
    var cents2 = Math.round(amount2 * 100);
    var result = (cents1 + cents2) / 100;
    return result;
  }
  
  console.log(moneyCalc(10.10, 20.20)); // 30.3
}
```

## Blob and File Handling

### Working with Blobs

```javascript
// Blobs represent binary data
function blobHandling() {
  // Create blob from string
  var textBlob = Utilities.newBlob('Hello World', 'text/plain', 'hello.txt');
  
  // Create blob from base64
  var base64 = 'SGVsbG8gV29ybGQ='; // "Hello World" in base64
  var base64Blob = Utilities.newBlob(
    Utilities.base64Decode(base64),
    'text/plain',
    'decoded.txt'
  );
  
  // Get blob from file
  var file = DriveApp.getFileById('FILE_ID');
  var fileBlob = file.getBlob();
  
  // Blob operations
  console.log('Name:', textBlob.getName());
  console.log('Type:', textBlob.getContentType());
  console.log('Size:', textBlob.getBytes().length);
  
  // Convert blob to string
  var content = textBlob.getDataAsString();
  console.log('Content:', content);
  
  // Convert blob to base64
  var encoded = Utilities.base64Encode(textBlob.getBytes());
  console.log('Base64:', encoded);
  
  return textBlob;
}

// Image blob handling
function imageHandling() {
  // Get image from URL
  var imageUrl = 'https://example.com/image.png';
  var response = UrlFetchApp.fetch(imageUrl);
  var imageBlob = response.getBlob();
  
  // Set proper content type
  imageBlob.setContentType('image/png');
  imageBlob.setName('downloaded-image.png');
  
  // Resize image (requires third-party service)
  // var resizedBlob = resizeImage(imageBlob, 200, 200);
  
  // Save to Drive
  var file = DriveApp.createFile(imageBlob);
  
  // Insert into document
  var doc = DocumentApp.create('Image Test');
  doc.getBody().appendImage(imageBlob);
  
  // Insert into spreadsheet
  var sheet = SpreadsheetApp.getActiveSheet();
  sheet.insertImage(imageBlob, 1, 1);
  
  return file;
}
```

### File Type Conversions

```javascript
function fileConversions() {
  // Convert between Google file types
  var doc = DocumentApp.create('Test Document');
  var docFile = DriveApp.getFileById(doc.getId());
  
  // Export as PDF
  var pdfBlob = docFile.getAs('application/pdf');
  pdfBlob.setName(doc.getName() + '.pdf');
  var pdfFile = DriveApp.createFile(pdfBlob);
  
  // Export as Word
  var docxBlob = docFile.getAs('application/vnd.openxmlformats-officedocument.wordprocessingml.document');
  docxBlob.setName(doc.getName() + '.docx');
  
  // Convert spreadsheet to CSV
  var spreadsheet = SpreadsheetApp.create('Test Spreadsheet');
  var ssFile = DriveApp.getFileById(spreadsheet.getId());
  var csvBlob = ssFile.getAs('text/csv');
  
  // Handle encoding
  var csvContent = csvBlob.getDataAsString('UTF-8');
  
  return {
    pdf: pdfFile.getId(),
    csvContent: csvContent
  };
}
```

## Array and Object Handling

### Array Operations

```javascript
function arrayOperations() {
  // 2D arrays from spreadsheets
  var sheet = SpreadsheetApp.getActiveSheet();
  var data = sheet.getDataRange().getValues();
  
  // Transpose array
  function transpose(array) {
    return array[0].map((_, colIndex) => 
      array.map(row => row[colIndex])
    );
  }
  
  // Flatten 2D array
  function flatten(array) {
    return array.reduce((flat, row) => flat.concat(row), []);
  }
  
  // Filter empty rows
  function removeEmptyRows(data) {
    return data.filter(row => 
      row.some(cell => cell !== '')
    );
  }
  
  // Map with index
  var numbered = data.map((row, index) => {
    return [index + 1].concat(row);
  });
  
  // Sort by column
  function sortByColumn(data, columnIndex) {
    return data.sort((a, b) => {
      if (a[columnIndex] < b[columnIndex]) return -1;
      if (a[columnIndex] > b[columnIndex]) return 1;
      return 0;
    });
  }
  
  return {
    transposed: transpose(data),
    flattened: flatten(data),
    cleaned: removeEmptyRows(data)
  };
}

// Object manipulation
function objectOperations() {
  // Convert sheet data to objects
  function sheetToObjects(data) {
    var headers = data[0];
    var objects = [];
    
    for (var i = 1; i < data.length; i++) {
      var obj = {};
      for (var j = 0; j < headers.length; j++) {
        obj[headers[j]] = data[i][j];
      }
      objects.push(obj);
    }
    
    return objects;
  }
  
  // Convert objects back to sheet data
  function objectsToSheet(objects) {
    if (objects.length === 0) return [];
    
    var headers = Object.keys(objects[0]);
    var data = [headers];
    
    objects.forEach(function(obj) {
      var row = headers.map(header => obj[header] || '');
      data.push(row);
    });
    
    return data;
  }
  
  // Deep clone objects
  function deepClone(obj) {
    return JSON.parse(JSON.stringify(obj));
  }
  
  // Merge objects
  function mergeObjects(...objects) {
    return Object.assign({}, ...objects);
  }
  
  return {
    sheetToObjects: sheetToObjects,
    objectsToSheet: objectsToSheet
  };
}
```

## String Handling and Encoding

### String Operations

```javascript
function stringHandling() {
  // String encoding issues
  var text = "Hello 世界 🌍";
  
  // UTF-8 encoding
  var bytes = Utilities.newBlob(text).getBytes();
  var encoded = Utilities.base64Encode(bytes);
  var decoded = Utilities.newBlob(
    Utilities.base64Decode(encoded)
  ).getDataAsString();
  
  console.log('Original:', text);
  console.log('Encoded:', encoded);
  console.log('Decoded:', decoded);
  
  // URL encoding
  var urlParam = "Hello World & Special Chars!";
  var urlEncoded = encodeURIComponent(urlParam);
  var urlDecoded = decodeURIComponent(urlEncoded);
  
  // HTML encoding
  function htmlEncode(str) {
    return str.replace(/&/g, '&amp;')
              .replace(/</g, '&lt;')
              .replace(/>/g, '&gt;')
              .replace(/"/g, '&quot;')
              .replace(/'/g, '&#39;');
  }
  
  // Clean strings from sheets
  function cleanString(str) {
    if (typeof str !== 'string') return str;
    
    return str
      .trim()                      // Remove whitespace
      .replace(/\s+/g, ' ')       // Normalize spaces
      .replace(/[\u200B-\u200D\uFEFF]/g, ''); // Remove zero-width chars
  }
  
  return {
    original: text,
    cleaned: cleanString("  Hello   World  ")
  };
}

// Regular expressions
function regexOperations() {
  var text = "Contact: john@example.com or (555) 123-4567";
  
  // Extract emails
  var emailRegex = /[\w._%+-]+@[\w.-]+\.[A-Za-z]{2,}/g;
  var emails = text.match(emailRegex) || [];
  
  // Extract phone numbers
  var phoneRegex = /\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}/g;
  var phones = text.match(phoneRegex) || [];
  
  // Replace with capture groups
  var formatted = text.replace(
    /(\w+)@(\w+)\.(\w+)/g,
    '$1 at $2 dot $3'
  );
  
  // Test patterns
  function isValidEmail(email) {
    return /^[\w._%+-]+@[\w.-]+\.[A-Za-z]{2,}$/.test(email);
  }
  
  return {
    emails: emails,
    phones: phones,
    formatted: formatted
  };
}
```

## Type Conversion and Validation

### Safe Type Conversions

```javascript
function safeConversions() {
  // String to number
  function toNumber(value, defaultValue = 0) {
    var num = Number(value);
    return isNaN(num) ? defaultValue : num;
  }
  
  // Parse with validation
  function parseIntSafe(value, defaultValue = 0) {
    var parsed = parseInt(value, 10);
    return isNaN(parsed) ? defaultValue : parsed;
  }
  
  // Boolean conversion
  function toBoolean(value) {
    if (typeof value === 'boolean') return value;
    if (typeof value === 'string') {
      return ['true', 'yes', '1'].includes(value.toLowerCase());
    }
    return Boolean(value);
  }
  
  // Date parsing
  function parseDate(value) {
    if (value instanceof Date) return value;
    if (typeof value === 'string' || typeof value === 'number') {
      var date = new Date(value);
      return isNaN(date.getTime()) ? null : date;
    }
    return null;
  }
  
  // Array ensure
  function ensureArray(value) {
    if (Array.isArray(value)) return value;
    if (value == null) return [];
    return [value];
  }
  
  return {
    toNumber: toNumber,
    toBoolean: toBoolean,
    parseDate: parseDate,
    ensureArray: ensureArray
  };
}

// Type validation
function validation() {
  // Email validation
  function isValidEmail(email) {
    var re = /^[\w._%+-]+@[\w.-]+\.[A-Za-z]{2,}$/;
    return re.test(String(email).toLowerCase());
  }
  
  // URL validation
  function isValidUrl(url) {
    try {
      new URL(url);
      return true;
    } catch (e) {
      return false;
    }
  }
  
  // Range validation
  function isInRange(value, min, max) {
    var num = Number(value);
    return !isNaN(num) && num >= min && num <= max;
  }
  
  // Schema validation
  function validateObject(obj, schema) {
    var errors = [];
    
    for (var key in schema) {
      var rule = schema[key];
      var value = obj[key];
      
      if (rule.required && value == null) {
        errors.push(key + ' is required');
      }
      
      if (value != null && rule.type) {
        var actualType = Array.isArray(value) ? 'array' : typeof value;
        if (actualType !== rule.type) {
          errors.push(key + ' must be ' + rule.type);
        }
      }
      
      if (rule.validator && !rule.validator(value)) {
        errors.push(key + ' is invalid');
      }
    }
    
    return {
      valid: errors.length === 0,
      errors: errors
    };
  }
  
  // Example usage
  var schema = {
    email: {
      required: true,
      type: 'string',
      validator: isValidEmail
    },
    age: {
      required: true,
      type: 'number',
      validator: (v) => isInRange(v, 0, 150)
    }
  };
  
  var data = {email: 'test@example.com', age: 25};
  return validateObject(data, schema);
}
```

## JSON Handling

### JSON Operations

```javascript
function jsonHandling() {
  var obj = {
    name: 'John',
    age: 30,
    date: new Date(),
    func: function() {}, // Will be removed
    undef: undefined,    // Will be removed
    nullVal: null       // Will be kept
  };
  
  // Basic stringify
  var json = JSON.stringify(obj);
  console.log('Basic:', json);
  
  // Pretty print
  var pretty = JSON.stringify(obj, null, 2);
  console.log('Pretty:', pretty);
  
  // Custom replacer
  function replacer(key, value) {
    if (value instanceof Date) {
      return value.toISOString();
    }
    if (typeof value === 'number' && !isFinite(value)) {
      return String(value);
    }
    return value;
  }
  
  var customJson = JSON.stringify(obj, replacer, 2);
  
  // Safe parsing
  function parseJsonSafe(jsonString, defaultValue = null) {
    try {
      return JSON.parse(jsonString);
    } catch (e) {
      console.error('JSON parse error:', e);
      return defaultValue;
    }
  }
  
  // Parse with reviver
  function reviver(key, value) {
    if (typeof value === 'string' && /^\d{4}-\d{2}-\d{2}T/.test(value)) {
      return new Date(value);
    }
    return value;
  }
  
  var parsed = JSON.parse(customJson, reviver);
  
  return {
    original: obj,
    stringified: json,
    parsed: parsed
  };
}
```

## Common Type Pitfalls

### Null and Undefined

```javascript
function nullUndefinedHandling() {
  var sheet = SpreadsheetApp.getActiveSheet();
  
  // Spreadsheet cells never return null/undefined
  var emptyCell = sheet.getRange('A1').getValue(); // Returns ''
  
  // But formulas can produce errors
  sheet.getRange('A1').setFormula('=1/0'); // #DIV/0!
  var errorValue = sheet.getRange('A1').getValue(); // Error or string
  
  // Safe property access
  function safeGet(obj, path, defaultValue = null) {
    return path.split('.').reduce((current, key) => 
      current?.[key], obj) ?? defaultValue;
  }
  
  var data = {user: {name: 'John'}};
  console.log(safeGet(data, 'user.name')); // 'John'
  console.log(safeGet(data, 'user.email', 'N/A')); // 'N/A'
  
  // Null coalescing
  function getValue(value, defaultValue) {
    return value ?? defaultValue; // null or undefined
  }
}

// Type coercion surprises
function typeCoercion() {
  // String concatenation vs addition
  console.log('1' + 2);     // '12'
  console.log(1 + '2');     // '12'
  console.log(1 + 2 + '3'); // '33'
  
  // Boolean coercion
  console.log(!!'');        // false
  console.log(!!'0');       // true
  console.log(!!0);         // false
  console.log(!!{});        // true
  console.log(!![]);        // true
  
  // Equality checks
  console.log(0 == '');     // true
  console.log(0 === '');    // false
  console.log(null == undefined); // true
  console.log(null === undefined); // false
  
  // Always use === for comparisons!
}
```

## Best Practices

1. **Always validate external data** (from sheets, forms, APIs)
2. **Use strict equality** (===) instead of loose equality (==)
3. **Handle timezone explicitly** in date operations
4. **Check for empty strings** from spreadsheet cells, not null
5. **Use type guards** before operations
6. **Be careful with number precision** in financial calculations
7. **Encode/decode properly** when working with URLs and HTML
8. **Use JSON for complex data storage** in Properties Service
9. **Validate array dimensions** from getValues()
10. **Handle blob size limits** (50MB for most operations)