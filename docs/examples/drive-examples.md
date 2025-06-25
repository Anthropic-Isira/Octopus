# Google Drive Service Examples

[← Back to Drive API](../services/drive.md) | [← Back to Main Documentation](../README.md)

## Basic Operations

### Creating Files and Folders

```javascript
// Create a simple text file
var file = DriveApp.createFile('test.txt', 'Hello, World!');
console.log('Created file: ' + file.getId());

// Create a file with specific MIME type
var htmlFile = DriveApp.createFile('page.html', '<h1>Hello</h1>', MimeType.HTML);

// Create a folder
var folder = DriveApp.createFolder('My New Folder');

// Create a file in a specific folder
var subFile = folder.createFile('document.txt', 'Content in folder');
```

### Finding Files and Folders

```javascript
// Get all files
var files = DriveApp.getFiles();
while (files.hasNext()) {
  var file = files.next();
  console.log(file.getName() + ' - ' + file.getId());
}

// Search for files by name
var filesByName = DriveApp.getFilesByName('report.pdf');

// Search for files by MIME type
var spreadsheets = DriveApp.getFilesByType(MimeType.GOOGLE_SHEETS);

// Get a specific file by ID
var specificFile = DriveApp.getFileById('FILE_ID_HERE');

// Search with parameters
var searchResults = DriveApp.searchFiles('title contains "Project" and modifiedDate > "2024-01-01"');
```

### File Operations

```javascript
// Get file information
var file = DriveApp.getFileById('FILE_ID');
console.log('Name: ' + file.getName());
console.log('Size: ' + file.getSize() + ' bytes');
console.log('Created: ' + file.getDateCreated());
console.log('Last Updated: ' + file.getLastUpdated());
console.log('Owner: ' + file.getOwner().getEmail());

// Rename a file
file.setName('New File Name');

// Move a file to a folder
var destinationFolder = DriveApp.getFolderById('FOLDER_ID');
file.moveTo(destinationFolder);

// Copy a file
var copy = file.makeCopy('Copy of ' + file.getName());

// Delete a file (move to trash)
file.setTrashed(true);
```

## Advanced Examples

### File Sharing and Permissions

```javascript
// Share a file with specific users
var file = DriveApp.getFileById('FILE_ID');

// Add editor
file.addEditor('editor@example.com');

// Add viewer
file.addViewer('viewer@example.com');

// Add multiple users at once
file.addEditors(['user1@example.com', 'user2@example.com']);

// Set general sharing permissions
file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);

// Remove permissions
file.removeEditor('editor@example.com');

// Check user permissions
var permission = file.getAccess('user@example.com');
if (permission === DriveApp.Permission.EDIT) {
  console.log('User can edit');
}
```

### Working with Folders

```javascript
// Create nested folder structure
var rootFolder = DriveApp.createFolder('Project Root');
var docsFolder = rootFolder.createFolder('Documents');
var imagesFolder = rootFolder.createFolder('Images');
var archiveFolder = docsFolder.createFolder('Archive');

// Get folder contents
var files = docsFolder.getFiles();
while (files.hasNext()) {
  var file = files.next();
  console.log('File: ' + file.getName());
}

var subfolders = rootFolder.getFolders();
while (subfolders.hasNext()) {
  var folder = subfolders.next();
  console.log('Subfolder: ' + folder.getName());
}

// Search within a folder
var folderSearchResults = docsFolder.searchFiles('title contains "Report"');
```

### File Content Manipulation

```javascript
// Read and update text file content
var file = DriveApp.getFileById('TEXT_FILE_ID');
var content = file.getBlob().getDataAsString();
console.log('Current content: ' + content);

// Update content
var newContent = content + '\nAppended line';
file.setContent(newContent);

// Create a file from a blob
var blob = Utilities.newBlob('CSV data here', 'text/csv', 'data.csv');
var csvFile = DriveApp.createFile(blob);

// Convert file types
var doc = DocumentApp.create('Test Doc');
var docFile = DriveApp.getFileById(doc.getId());
var pdfBlob = docFile.getAs('application/pdf');
var pdfFile = DriveApp.createFile(pdfBlob).setName('Test Doc.pdf');
```

## Practical Use Cases

### Backup System

