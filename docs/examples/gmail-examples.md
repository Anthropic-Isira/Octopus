# Gmail Service Examples

[← Back to Gmail API](../services/gmail.md) | [← Back to Main Documentation](../README.md)

## Basic Operations

### Sending Emails

```javascript
// Send a simple email
GmailApp.sendEmail('recipient@example.com', 'Subject Line', 'Email body text');

// Send email with options
GmailApp.sendEmail('recipient@example.com', 'Meeting Reminder', 'Don\'t forget about our meeting!', {
  cc: 'cc@example.com',
  bcc: 'bcc@example.com',
  htmlBody: '<h2>Meeting Reminder</h2><p>Don\'t forget about our <b>meeting</b>!</p>',
  replyTo: 'noreply@example.com',
  attachments: [DriveApp.getFileById('FILE_ID').getBlob()]
});

// Send email with multiple attachments
var file1 = DriveApp.getFileById('FILE_ID_1').getBlob();
var file2 = DriveApp.getFileById('FILE_ID_2').getBlob();

GmailApp.sendEmail('recipient@example.com', 'Documents Attached', 'Please find the documents attached.', {
  attachments: [file1, file2]
});
```

### Reading Emails

```javascript
// Get inbox threads
var threads = GmailApp.getInboxThreads(0, 10); // First 10 threads

threads.forEach(function(thread) {
  console.log('Subject: ' + thread.getFirstMessageSubject());
  console.log('Message count: ' + thread.getMessageCount());
  console.log('Last message date: ' + thread.getLastMessageDate());
});

// Get specific messages
var messages = threads[0].getMessages();
messages.forEach(function(message) {
  console.log('From: ' + message.getFrom());
  console.log('Date: ' + message.getDate());
  console.log('Body: ' + message.getPlainBody());
});
```

### Searching Emails

```javascript
// Search with Gmail search operators
var threads = GmailApp.search('from:important@example.com is:unread');

// Search with date range
var threads = GmailApp.search('after:2024/1/1 before:2024/1/31 has:attachment');

// Complex search
var threads = GmailApp.search('subject:"Project Update" OR subject:"Status Report" -label:processed');

// Search with size limits
var threads = GmailApp.search('larger:5M', 0, 50); // First 50 results
```

## Advanced Examples

### Email Processing System

```javascript
function processIncomingEmails() {
  var label = GmailApp.getUserLabelByName('To Process') || 
              GmailApp.createLabel('To Process');
  var processedLabel = GmailApp.getUserLabelByName('Processed') || 
                       GmailApp.createLabel('Processed');
  
  var threads = label.getThreads();
  
  threads.forEach(function(thread) {
    var messages = thread.getMessages();
    
    messages.forEach(function(message) {
      // Process only unread messages
      if (message.isUnread()) {
        processMessage(message);
        message.markRead();
      }
    });
    
    // Move to processed
    thread.removeLabel(label);
    thread.addLabel(processedLabel);
  });
}

function processMessage(message) {
  var subject = message.getSubject();
  var from = message.getFrom();
  var body = message.getPlainBody();
  
  // Extract data, save to spreadsheet, etc.
  console.log('Processing: ' + subject + ' from ' + from);
}
```

### Attachment Handler

```javascript
function saveEmailAttachments() {
  var query = 'has:attachment is:unread';
  var threads = GmailApp.search(query);
  var folder = DriveApp.createFolder('Email Attachments - ' + new Date().toDateString());
  
  threads.forEach(function(thread) {
    var messages = thread.getMessages();
    
    messages.forEach(function(message) {
      if (message.isUnread()) {
        var attachments = message.getAttachments();
        
        attachments.forEach(function(attachment) {
          // Check file type and size
          if (attachment.getSize() < 25 * 1024 * 1024) { // Less than 25MB
            var file = folder.createFile(attachment);
            console.log('Saved: ' + file.getName());
          }
        });
        
        message.markRead();
      }
    });
  });
  
  return folder.getUrl();
}
```

### Draft Management

```javascript
// Create a draft
var draft = GmailApp.createDraft('recipient@example.com', 
  'Draft Subject', 
  'This is a draft message.', {
    cc: 'cc@example.com',
    attachments: [DriveApp.getFileById('FILE_ID').getBlob()]
  });

console.log('Draft created with ID: ' + draft.getId());

// Update a draft
draft.update('recipient@example.com',
  'Updated Subject',
  'This is the updated draft content.', {
    cc: 'newcc@example.com'
  });

// Get all drafts
var drafts = GmailApp.getDrafts();
drafts.forEach(function(draft) {
  var message = draft.getMessage();
  console.log('Draft: ' + message.getSubject());
});

// Send a draft
draft.send();
```

### Label Management

