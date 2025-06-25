# Google Docs Service Examples

[← Back to Docs API](../services/docs.md) | [← Back to Main Documentation](../README.md)

## Basic Operations

### Creating and Opening Documents

```javascript
// Create a new document
var doc = DocumentApp.create('My New Document');
console.log('Created document with ID: ' + doc.getId());

// Open an existing document by ID
var existingDoc = DocumentApp.openById('DOCUMENT_ID_HERE');

// Open by URL
var docByUrl = DocumentApp.openByUrl('https://docs.google.com/document/d/DOCUMENT_ID/edit');

// Get the active document (when running as a bound script)
var activeDoc = DocumentApp.getActiveDocument();
```

### Working with Document Content

```javascript
// Get the document body
var body = doc.getBody();

// Add basic content
body.appendParagraph('This is a new paragraph.');
body.appendParagraph('This is another paragraph with different formatting.');

// Add a heading
var heading = body.appendParagraph('Chapter 1: Introduction');
heading.setHeading(DocumentApp.ParagraphHeading.HEADING1);

// Add a list
body.appendListItem('First item').setGlyphType(DocumentApp.GlyphType.BULLET);
body.appendListItem('Second item').setGlyphType(DocumentApp.GlyphType.BULLET);
body.appendListItem('Third item').setGlyphType(DocumentApp.GlyphType.BULLET);
```

### Text Formatting

```javascript
// Create and format text
var paragraph = body.appendParagraph('This text will be formatted.');
var text = paragraph.editAsText();

// Apply formatting
text.setBold(0, 4, true);  // "This" will be bold
text.setItalic(5, 8, true);  // "text" will be italic
text.setFontSize(10, 17, 18);  // "will be" will be size 18
text.setForegroundColor(19, 28, '#FF0000');  // "formatted" will be red

// Set paragraph alignment
paragraph.setAlignment(DocumentApp.HorizontalAlignment.CENTER);

// Set line spacing
paragraph.setLineSpacing(1.5);
```

## Advanced Examples

### Creating Tables

```javascript
// Create a table with 3 rows and 2 columns
var table = body.appendTable();

// Add header row
var headerRow = table.appendTableRow();
headerRow.appendTableCell('Name');
headerRow.appendTableCell('Value');

// Add data rows
var row1 = table.appendTableRow();
row1.appendTableCell('Temperature');
row1.appendTableCell('72°F');

var row2 = table.appendTableRow();
row2.appendTableCell('Humidity');
row2.appendTableCell('45%');

// Format the table
table.setBorderColor('#000000');
table.setBorderWidth(1);

// Style header row
var headerCells = headerRow.getNumCells();
for (var i = 0; i < headerCells; i++) {
  headerRow.getCell(i)
    .setBackgroundColor('#4285F4')
    .editAsText()
    .setForegroundColor('#FFFFFF')
    .setBold(true);
}
```

### Working with Images

```javascript
// Insert an image from URL
var image = body.appendImage(UrlFetchApp.fetch('https://example.com/image.png').getBlob());

// Set image properties
image.setWidth(300);
image.setHeight(200);

// Insert an image from Drive
var driveImage = DriveApp.getFileById('IMAGE_FILE_ID').getBlob();
body.appendImage(driveImage);
```

### Document Manipulation

```javascript
// Find and replace text
body.replaceText('oldText', 'newText');

// Find and replace with regular expressions
body.replaceText('\\b[0-9]{3}-[0-9]{3}-[0-9]{4}\\b', 'XXX-XXX-XXXX');  // Replace phone numbers

// Get all paragraphs and process them
var paragraphs = body.getParagraphs();
paragraphs.forEach(function(para) {
  var text = para.getText();
  if (text.includes('important')) {
    para.setAttributes({
      BOLD: true,
      FOREGROUND_COLOR: '#FF0000'
    });
  }
});
```

## Practical Use Cases

### Document Template System

```javascript
function createFromTemplate(templateId, replacements) {
  // Copy the template
  var templateDoc = DocumentApp.openById(templateId);
  var newDoc = DocumentApp.create('Generated Document - ' + new Date());
  
  var templateBody = templateDoc.getBody();
  var newBody = newDoc.getBody();
  
  // Copy all elements
  var totalElements = templateBody.getNumChildren();
  for (var i = 0; i < totalElements; i++) {
    var element = templateBody.getChild(i).copy();
    newBody.appendParagraph(element.getText());
  }
  
  // Replace placeholders
  for (var key in replacements) {
    newBody.replaceText('{{' + key + '}}', replacements[key]);
  }
  
  return newDoc;
}

// Usage
var doc = createFromTemplate('TEMPLATE_ID', {
  name: 'John Doe',
  date: new Date().toLocaleDateString(),
  amount: '$1,500.00'
});
```

### Report Generator

```javascript
function generateReport(data) {
  var doc = DocumentApp.create('Monthly Report - ' + new Date().toLocaleDateString());
  var body = doc.getBody();
  
  // Add title
  var title = body.appendParagraph('Monthly Sales Report');
  title.setHeading(DocumentApp.ParagraphHeading.TITLE);
  title.setAlignment(DocumentApp.HorizontalAlignment.CENTER);
  
  // Add summary section
  body.appendParagraph('Executive Summary')
    .setHeading(DocumentApp.ParagraphHeading.HEADING1);
  
  body.appendParagraph('Total Sales: $' + data.totalSales.toLocaleString());
  body.appendParagraph('Total Units: ' + data.totalUnits.toLocaleString());
  
  // Add data table
  body.appendParagraph('Sales by Region')
    .setHeading(DocumentApp.ParagraphHeading.HEADING2);
  
  var table = body.appendTable();
  var headerRow = table.appendTableRow();
  headerRow.appendTableCell('Region').setBackgroundColor('#EEEEEE');
  headerRow.appendTableCell('Sales').setBackgroundColor('#EEEEEE');
  headerRow.appendTableCell('Units').setBackgroundColor('#EEEEEE');
  
  data.regions.forEach(function(region) {
    var row = table.appendTableRow();
    row.appendTableCell(region.name);
    row.appendTableCell('$' + region.sales.toLocaleString());
    row.appendTableCell(region.units.toLocaleString());
  });
  
  // Add chart placeholder
  body.appendPageBreak();
  body.appendParagraph('[Chart: Sales Trend]')
    .setAlignment(DocumentApp.HorizontalAlignment.CENTER);
  
  return doc;
}
```