```javascript
function backupFolderToArchive(sourceFolderId, archiveFolderId) {
  var sourceFolder = DriveApp.getFolderById(sourceFolderId);
  var archiveFolder = DriveApp.getFolderById(archiveFolderId);
  
  // Create dated backup folder
  var backupDate = new Date().toISOString().split('T')[0];
  var backupFolder = archiveFolder.createFolder('Backup_' + backupDate);
  
  // Copy all files
  var files = sourceFolder.getFiles();
  while (files.hasNext()) {
    var file = files.next();
    file.makeCopy(file.getName(), backupFolder);
  }
  
  // Recursively copy subfolders
  var folders = sourceFolder.getFolders();
  while (folders.hasNext()) {
    var folder = folders.next();
    var newSubFolder = backupFolder.createFolder(folder.getName());
    copyFolderContents(folder.getId(), newSubFolder.getId());
  }
  
  return backupFolder.getId();
}

function copyFolderContents(sourceFolderId, destFolderId) {
  var sourceFolder = DriveApp.getFolderById(sourceFolderId);
  var destFolder = DriveApp.getFolderById(destFolderId);
  
  var files = sourceFolder.getFiles();
  while (files.hasNext()) {
    var file = files.next();
    file.makeCopy(file.getName(), destFolder);
  }
}
```

### File Organization System

```javascript
function organizeFilesByType(rootFolderId) {
  var rootFolder = DriveApp.getFolderById(rootFolderId);
  
  // Create folders for different file types
  var folders = {
    'Documents': rootFolder.createFolder('Documents'),
    'Spreadsheets': rootFolder.createFolder('Spreadsheets'),
    'Presentations': rootFolder.createFolder('Presentations'),
    'Images': rootFolder.createFolder('Images'),
    'PDFs': rootFolder.createFolder('PDFs'),
    'Others': rootFolder.createFolder('Others')
  };
  
  // Get all files in root folder
  var files = rootFolder.getFiles();
  
  while (files.hasNext()) {
    var file = files.next();
    var mimeType = file.getMimeType();
    var targetFolder;
    
    // Determine target folder based on MIME type
    if (mimeType === MimeType.GOOGLE_DOCS) {
      targetFolder = folders['Documents'];
    } else if (mimeType === MimeType.GOOGLE_SHEETS) {
      targetFolder = folders['Spreadsheets'];
    } else if (mimeType === MimeType.GOOGLE_SLIDES) {
      targetFolder = folders['Presentations'];
    } else if (mimeType.startsWith('image/')) {
      targetFolder = folders['Images'];
    } else if (mimeType === MimeType.PDF) {
      targetFolder = folders['PDFs'];
    } else {
      targetFolder = folders['Others'];
    }
    
    // Move file to appropriate folder
    file.moveTo(targetFolder);
  }
}
```

### Storage Usage Report

```javascript
function generateStorageReport() {
  var report = {
    totalFiles: 0,
    totalSize: 0,
    filesByType: {},
    largestFiles: [],
    oldestFiles: [],
    recentlyModified: []
  };
  
  var files = DriveApp.getFiles();
  var allFiles = [];
  
  // Collect all file data
  while (files.hasNext()) {
    var file = files.next();
    var fileInfo = {
      name: file.getName(),
      id: file.getId(),
      size: file.getSize(),
      type: file.getMimeType(),
      created: file.getDateCreated(),
      modified: file.getLastUpdated()
    };
    
    allFiles.push(fileInfo);
    report.totalFiles++;
    report.totalSize += fileInfo.size;
    
    // Count by type
    report.filesByType[fileInfo.type] = (report.filesByType[fileInfo.type] || 0) + 1;
  }
  
  // Sort and get top files
  allFiles.sort((a, b) => b.size - a.size);
  report.largestFiles = allFiles.slice(0, 10);
  
  allFiles.sort((a, b) => a.created - b.created);
  report.oldestFiles = allFiles.slice(0, 10);
  
  allFiles.sort((a, b) => b.modified - a.modified);
  report.recentlyModified = allFiles.slice(0, 10);
  
  // Convert size to readable format
  report.totalSizeFormatted = formatFileSize(report.totalSize);
  
  return report;
}

function formatFileSize(bytes) {
  if (bytes === 0) return '0 Bytes';
  var k = 1024;
  var sizes = ['Bytes', 'KB', 'MB', 'GB'];
  var i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
}
```

### Duplicate File Finder