```javascript
function organizEmailsByLabels() {
  // Create labels
  var projectLabel = GmailApp.createLabel('Projects');
  var clientLabel = GmailApp.createLabel('Clients');
  var urgentLabel = GmailApp.createLabel('Urgent');
  
  // Apply labels based on criteria
  var threads = GmailApp.getInboxThreads(0, 50);
  
  threads.forEach(function(thread) {
    var subject = thread.getFirstMessageSubject();
    var messages = thread.getMessages();
    var firstMessage = messages[0];
    
    // Label by subject keywords
    if (subject.includes('Project') || subject.includes('PRJ-')) {
      thread.addLabel(projectLabel);
    }
    
    // Label by sender
    if (firstMessage.getFrom().includes('@client.com')) {
      thread.addLabel(clientLabel);
    }
    
    // Label urgent emails
    if (subject.includes('URGENT') || subject.includes('ASAP')) {
      thread.addLabel(urgentLabel);
      thread.markImportant();
    }
  });
}
```

## Practical Use Cases

### Auto-Responder

```javascript
function setupAutoResponder() {
  var threads = GmailApp.search('is:unread to:me');
  var responded = [];
  
  threads.forEach(function(thread) {
    var messages = thread.getMessages();
    var lastMessage = messages[messages.length - 1];
    
    // Check if we haven't already responded
    var from = lastMessage.getFrom();
    if (!responded.includes(from) && !lastMessage.isDraft()) {
      
      // Send auto-response
      GmailApp.sendEmail(
        lastMessage.getReplyTo() || from,
        'Re: ' + lastMessage.getSubject(),
        'Thank you for your email. I am currently out of office and will respond to your message when I return.\n\n' +
        'For urgent matters, please contact support@example.com.\n\n' +
        'Best regards',
        {
          replyTo: 'noreply@example.com'
        }
      );
      
      responded.push(from);
      lastMessage.markRead();
    }
  });
}
```

### Email Report Generator

```javascript
function generateEmailReport(startDate, endDate) {
  var searchQuery = 'after:' + formatDate(startDate) + ' before:' + formatDate(endDate);
  var threads = GmailApp.search(searchQuery);
  
  var report = {
    totalThreads: threads.length,
    totalMessages: 0,
    senders: {},
    subjects: [],
    attachmentCount: 0,
    labelDistribution: {}
  };
  
  threads.forEach(function(thread) {
    var messages = thread.getMessages();
    report.totalMessages += messages.length;
    
    // Track subjects
    report.subjects.push({
      subject: thread.getFirstMessageSubject(),
      messageCount: messages.length,
      lastDate: thread.getLastMessageDate()
    });
    
    // Analyze messages
    messages.forEach(function(message) {
      // Track senders
      var sender = message.getFrom();
      report.senders[sender] = (report.senders[sender] || 0) + 1;
      
      // Count attachments
      report.attachmentCount += message.getAttachments().length;
    });
    
    // Track labels
    var labels = thread.getLabels();
    labels.forEach(function(label) {
      var labelName = label.getName();
      report.labelDistribution[labelName] = (report.labelDistribution[labelName] || 0) + 1;
    });
  });
  
  return report;
}

function formatDate(date) {
  return date.getFullYear() + '/' + (date.getMonth() + 1) + '/' + date.getDate();
}
```

### Email-to-Task Converter

```javascript
function convertEmailsToTasks() {
  var label = GmailApp.getUserLabelByName('Convert to Task');
  if (!label) return;
  
  var threads = label.getThreads();
  var tasks = [];
  
  threads.forEach(function(thread) {
    var firstMessage = thread.getMessages()[0];
    
    var task = {
      title: thread.getFirstMessageSubject(),
      description: firstMessage.getPlainBody().substring(0, 500),
      from: firstMessage.getFrom(),
      date: firstMessage.getDate(),
      threadId: thread.getId(),
      priority: thread.isImportant() ? 'High' : 'Normal'
    };
    
    tasks.push(task);
    
    // Create task in external system (example)
    createTaskInSystem(task);
    
    // Remove label and archive
    thread.removeLabel(label);
    thread.moveToArchive();
  });
  
  return tasks;
}

function createTaskInSystem(task) {
  // Example: Save to spreadsheet
  var sheet = SpreadsheetApp.openById('SPREADSHEET_ID').getActiveSheet();
  sheet.appendRow([
    task.title,
    task.description,
    task.from,
    task.date,
    task.priority,
    task.threadId
  ]);
}
```

### Newsletter Manager

