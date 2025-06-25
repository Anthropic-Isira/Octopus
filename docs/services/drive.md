# Google Drive Service API Reference

[← Back to Main Documentation](../README.md) | [View Examples →](../examples/drive-examples.md)

## Service Overview
**SERVICE_NAME:** DriveApp  
**DESCRIPTION:** Allows scripts to create, find, and modify files and folders in Google Drive  
**ACCESS_PATTERN:** `DriveApp.methodName()`

## Important Notes
- Google Workspace Administrators can turn off the Drive SDK for their domain
- Scripts using standard Cloud projects must manually turn on the Drive API
- For advanced features and shared drive access, use the Advanced Drive Service

## Core Classes

### CLASS: DriveApp
**PROPERTIES:**
- `Access` → Access: Enum for user access classes
- `Permission` → Permission: Enum for user permissions

**METHODS:**
- `createFile(blob)` → File: Creates file from Blob in root
- `createFile(name, content)` → File: Creates text file in root
- `createFile(name, content, mimeType)` → File: Creates file with MIME type
- `createFolder(name)` → Folder: Creates folder in root
- `createShortcut(targetId)` → File: Creates shortcut to Drive item
- `getFileById(id)` → File: Gets file by ID
- `getFiles()` → FileIterator: Gets all files in Drive
- `getFilesByName(name)` → FileIterator: Gets files by name
- `getFilesByType(mimeType)` → FileIterator: Gets files by MIME type
- `getFolderById(id)` → Folder: Gets folder by ID
- `getFolders()` → FolderIterator: Gets all folders
- `getFoldersByName(name)` → FolderIterator: Gets folders by name
- `getRootFolder()` → Folder: Gets root folder
- `getStorageLimit()` → Integer: Gets storage limit in bytes
- `getStorageUsed()` → Integer: Gets storage used in bytes
- `getTrashedFiles()` → FileIterator: Gets trashed files
- `getTrashedFolders()` → FolderIterator: Gets trashed folders
- `searchFiles(params)` → FileIterator: Search files by criteria
- `searchFolders(params)` → FolderIterator: Search folders by criteria

### CLASS: File
**METHODS:**
- `addEditor(emailAddress)` → File: Adds editor
- `addViewer(emailAddress)` → File: Adds viewer
- `addCommenter(emailAddress)` → File: Adds commenter
- `getAccess(email)` → Permission: Gets user permission
- `getAs(contentType)` → Blob: Gets as blob with content type
- `getBlob()` → Blob: Gets as blob
- `getDateCreated()` → Date: Gets creation date
- `getDescription()` → String: Gets description
- `getDownloadUrl()` → String: Gets download URL
- `getId()` → String: Gets file ID
- `getLastUpdated()` → Date: Gets last update date
- `getMimeType()` → String: Gets MIME type
- `getName()` → String: Gets file name
- `getOwner()` → User: Gets file owner
- `getParents()` → FolderIterator: Gets parent folders
- `getSize()` → Integer: Gets file size in bytes
- `getUrl()` → String: Gets file URL
- `makeCopy()` → File: Creates copy
- `makeCopy(name)` → File: Creates copy with name
- `makeCopy(name, destination)` → File: Creates copy in destination
- `moveTo(destination)` → File: Moves to folder
- `removeEditor(emailAddress)` → File: Removes editor
- `setContent(content)` → File: Sets file content
- `setDescription(description)` → File: Sets description
- `setName(name)` → File: Sets name
- `setOwner(emailAddress)` → File: Changes owner
- `setSharing(accessType, permissionType)` → File: Sets sharing
- `setTrashed(trashed)` → File: Sets trash status

### CLASS: Folder
**METHODS:**
- `addEditor(emailAddress)` → Folder: Adds editor
- `addViewer(emailAddress)` → Folder: Adds viewer
- `createFile(blob)` → File: Creates file from blob
- `createFile(name, content)` → File: Creates text file
- `createFolder(name)` → Folder: Creates subfolder
- `getAccess(email)` → Permission: Gets user permission
- `getDateCreated()` → Date: Gets creation date
- `getDescription()` → String: Gets description
- `getFiles()` → FileIterator: Gets child files
- `getFilesByName(name)` → FileIterator: Gets files by name
- `getFilesByType(mimeType)` → FileIterator: Gets files by type
- `getFolders()` → FolderIterator: Gets child folders
- `getFoldersByName(name)` → FolderIterator: Gets folders by name
- `getId()` → String: Gets folder ID
- `getName()` → String: Gets folder name
- `getOwner()` → User: Gets folder owner
- `getParents()` → FolderIterator: Gets parent folders
- `getUrl()` → String: Gets folder URL
- `moveTo(destination)` → Folder: Moves to folder
- `removeEditor(emailAddress)` → Folder: Removes editor
- `searchFiles(params)` → FileIterator: Search child files
- `searchFolders(params)` → FolderIterator: Search child folders
- `setName(name)` → Folder: Sets name
- `setOwner(emailAddress)` → Folder: Changes owner
- `setSharing(accessType, permissionType)` → Folder: Sets sharing
- `setTrashed(trashed)` → Folder: Sets trash status

### CLASS: FileIterator
**METHODS:**
- `getContinuationToken()` → String: Gets continuation token
- `hasNext()` → Boolean: Checks if has next item
- `next()` → File: Gets next file

### CLASS: FolderIterator
**METHODS:**
- `getContinuationToken()` → String: Gets continuation token
- `hasNext()` → Boolean: Checks if has next item
- `next()` → Folder: Gets next folder

### CLASS: User
**METHODS:**
- `getDomain()` → String: Gets user domain
- `getEmail()` → String: Gets user email
- `getName()` → String: Gets user name
- `getPhotoUrl()` → String: Gets user photo URL

## Enumerations

### ENUM: Access
**VALUES:**
- **ANYONE**: Anyone on Internet can find and access
- **ANYONE_WITH_LINK**: Anyone with link can access
- **DOMAIN**: People in domain can find and access
- **DOMAIN_WITH_LINK**: People in domain with link can access
- **PRIVATE**: Only explicitly granted users can access

### ENUM: Permission
**VALUES:**
- **VIEW**: Can only view or copy
- **EDIT**: Can edit
- **COMMENT**: Can view, copy, or comment
- **OWNER**: Owns the file/folder
- **ORGANIZER**: Can organize in shared drive
- **FILE_ORGANIZER**: Can edit, trash, move in shared drive
- **NONE**: No permissions

## Common Patterns

### LOG_ALL_FILES:
```javascript
var files = DriveApp.getFiles();
while (files.hasNext()) {
  var file = files.next();
  console.log(file.getName());
}
```

### CREATE_TEXT_FILE:
```javascript
var file = DriveApp.createFile('MyFile.txt', 'File content here');
```

### GET_FILE_BY_ID:
```javascript
var file = DriveApp.getFileById('FILE_ID_HERE');
```

### CREATE_FOLDER:
```javascript
var folder = DriveApp.createFolder('New Folder');
```

### SEARCH_FILES:
```javascript
var files = DriveApp.searchFiles('title contains "Report"');
```

### COPY_FILE:
```javascript
var original = DriveApp.getFileById('FILE_ID');
var copy = original.makeCopy('Copy of ' + original.getName());
```

### SHARE_FILE:
```javascript
file.addEditor('user@example.com');
file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
```

### MOVE_FILE:
```javascript
var file = DriveApp.getFileById('FILE_ID');
var folder = DriveApp.getFolderById('FOLDER_ID');
file.moveTo(folder);
```