### Form Letter Generator

```javascript
function generateFormLetters(recipients) {
  recipients.forEach(function(recipient) {
    var doc = DocumentApp.create('Letter to ' + recipient.name);
    var body = doc.getBody();
    
    // Add date
    body.appendParagraph(new Date().toLocaleDateString())
      .setAlignment(DocumentApp.HorizontalAlignment.RIGHT);
    
    // Add recipient address
    body.appendParagraph(recipient.name);
    body.appendParagraph(recipient.address);
    body.appendParagraph(recipient.city + ', ' + recipient.state + ' ' + recipient.zip);
    
    body.appendParagraph(''); // Blank line
    
    // Add salutation
    body.appendParagraph('Dear ' + recipient.name + ',');
    
    // Add body text
    var letterBody = body.appendParagraph(
      'We are pleased to inform you that your application has been approved. ' +
      'Your account number is ' + recipient.accountNumber + '.'
    );
    letterBody.setSpacingAfter(10);
    
    // Add signature
    body.appendParagraph('Sincerely,');
    body.appendParagraph('John Smith');
    body.appendParagraph('Account Manager');
    
    // Save and share
    doc.saveAndClose();
    doc.addViewer(recipient.email);
  });
}
```

### Document Analyzer

```javascript
function analyzeDocument(docId) {
  var doc = DocumentApp.openById(docId);
  var body = doc.getBody();
  var text = body.getText();
  
  var analysis = {
    wordCount: text.split(/\s+/).length,
    characterCount: text.length,
    paragraphCount: body.getParagraphs().length,
    tableCount: body.getTables().length,
    imageCount: body.getImages().length,
    headings: []
  };
  
  // Find all headings
  var paragraphs = body.getParagraphs();
  paragraphs.forEach(function(para) {
    var heading = para.getHeading();
    if (heading !== DocumentApp.ParagraphHeading.NORMAL) {
      analysis.headings.push({
        text: para.getText(),
        level: heading.toString()
      });
    }
  });
  
  return analysis;
}
```

## Advanced Formatting

### Custom Styles

```javascript
function applyCustomStyles(doc) {
  var body = doc.getBody();
  
  // Define custom styles
  var styles = {
    code: {
      FONT_FAMILY: 'Courier New',
      FONT_SIZE: 10,
      BACKGROUND_COLOR: '#F5F5F5',
      FOREGROUND_COLOR: '#333333'
    },
    highlight: {
      BACKGROUND_COLOR: '#FFFF00',
      BOLD: true
    },
    quote: {
      ITALIC: true,
      INDENT_START: 36,
      FOREGROUND_COLOR: '#666666'
    }
  };
  
  // Apply styles to specific text patterns
  var paragraphs = body.getParagraphs();
  paragraphs.forEach(function(para) {
    var text = para.getText();
    
    // Apply code style to text between backticks
    var codePattern = /`([^`]+)`/g;
    var match;
    while ((match = codePattern.exec(text)) !== null) {
      para.editAsText().setAttributes(
        match.index,
        match.index + match[0].length - 1,
        styles.code
      );
    }
    
    // Apply quote style to lines starting with ">"
    if (text.startsWith('>')) {
      para.setAttributes(styles.quote);
    }
  });
}
```

### Creating a Table of Contents

```javascript
function createTableOfContents(doc) {
  var body = doc.getBody();
  var paragraphs = body.getParagraphs();
  var toc = [];
  
  // Find all headings
  paragraphs.forEach(function(para, index) {
    var heading = para.getHeading();
    if (heading !== DocumentApp.ParagraphHeading.NORMAL) {
      toc.push({
        text: para.getText(),
        level: heading,
        index: index
      });
    }
  });
  
  // Insert TOC at the beginning
  body.insertParagraph(0, 'Table of Contents')
    .setHeading(DocumentApp.ParagraphHeading.HEADING1);
  
  toc.forEach(function(item, i) {
    var indent = '';
    if (item.level === DocumentApp.ParagraphHeading.HEADING2) {
      indent = '  ';
    } else if (item.level === DocumentApp.ParagraphHeading.HEADING3) {
      indent = '    ';
    }
    
    body.insertParagraph(i + 1, indent + item.text);
  });
  
  // Add a page break after TOC
  body.insertPageBreak(toc.length + 1);
}
```

## Error Handling

```javascript
function safeDocumentOperation(docId, operation) {
  try {
    var doc = DocumentApp.openById(docId);
    return operation(doc);
  } catch (e) {
    console.error('Document operation failed: ' + e.toString());
    
    if (e.toString().includes('No item with the given ID')) {
      console.error('Document not found. Please check the document ID.');
    } else if (e.toString().includes('does not have permission')) {
      console.error('You do not have permission to access this document.');
    }
    
    return null;
  }
}

// Usage
var result = safeDocumentOperation('DOCUMENT_ID', function(doc) {
  var body = doc.getBody();
  body.appendParagraph('New content');
  return doc.getName();
});
```