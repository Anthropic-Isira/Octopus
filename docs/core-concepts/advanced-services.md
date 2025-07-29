# Advanced Google Services

Advanced services in Apps Script let experienced developers connect to certain public Google APIs with less set-up than using their HTTP interfaces. Advanced services are essentially thin wrappers around those Google APIs. They work much like Apps Script's built-in services—for example, they offer autocomplete, and Apps Script handles the authorization flow automatically. However, you must enable an advanced service before you can use it in a script.

To see which Google APIs are available as advanced services, look for the Advanced Google Services section in the Reference. If you want to use a Google API that isn't available as an advanced service, just connect to it like any other external API.

## Advanced Services or HTTP?

Each of the advanced Google services is associated with a public Google API. In Apps Script, you can access these APIs via advanced services or by simply making the API requests directly using UrlFetch.

If you use the advanced service method, Apps Script handles the authorization flow and offers autocomplete support. However, you must enable the advanced service before you can use it. In addition, some advanced services only provide a subset of the functionality available in the API.

If you use the UrlFetch method to access the API directly, you are essentially treating the Google API as an external API. With this method, all aspects of the API can be used. However, it requires you to handle the API authorization yourself. You must also construct any needed headers and parse the API responses.

In general it's easiest to use an advanced service where possible and only use the UrlFetch method when the advanced service doesn't provide the functionality you need.

## Requirements

Before you can use an advanced service, you must satisfy the following requirements:

- You must enable the advanced service in your script project.
- You must make sure the API corresponding to the advanced service is enabled in the Cloud Platform (GCP) project your script uses.

If your script project uses a default GCP project created on or after April 8, 2019, the API is enabled automatically after you enable the advanced service and save the script project. If you have not done so already, you may also be asked to agree to the Google Cloud and Google APIs Terms of Service as well.

If your script project uses a standard GCP project or an older default GCP project, you must enable the advanced service's corresponding API in the GCP project manually. You must have edit access to the GCP project to make this change.

See Cloud Platform projects for more information.

## Enable Advanced Services

To use an advanced Google service, follow these instructions:

1. Open the Apps Script project.
2. At the left, click Editor code.
3. At the left, next to Services, click Add a service add.
4. Select an advanced Google service and click Add.

**Note:** If you're using a standard GCP project (or an older default GCP project that was created prior to April 8, 2019), you must also manually enable the API corresponding to the advanced service. Enable the API by doing the following:
- Open the GCP project associated with your script in the Google Cloud console.
- At the top of the console, click into the search bar and type part of the name of the API (for example, "Calendar"), then click the name once you see it.
- On the next screen, click Enable API.
- Close the Google Cloud console and return to the script editor. Click OK in the dialog.

After you enable an advanced service, it's available in autocomplete.

## How Method Signatures are Determined

Advanced services generally use the same objects, method names, and parameters as the corresponding public APIs, although method signatures are translated for use in Apps Script. The script editor's autocomplete function usually provides enough information to get started, but the rules below explain how Apps Script generates a method signature from a public Google API.

Requests to Google APIs can accept a variety of different types of data, including path parameters, query parameters, a request body, and/or a media upload attachment. Some advanced services can also accept specific HTTP request headers (for example, the Calendar advanced service).

The corresponding method signature in Google Apps Script has the following arguments:

1. The request body (usually a resource), as a JavaScript object.
2. Path or required parameters, as individual arguments.
3. The media upload attachment, as a Blob argument.
4. Optional parameters, as a JavaScript object mapping parameter names to values.
5. HTTP request headers, as a JavaScript object mapping header names to header values.

If the method doesn't have any items in a given category, that part of the signature is omitted.

There are some special exceptions to be aware of:

