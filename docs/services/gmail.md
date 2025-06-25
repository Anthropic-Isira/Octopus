# Gmail Service API Reference

[← Back to Main Documentation](../README.md) | [View Examples →](../examples/gmail-examples.md)

## Service Overview
**SERVICE_NAME:** GmailApp  
**DESCRIPTION:** Provides access to Gmail threads, messages, and labels. Allows sending email, composing drafts, managing labels, and other Gmail account management tasks  
**ACCESS_PATTERN:** `GmailApp.methodName()`  
**NOTE:** For simpler email sending only, see Mail Service

## Core Classes

### CLASS: GmailApp
**METHODS:**
- `createDraft(recipient, subject, body)` → GmailDraft: Creates draft email
- `createDraft(recipient, subject, body, options)` → GmailDraft: Creates draft with options
- `createLabel(name)` → GmailLabel: Creates new user label
- `deleteLabel(label)` → GmailApp: Deletes specified label
- `getAliases()` → String[]: Gets account email aliases
- `getDraft(draftId)` → GmailDraft: Retrieves draft by ID
- `getDraftMessages()` → GmailMessage[]: Gets all draft messages
- `getDrafts()` → GmailDraft[]: Gets all drafts
- `getInboxThreads()` → GmailThread[]: Gets all inbox threads
- `getInboxThreads(start, max)` → GmailThread[]: Gets range of inbox threads
- `getInboxUnreadCount()` → Integer: Gets unread inbox count
- `getMessageById(id)` → GmailMessage: Gets message by ID
- `getMessagesForThread(thread)` → GmailMessage[]: Gets thread messages
- `getPriorityInboxThreads()` → GmailThread[]: Gets priority inbox threads
- `getSpamThreads()` → GmailThread[]: Gets spam threads
- `getStarredThreads()` → GmailThread[]: Gets starred threads
- `getThreadById(id)` → GmailThread: Gets thread by ID
- `getTrashThreads()` → GmailThread[]: Gets trash threads
- `getUserLabelByName(name)` → GmailLabel: Gets label by name
- `getUserLabels()` → GmailLabel[]: Gets user-created labels
- `markMessageRead(message)` → GmailApp: Marks message as read
- `markThreadImportant(thread)` → GmailApp: Marks thread important
- `moveMessageToTrash(message)` → GmailApp: Moves message to trash
- `moveThreadToArchive(thread)` → GmailApp: Archives thread
- `search(query)` → GmailThread[]: Searches Gmail
- `search(query, start, max)` → GmailThread[]: Searches with range
- `sendEmail(recipient, subject, body)` → GmailApp: Sends email
- `sendEmail(recipient, subject, body, options)` → GmailApp: Sends email with options

### CLASS: GmailThread
**METHODS:**
- `addLabel(label)` → GmailThread: Adds label to thread
- `createDraftReply(body)` → GmailDraft: Creates reply draft
- `createDraftReplyAll(body)` → GmailDraft: Creates reply-all draft
- `getFirstMessageSubject()` → String: Gets first message subject
- `getId()` → String: Gets thread ID
- `getLabels()` → GmailLabel[]: Gets thread labels
- `getLastMessageDate()` → Date: Gets last message date
- `getMessageCount()` → Integer: Gets message count
- `getMessages()` → GmailMessage[]: Gets thread messages
- `getPermalink()` → String: Gets thread permalink
- `hasStarredMessages()` → Boolean: Checks for starred messages
- `isImportant()` → Boolean: Checks if important
- `isInInbox()` → Boolean: Checks if in inbox
- `isInSpam()` → Boolean: Checks if spam
- `isInTrash()` → Boolean: Checks if in trash
- `isUnread()` → Boolean: Checks if unread
- `markImportant()` → GmailThread: Marks as important
- `markRead()` → GmailThread: Marks as read
- `markUnread()` → GmailThread: Marks as unread
- `moveToArchive()` → GmailThread: Archives thread
- `moveToInbox()` → GmailThread: Moves to inbox
- `moveToSpam()` → GmailThread: Marks as spam
- `moveToTrash()` → GmailThread: Moves to trash
- `refresh()` → GmailThread: Refreshes thread state
- `removeLabel(label)` → GmailThread: Removes label
- `reply(body)` → GmailThread: Replies to thread
- `replyAll(body)` → GmailThread: Replies to all