```javascript
function findDuplicateFiles() {
  var files = DriveApp.getFiles();
  var fileMap = {};
  var duplicates = [];
  
  while (files.hasNext()) {
    var file = files.next();
    var key = file.getName() + '_' + file.getSize();
    
    if (fileMap[key]) {
      duplicates.push({
        name: file.getName(),
        size: file.getSize(),
        files: [fileMap[key], {
          id: file.getId(),
          created: file.getDateCreated(),
          modified: file.getLastUpdated(),
          owner: file.getOwner().getEmail()
        }]
      });
    } else {
      fileMap[key] = {
        id: file.getId(),
        created: file.getDateCreated(),
        modified: file.getLastUpdated(),
        owner: file.getOwner().getEmail()
      };
    }
  }
  
  return duplicates;
}
```

### Automated File Cleanup

```javascript
function cleanupOldFiles(daysToKeep) {
  var cutoffDate = new Date();
  cutoffDate.setDate(cutoffDate.getDate() - daysToKeep);
  
  var trashedCount = 0;
  var totalSize = 0;
  
  // Search for old files
  var searchQuery = 'modifiedDate < "' + cutoffDate.toISOString() + '"';
  var oldFiles = DriveApp.searchFiles(searchQuery);
  
  while (oldFiles.hasNext()) {
    var file = oldFiles.next();
    
    // Skip if file is already in trash
    if (file.isTrashed()) continue;
    
    // Skip if file is starred (important)
    if (file.isStarred()) continue;
    
    totalSize += file.getSize();
    file.setTrashed(true);
    trashedCount++;
  }
  
  return {
    filesDeleted: trashedCount,
    spaceFreed: formatFileSize(totalSize)
  };
}
```

### Shared Drive Management

```javascript
function auditSharedFiles() {
  var sharedFiles = [];
  var files = DriveApp.getFiles();
  
  while (files.hasNext()) {
    var file = files.next();
    
    // Check if file is shared
    var editors = file.getEditors();
    var viewers = file.getViewers();
    
    if (editors.length > 0 || viewers.length > 0) {
      sharedFiles.push({
        name: file.getName(),
        id: file.getId(),
        owner: file.getOwner().getEmail(),
        editors: editors.map(e => e.getEmail()),
        viewers: viewers.map(v => v.getEmail()),
        sharingAccess: file.getSharingAccess(),
        sharingPermission: file.getSharingPermission()
      });
    }
  }
  
  return sharedFiles;
}
```

## Error Handling

```javascript
function safeDriveOperation(operation) {
  try {
    return operation();
  } catch (e) {
    console.error('Drive operation failed: ' + e.toString());
    
    if (e.toString().includes('No item with the given ID')) {
      console.error('File or folder not found');
    } else if (e.toString().includes('does not have permission')) {
      console.error('Insufficient permissions');
    } else if (e.toString().includes('Storage quota exceeded')) {
      console.error('Drive storage full');
    }
    
    return null;
  }
}

// Usage
var result = safeDriveOperation(function() {
  var file = DriveApp.getFileById('FILE_ID');
  return file.getName();
});
```

## Best Practices

1. **Use Iterators Efficiently**: When working with large numbers of files, process them in batches:
```javascript
function processFilesInBatches(query, batchSize) {
  var files = DriveApp.searchFiles(query);
  var batch = [];
  
  while (files.hasNext() && batch.length < batchSize) {
    batch.push(files.next());
  }
  
  // Process batch
  batch.forEach(function(file) {
    // Process file
  });
  
  // Save continuation token if needed
  if (files.hasNext()) {
    var token = files.getContinuationToken();
    // Save token for next run
  }
}
```

2. **Check Permissions Before Operations**:
```javascript
function canUserEdit(fileId, userEmail) {
  var file = DriveApp.getFileById(fileId);
  var permission = file.getAccess(userEmail);
  return permission === DriveApp.Permission.EDIT || 
         permission === DriveApp.Permission.OWNER;
}
```

3. **Handle Large Files Carefully**:
```javascript
function processLargeFile(fileId) {
  var file = DriveApp.getFileById(fileId);
  
  // Check file size before processing
  if (file.getSize() > 50 * 1024 * 1024) { // 50MB
    console.log('File is large, processing in chunks...');
    // Implement chunked processing
  }
}
```