- For methods that accept a media upload, the parameter `uploadType` is set automatically.
- Methods named `delete` in the Google API are named `remove` in Apps Script, since `delete` is a reserved word in JavaScript.
- If an advanced service is configured to accept HTTP request headers, and you set a request headers JavaScript object, then you must also set the optional parameters JavaScript object (to an empty object if you aren't using optional parameters).

## Support for Advanced Services

Advanced services are just thin wrappers that enables the use of a Google APIs within Apps Script. As such, any issue encountered while using them is usually an issue with the underlying API, not with Apps Script itself.

If you encounter a problem while using an advanced service, it should be reported using the support instructions for the underlying API. Links to these support instructions are provided in each advanced service guide in the Apps Script Reference section.

## Common Advanced Services

### Drive API v3

The Drive API provides more features than DriveApp.

```javascript
// Enable: Drive API v3 (identifier: Drive)

// List files with more details
function listFilesAdvanced() {
  var response = Drive.Files.list({
    pageSize: 100,
    fields: 'files(id, name, mimeType, size, createdTime, modifiedTime, owners, permissions)',
    q: "mimeType='application/vnd.google-apps.spreadsheet'",
    orderBy: 'modifiedTime desc'
  });
  
  response.files.forEach(function(file) {
    console.log(file.name + ' - ' + file.size + ' bytes');
    
    // Access detailed permissions
    file.permissions.forEach(function(permission) {
      console.log('  ' + permission.role + ': ' + permission.emailAddress);
    });
  });
}

// Create file with metadata
function createFileAdvanced() {
  var fileMetadata = {
    name: 'Advanced Document',
    mimeType: 'application/vnd.google-apps.document',
    parents: ['FOLDER_ID'],
    description: 'Created via Drive API',
    properties: {
      department: 'Sales',
      project: 'Q4Campaign'
    }
  };
  
  var file = Drive.Files.create(fileMetadata);
  console.log('Created file:', file.id);
}

// Batch update permissions
function batchUpdatePermissions(fileId) {
  var batch = {
    requests: [
      {
        createPermission: {
          fileId: fileId,
          permission: {
            type: 'user',
            role: 'writer',
            emailAddress: 'user1@example.com'
          }
        }
      },
      {
        createPermission: {
          fileId: fileId,
          permission: {
            type: 'user',
            role: 'reader',
            emailAddress: 'user2@example.com'
          }
        }
      }
    ]
  };
  
  // Note: Batch requests require additional setup
  var response = Drive.Files.update({
    fileId: fileId,
    addParents: 'NEW_FOLDER_ID',
    removeParents: 'OLD_FOLDER_ID',
    fields: 'id, parents'
  });
}
```

### Sheets API v4

More powerful than SpreadsheetApp for batch operations.

```javascript
// Enable: Google Sheets API (identifier: Sheets)

// Batch update multiple ranges
function batchUpdateSpreadsheet(spreadsheetId) {
  var requests = [
    {
      updateCells: {
        range: {
          sheetId: 0,
          startRowIndex: 0,
          endRowIndex: 1,
          startColumnIndex: 0,
          endColumnIndex: 3
        },
        rows: [{
          values: [
            {userEnteredValue: {stringValue: 'Name'}},
            {userEnteredValue: {stringValue: 'Email'}},
            {userEnteredValue: {stringValue: 'Score'}}
          ]
        }],
        fields: 'userEnteredValue'
      }
    },
    {
      repeatCell: {
        range: {
          sheetId: 0,
          startRowIndex: 0,
          endRowIndex: 1,
          startColumnIndex: 0,
          endColumnIndex: 3
        },
        cell: {
          userEnteredFormat: {
            backgroundColor: {red: 0.0, green: 0.0, blue: 1.0},
            textFormat: {
              foregroundColor: {red: 1.0, green: 1.0, blue: 1.0},
              bold: true
            }
          }
        },
        fields: 'userEnteredFormat(backgroundColor,textFormat)'
      }
    }
  ];
  
  var response = Sheets.Spreadsheets.batchUpdate({
    requests: requests
  }, spreadsheetId);
  
  console.log('Updated ' + response.replies.length + ' cells');
}

// Get values with specific render options
function getValuesAdvanced(spreadsheetId) {
  var response = Sheets.Spreadsheets.Values.batchGet(spreadsheetId, {
    ranges: ['Sheet1!A1:C10', 'Sheet2!D1:F10'],
    valueRenderOption: 'FORMULA',  // or 'FORMATTED_VALUE', 'UNFORMATTED_VALUE'
    dateTimeRenderOption: 'FORMATTED_STRING'  // or 'SERIAL_NUMBER'
  });
  
  response.valueRanges.forEach(function(valueRange) {
    console.log('Range: ' + valueRange.range);
    console.log('Values: ' + JSON.stringify(valueRange.values));
  });
}

// Append with different input options
function appendValuesAdvanced(spreadsheetId) {
  var values = [
    ['John Doe', 'john@example.com', '=B2&"@company.com"'],
    ['Jane Smith', 'jane@example.com', '=LEN(A3)']
  ];
  
  var response = Sheets.Spreadsheets.Values.append({
    range: 'Sheet1!A1',
    valueInputOption: 'USER_ENTERED',  // Interprets formulas
    insertDataOption: 'INSERT_ROWS',
    includeValuesInResponse: true,
    values: values
  }, spreadsheetId);
  
  console.log('Appended range: ' + response.updates.updatedRange);
}
```

### Calendar API v3

Advanced calendar operations not available in CalendarApp.

```javascript
// Enable: Calendar API (identifier: Calendar)

// List calendars with details
function listCalendarsAdvanced() {
  var response = Calendar.CalendarList.list({
    showDeleted: false,
    showHidden: true,
    minAccessRole: 'writer'
  });
  
  response.items.forEach(function(calendar) {
    console.log(calendar.summary + ' (' + calendar.accessRole + ')');
    console.log('  Primary: ' + calendar.primary);
    console.log('  Color: ' + calendar.backgroundColor);
    console.log('  Time Zone: ' + calendar.timeZone);
  });
}

// Create recurring event with advanced options
function createRecurringEventAdvanced(calendarId) {
  var event = {
    summary: 'Weekly Team Meeting',
    location: 'Conference Room A',
    description: 'Weekly sync with the team',
    start: {
      dateTime: '2024-01-15T10:00:00',
      timeZone: 'America/New_York'
    },
    end: {
      dateTime: '2024-01-15T11:00:00',
      timeZone: 'America/New_York'
    },
    recurrence: [
      'RRULE:FREQ=WEEKLY;BYDAY=MO;UNTIL=20241231T000000Z'
    ],
    attendees: [
      {email: 'team@example.com', optional: false},
      {email: 'manager@example.com', optional: true}
    ],
    conferenceData: {
      createRequest: {
        requestId: Utilities.getUuid(),
        conferenceSolutionKey: {type: 'hangoutsMeet'}
      }
    },
    reminders: {
      useDefault: false,
      overrides: [
        {method: 'email', minutes: 24 * 60},
        {method: 'popup', minutes: 10}
      ]
    },
    colorId: '2',  // Color palette ID
    transparency: 'opaque',  // or 'transparent'
    visibility: 'private'  // or 'public', 'confidential'
  };
  
  var response = Calendar.Events.insert(event, calendarId, {
    conferenceDataVersion: 1,
    sendUpdates: 'all'  // or 'externalOnly', 'none'
  });
  
  console.log('Event created: ' + response.htmlLink);
  console.log('Meet link: ' + response.conferenceData.entryPoints[0].uri);
}

// Batch get events
function batchGetEvents(calendarId, eventIds) {
  var batch = eventIds.map(function(eventId) {
    return Calendar.Events.get(calendarId, eventId);
  });
  
  // Process responses
  batch.forEach(function(event) {
    console.log(event.summary + ' - ' + event.start.dateTime);
  });
}
```

### Gmail API

Advanced email operations beyond GmailApp.

```javascript
// Enable: Gmail API (identifier: Gmail)

// Get message with full details
function getMessageAdvanced(messageId) {
  var message = Gmail.Users.Messages.get('me', messageId, {
    format: 'full'  // or 'metadata', 'minimal', 'raw'
  });
  
  // Parse message parts
  var parts = message.payload.parts || [];
  parts.forEach(function(part) {
    if (part.mimeType === 'text/plain') {
      var data = Utilities.newBlob(
        Utilities.base64Decode(part.body.data)
      ).getDataAsString();
      console.log('Plain text:', data);
    }
  });
  
  // Get headers
  var headers = message.payload.headers;
  headers.forEach(function(header) {
    if (['From', 'To', 'Subject', 'Date'].indexOf(header.name) !== -1) {
      console.log(header.name + ': ' + header.value);
    }
  });
}

// Batch modify messages
function batchModifyMessages(query) {
  var threads = Gmail.Users.Threads.list('me', {
    q: query,
    maxResults: 100
  });
  
  var messageIds = [];
  threads.threads.forEach(function(thread) {
    thread.messages.forEach(function(message) {
      messageIds.push(message.id);
    });
  });
  
  // Batch modify
  var response = Gmail.Users.Messages.batchModify({
    ids: messageIds,
    addLabelIds: ['LABEL_ID_1'],
    removeLabelIds: ['UNREAD', 'INBOX']
  }, 'me');
  
  console.log('Modified ' + messageIds.length + ' messages');
}

// Send email with attachments using Gmail API
function sendEmailAdvanced() {
  var message = {
    to: 'recipient@example.com',
    subject: 'Test Email',
    htmlBody: '<h1>Hello</h1><p>This is a test.</p>',
    attachments: [
      DriveApp.getFileById('FILE_ID').getBlob()
    ]
  };
  
  // Create MIME message
  var raw = createMimeMessage(message);
  
  Gmail.Users.Messages.send({
    raw: Utilities.base64EncodeWebSafe(raw)
  }, 'me');
}
```

## Rate Limiting and Quotas

### Understanding API Quotas

```javascript
// Check quota usage
function checkQuotaUsage() {
  // Most Google APIs have these common limits:
  var quotas = {
    'Drive API': {
      queriesPerDay: 1000000000,  // 1 billion
      queriesPerMinute: 12000,
      queriesPerMinutePerUser: 60
    },
    'Sheets API': {
      requestsPerMinute: 60,
      requestsPerMinutePerUser: 60,
      readRequestsPerMinute: 300
    },
    'Calendar API': {
      queriesPerDay: 1000000,
      queriesPerMinute: 600,
      queriesPerMinutePerUser: 60
    },
    'Gmail API': {
      dailyUsage: 1000000000,  // 1 billion
      perUserRateLimit: 250,  // quota units per user per second
      messageSendLimit: 500  // per user per day
    }
  };
  
  console.log('Check console.cloud.google.com for actual usage');
}
```

### Implementing Rate Limiting

```javascript
class RateLimiter {
  constructor(maxRequests, windowMs) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
    this.requests = [];
  }
  
  canMakeRequest() {
    var now = new Date().getTime();
    var windowStart = now - this.windowMs;
    
    // Remove old requests
    this.requests = this.requests.filter(time => time > windowStart);
    
    if (this.requests.length < this.maxRequests) {
      this.requests.push(now);
      return true;
    }
    
    return false;
  }
  
  waitTime() {
    if (this.requests.length === 0) return 0;
    
    var oldestRequest = Math.min.apply(null, this.requests);
    var now = new Date().getTime();
    var windowStart = now - this.windowMs;
    
    return Math.max(0, oldestRequest + this.windowMs - now);
  }
}

// Usage
var limiter = new RateLimiter(60, 60000); // 60 requests per minute

function makeApiCall() {
  if (!limiter.canMakeRequest()) {
    var waitMs = limiter.waitTime();
    console.log('Rate limit reached. Waiting ' + waitMs + 'ms');
    Utilities.sleep(waitMs);
  }
  
  // Make API call
  var response = Drive.Files.list({pageSize: 10});
  return response;
}
```

### Batch Operations

```javascript
// Batch operations to reduce API calls
function batchOperations() {
  // Instead of multiple API calls
  // BAD:
  for (var i = 0; i < 100; i++) {
    Drive.Files.get(fileIds[i]);  // 100 API calls!
  }
  
  // GOOD: Use batch requests where available
  var batch = Drive.newBatch();
  fileIds.forEach(function(fileId) {
    batch.add(Drive.Files.get(fileId));
  });
  var responses = batch.execute();
  
  // For Sheets API
  var requests = [];
  for (var i = 0; i < 10; i++) {
    requests.push({
      updateCells: {
        range: {sheetId: 0, startRowIndex: i, endRowIndex: i + 1},
        rows: [{values: [{userEnteredValue: {stringValue: 'Value'}}]}],
        fields: 'userEnteredValue'
      }
    });
  }
  
  Sheets.Spreadsheets.batchUpdate({requests: requests}, spreadsheetId);
}
```

## Error Handling for API Calls

### Comprehensive Error Handling

```javascript
function apiCallWithErrorHandling() {
  var maxRetries = 3;
  var backoffMs = 1000;
  
  for (var attempt = 0; attempt < maxRetries; attempt++) {
    try {
      var response = Drive.Files.list({
        pageSize: 100,
        fields: 'files(id, name)'
      });
      
      return response;
      
    } catch (error) {
      console.error('API Error:', error.toString());
      
      // Parse error details
      var errorCode = error.details?.code || error.code;
      var errorMessage = error.details?.message || error.message;
      
      // Handle specific errors
      switch (errorCode) {
        case 403:
          if (errorMessage.includes('Rate Limit')) {
            // Rate limit - wait and retry
            var waitTime = backoffMs * Math.pow(2, attempt);
            console.log('Rate limited. Waiting ' + waitTime + 'ms');
            Utilities.sleep(waitTime);
            continue;
          } else if (errorMessage.includes('forbidden')) {
            // Permission error - don't retry
            console.error('Permission denied. Check OAuth scopes.');
            throw error;
          }
          break;
          
        case 404:
          // Not found - don't retry
          console.error('Resource not found');
          throw error;
          
        case 500:
        case 502:
        case 503:
          // Server error - retry with backoff
          var waitTime = backoffMs * Math.pow(2, attempt);
          console.log('Server error. Retrying in ' + waitTime + 'ms');
          Utilities.sleep(waitTime);
          continue;
          
        case 429:
          // Too many requests
          var retryAfter = error.details?.retryAfter || backoffMs * Math.pow(2, attempt);
          console.log('Too many requests. Waiting ' + retryAfter + 'ms');
          Utilities.sleep(retryAfter);
          continue;
          
        default:
          // Unknown error
          console.error('Unknown error:', errorCode, errorMessage);
          throw error;
      }
    }
  }
  
  throw new Error('Max retries exceeded');
}
```

### Response Validation

```javascript
function validateApiResponse(response) {
  if (!response) {
    throw new Error('Empty response from API');
  }
  
  // Check for error in response
  if (response.error) {
    throw new Error('API Error: ' + response.error.message);
  }
  
  // Validate expected fields
  if (response.files && !Array.isArray(response.files)) {
    throw new Error('Invalid response format: files should be array');
  }
  
  return response;
}
```

## Working with Pagination

### Handling Large Result Sets

```javascript
function getAllResults(query) {
  var allFiles = [];
  var pageToken = null;
  var pageSize = 100;
  
  do {
    try {
      var response = Drive.Files.list({
        q: query,
        pageSize: pageSize,
        pageToken: pageToken,
        fields: 'nextPageToken, files(id, name, mimeType, modifiedTime)'
      });
      
      if (response.files && response.files.length > 0) {
        allFiles = allFiles.concat(response.files);
      }
      
      pageToken = response.nextPageToken;
      
      // Prevent timeout on large datasets
      if (allFiles.length % 1000 === 0) {
        console.log('Processed ' + allFiles.length + ' files...');
        Utilities.sleep(100); // Brief pause
      }
      
    } catch (error) {
      console.error('Error fetching page:', error);
      break;
    }
  } while (pageToken);
  
  console.log('Total files found: ' + allFiles.length);
  return allFiles;
}

// Generator pattern for memory efficiency
function* fileGenerator(query) {
  var pageToken = null;
  
  do {
    var response = Drive.Files.list({
      q: query,
      pageSize: 100,
      pageToken: pageToken
    });
    
    for (var file of response.files) {
      yield file;
    }
    
    pageToken = response.nextPageToken;
  } while (pageToken);
}

// Usage
function processLargeDataset() {
  var generator = fileGenerator('mimeType = "application/pdf"');
  var count = 0;
  
  for (var file of generator) {
    processFile(file);
    count++;
    
    if (count % 100 === 0) {
      console.log('Processed ' + count + ' files');
    }
  }
}
```

## Field Masks and Partial Responses

### Optimizing API Responses

```javascript
// Request only needed fields
function efficientApiCalls() {
  // Drive API - specify fields
  var driveResponse = Drive.Files.list({
    fields: 'files(id, name)',  // Only get id and name
    pageSize: 100
  });
  
  // Calendar API - partial response
  var calendarResponse = Calendar.Events.list('primary', {
    fields: 'items(summary, start, end)',
    timeMin: new Date().toISOString(),
    maxResults: 50
  });
  
  // Sheets API - specific ranges
  var sheetsResponse = Sheets.Spreadsheets.Values.batchGet(spreadsheetId, {
    ranges: ['A1:A10', 'D1:D10'],  // Only get specific columns
    majorDimension: 'COLUMNS'
  });
}

// Update only specific fields
function partialUpdate(fileId) {
  // Only update name and description
  var response = Drive.Files.update({
    name: 'New Name',
    description: 'Updated description'
  }, fileId, {
    fields: 'id, name, description, modifiedTime'
  });
  
  console.log('Updated file:', response.name);
}
```

## Advanced Authentication

### Service Account Impersonation

```javascript
// When using service accounts (requires domain-wide delegation)
function impersonateUser(userEmail) {
  // This requires setup in Google Cloud Console
  var service = getDriveService(userEmail);
  
  if (service.hasAccess()) {
    var response = UrlFetchApp.fetch(
      'https://www.googleapis.com/drive/v3/files',
      {
        headers: {
          Authorization: 'Bearer ' + service.getAccessToken()
        }
      }
    );
    
    return JSON.parse(response.getContentText());
  }
}
```

## Best Practices

### 1. Use Field Masks

```javascript
// Always specify only the fields you need
var response = Drive.Files.list({
  fields: 'files(id, name)',  // Not 'files' or '*'
  pageSize: 100
});
```

### 2. Implement Exponential Backoff

```javascript
function exponentialBackoff(func, maxRetries = 5) {
  var baseDelay = 1000;
  
  for (var i = 0; i < maxRetries; i++) {
    try {
      return func();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      var delay = baseDelay * Math.pow(2, i) + Math.random() * 1000;
      Utilities.sleep(delay);
    }
  }
}
```

### 3. Use Batch Operations

```javascript
// Combine multiple operations when possible
function batchExample() {
  var requests = [];
  
  // Build batch
  for (var i = 0; i < 50; i++) {
    requests.push({
      method: 'GET',
      endpoint: '/files/' + fileIds[i]
    });
  }
  
  // Execute batch
  var responses = executeBatch(requests);
}
```

### 4. Cache API Responses

```javascript
function cachedApiCall(cacheKey, apiFunction, expirationSeconds = 3600) {
  var cache = CacheService.getScriptCache();
  var cached = cache.get(cacheKey);
  
  if (cached) {
    return JSON.parse(cached);
  }
  
  var result = apiFunction();
  cache.put(cacheKey, JSON.stringify(result), expirationSeconds);
  
  return result;
}
```

### 5. Monitor Quota Usage

```javascript
function monitorQuota() {
  var props = PropertiesService.getScriptProperties();
  var quotaKey = 'api_calls_' + new Date().toDateString();
  var currentCount = parseInt(props.getProperty(quotaKey) || '0');
  
  currentCount++;
  props.setProperty(quotaKey, currentCount.toString());
  
  // Alert if approaching limits
  if (currentCount > 900) {
    console.warn('Approaching daily API limit: ' + currentCount);
  }
  
  return currentCount;
}
```