### CLASS: GmailMessage
**METHODS:**
- `createDraftReply(body)` → GmailDraft: Creates reply draft
- `createDraftReplyAll(body)` → GmailDraft: Creates reply-all draft
- `forward(recipient)` → GmailMessage: Forwards message
- `forward(recipient, options)` → GmailMessage: Forwards with options
- `getAttachments()` → GmailAttachment[]: Gets attachments
- `getBcc()` → String: Gets BCC recipients
- `getBody()` → String: Gets HTML body
- `getCc()` → String: Gets CC recipients
- `getDate()` → Date: Gets message date
- `getFrom()` → String: Gets sender
- `getHeader(name)` → String: Gets header value
- `getId()` → String: Gets message ID
- `getPlainBody()` → String: Gets plain text body
- `getRawContent()` → String: Gets raw content
- `getReplyTo()` → String: Gets reply-to address
- `getSubject()` → String: Gets subject
- `getThread()` → GmailThread: Gets parent thread
- `getTo()` → String: Gets recipients
- `isDraft()` → Boolean: Checks if draft
- `isInInbox()` → Boolean: Checks if in inbox
- `isStarred()` → Boolean: Checks if starred
- `isUnread()` → Boolean: Checks if unread
- `markRead()` → GmailMessage: Marks as read
- `markUnread()` → GmailMessage: Marks as unread
- `moveToTrash()` → GmailMessage: Moves to trash
- `refresh()` → GmailMessage: Refreshes message state
- `reply(body)` → GmailMessage: Replies to message
- `replyAll(body)` → GmailMessage: Replies to all
- `star()` → GmailMessage: Stars message
- `unstar()` → GmailMessage: Unstars message

### CLASS: GmailLabel
**METHODS:**
- `addToThread(thread)` → GmailLabel: Adds label to thread
- `addToThreads(threads)` → GmailLabel: Adds label to threads
- `deleteLabel()` → void: Deletes label
- `getName()` → String: Gets label name
- `getThreads()` → GmailThread[]: Gets labeled threads
- `getThreads(start, max)` → GmailThread[]: Gets range of threads
- `getUnreadCount()` → Integer: Gets unread count
- `removeFromThread(thread)` → GmailLabel: Removes from thread
- `removeFromThreads(threads)` → GmailLabel: Removes from threads

### CLASS: GmailDraft
**METHODS:**
- `deleteDraft()` → void: Deletes draft
- `getId()` → String: Gets draft ID
- `getMessage()` → GmailMessage: Gets draft as message
- `getMessageId()` → String: Gets message ID
- `send()` → GmailMessage: Sends draft
- `update(recipient, subject, body)` → GmailDraft: Updates draft
- `update(recipient, subject, body, options)` → GmailDraft: Updates with options

### CLASS: GmailAttachment
**METHODS:**
- `copyBlob()` → Blob: Copies blob
- `getAs(contentType)` → Blob: Gets as specific type
- `getBytes()` → Byte[]: Gets raw bytes
- `getContentType()` → String: Gets content type
- `getDataAsString()` → String: Gets as UTF-8 string
- `getHash()` → String: Gets SHA1 hash
- `getName()` → String: Gets attachment name
- `getSize()` → Integer: Gets size
- `isGoogleType()` → Boolean: Checks if Google Workspace file

## Common Patterns

### SEARCH_AND_PROCESS:
```javascript
var threads = GmailApp.search('is:unread');
for (var i = 0; i < threads.length; i++) {
  var messages = threads[i].getMessages();
  // Process messages
}
```

### SEND_EMAIL:
```javascript
GmailApp.sendEmail('recipient@example.com', 'Subject', 'Body', {
  cc: 'cc@example.com',
  bcc: 'bcc@example.com',
  attachments: [blob],
  htmlBody: '<h1>HTML Body</h1>'
});
```

### CREATE_DRAFT:
```javascript
var draft = GmailApp.createDraft(
  'recipient@example.com',
  'Draft Subject',
  'Draft body text'
);
```

### MANAGE_LABELS:
```javascript
var label = GmailApp.createLabel('My Label');
var thread = GmailApp.getThreadById('THREAD_ID');
thread.addLabel(label);
```

### GET_ATTACHMENTS:
```javascript
var thread = GmailApp.getInboxThreads(0, 1)[0];
var message = thread.getMessages()[0];
var attachments = message.getAttachments();
```

### MARK_AS_READ:
```javascript
var threads = GmailApp.getInboxThreads(0, 10);
GmailApp.markThreadsRead(threads);
```

### SEARCH_WITH_QUERY:
```javascript
var threads = GmailApp.search('from:sender@example.com subject:"Important"');
```

### FORWARD_MESSAGE:
```javascript
var message = GmailApp.getMessageById('MESSAGE_ID');
message.forward('newrecipient@example.com', {
  cc: 'cc@example.com',
  body: 'FYI - forwarding this message'
});
```