```javascript
function manageNewsletters() {
  var unsubscribeList = [];
  var newsletters = GmailApp.search('unsubscribe OR "email preferences" -label:reviewed');
  
  newsletters.forEach(function(thread) {
    var messages = thread.getMessages();
    var lastMessage = messages[messages.length - 1];
    
    // Extract sender info
    var from = lastMessage.getFrom();
    var subject = lastMessage.getSubject();
    
    // Look for unsubscribe link
    var body = lastMessage.getBody();
    var unsubscribeMatch = body.match(/https?:\/\/[^\s]*unsubscribe[^\s]*/i);
    
    if (unsubscribeMatch) {
      unsubscribeList.push({
        from: from,
        subject: subject,
        unsubscribeLink: unsubscribeMatch[0],
        messageDate: lastMessage.getDate()
      });
    }
    
    // Label as reviewed
    var reviewedLabel = GmailApp.getUserLabelByName('reviewed') || 
                       GmailApp.createLabel('reviewed');
    thread.addLabel(reviewedLabel);
  });
  
  // Create unsubscribe report
  if (unsubscribeList.length > 0) {
    createUnsubscribeReport(unsubscribeList);
  }
  
  return unsubscribeList;
}
```

### Email Backup System

```javascript
function backupEmails(labelName, backupFolderId) {
  var label = GmailApp.getUserLabelByName(labelName);
  if (!label) return;
  
  var backupFolder = DriveApp.getFolderById(backupFolderId);
  var dateFolder = backupFolder.createFolder('Email Backup - ' + new Date().toDateString());
  
  var threads = label.getThreads();
  var backupData = [];
  
  threads.forEach(function(thread) {
    var threadData = {
      subject: thread.getFirstMessageSubject(),
      messageCount: thread.getMessageCount(),
      messages: []
    };
    
    var messages = thread.getMessages();
    messages.forEach(function(message) {
      var messageData = {
        from: message.getFrom(),
        to: message.getTo(),
        cc: message.getCc(),
        date: message.getDate(),
        subject: message.getSubject(),
        body: message.getPlainBody(),
        htmlBody: message.getBody()
      };
      
      // Save attachments
      var attachments = message.getAttachments();
      if (attachments.length > 0) {
        var attachmentFolder = dateFolder.createFolder(
          'Attachments - ' + message.getId()
        );
        
        messageData.attachments = [];
        attachments.forEach(function(attachment) {
          var file = attachmentFolder.createFile(attachment);
          messageData.attachments.push(file.getName());
        });
      }
      
      threadData.messages.push(messageData);
    });
    
    backupData.push(threadData);
  });
  
  // Save backup data as JSON
  var jsonBlob = Utilities.newBlob(
    JSON.stringify(backupData, null, 2),
    'application/json',
    'email_backup.json'
  );
  dateFolder.createFile(jsonBlob);
  
  return dateFolder.getUrl();
}
```

## Error Handling

```javascript
function safeEmailOperation(operation) {
  try {
    return operation();
  } catch (e) {
    console.error('Email operation failed: ' + e.toString());
    
    if (e.toString().includes('Invalid email')) {
      console.error('Please check email addresses');
    } else if (e.toString().includes('Limit Exceeded')) {
      console.error('Gmail quota exceeded, please try again later');
    } else if (e.toString().includes('Message not found')) {
      console.error('Message or thread no longer exists');
    }
    
    return null;
  }
}

// Usage
var result = safeEmailOperation(function() {
  return GmailApp.sendEmail('recipient@example.com', 'Test', 'Body');
});
```

## Best Practices

1. **Batch Operations**: Process emails in batches to avoid timeouts:
```javascript
function processBatches() {
  var batchSize = 100;
  var start = 0;
  
  while (true) {
    var threads = GmailApp.getInboxThreads(start, batchSize);
    if (threads.length === 0) break;
    
    // Process batch
    threads.forEach(function(thread) {
      // Process thread
    });
    
    start += batchSize;
  }
}
```

2. **Use Labels for State Management**:
```javascript
function statefulProcessing() {
  var toProcess = GmailApp.getUserLabelByName('to-process');
  var processing = GmailApp.getUserLabelByName('processing');
  var processed = GmailApp.getUserLabelByName('processed');
  
  var threads = toProcess.getThreads(0, 10);
  threads.forEach(function(thread) {
    thread.addLabel(processing);
    
    try {
      // Process thread
      thread.addLabel(processed);
    } catch (e) {
      // Keep in processing for retry
      console.error('Failed to process: ' + thread.getId());
    }
    
    thread.removeLabel(toProcess);
    thread.removeLabel(processing);
  });
}
```

3. **Respect Rate Limits**:
```javascript
function sendBulkEmails(recipients, subject, body) {
  var sentCount = 0;
  var quota = MailApp.getRemainingDailyQuota();
  
  recipients.forEach(function(recipient, index) {
    if (sentCount >= quota - 10) { // Leave some buffer
      console.log('Approaching quota limit, stopping at ' + sentCount);
      return;
    }
    
    try {
      GmailApp.sendEmail(recipient, subject, body);
      sentCount++;
      
      // Add delay to avoid rate limiting
      if (index % 50 === 0) {
        Utilities.sleep(1000); // 1 second delay every 50 emails
      }
    } catch (e) {
      console.error('Failed to send to: ' + recipient);
    }
  });
  
  return sentCount;
}
```