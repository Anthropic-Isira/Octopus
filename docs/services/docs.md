# Google Docs Service API Reference

[← Back to Main Documentation](../README.md) | [View Examples →](../examples/docs-examples.md)

## Service Overview
**SERVICE_NAME:** DocumentApp  
**DESCRIPTION:** The document service creates and opens Documents that can be edited  
**ACCESS_PATTERN:** `DocumentApp.methodName()`

## Core Classes

### CLASS: Document
**METHODS:**
- `create(name)` → Document: Creates and returns a new document
- `openById(id)` → Document: Returns the document with the specified ID
- `openByUrl(url)` → Document: Opens and returns the document with the specified URL
- `getActiveDocument()` → Document: Returns the document to which the script is container-bound
- `addEditor(emailAddress)` → Document: Adds user to editors list
- `addViewer(emailAddress)` → Document: Adds user to viewers list
- `getBody()` → Body: Retrieves the document body
- `getId()` → String: Retrieves the document's unique identifier
- `getName()` → String: Retrieves the title of the document
- `setName(name)` → Document: Sets the document title
- `saveAndClose()` → void: Saves the current Document

### CLASS: Body
**PARENT:** ContainerElement  
**METHODS:**
- `appendParagraph(text)` → Paragraph: Creates and appends new paragraph
- `appendTable()` → Table: Creates and appends new table
- `appendListItem(text)` → ListItem: Creates and appends new list item
- `appendImage(blob)` → InlineImage: Creates and appends new image
- `appendPageBreak()` → PageBreak: Creates and appends page break
- `appendHorizontalRule()` → HorizontalRule: Creates and appends horizontal rule
- `clear()` → Body: Clears the contents
- `copy()` → Body: Returns detached copy
- `getText()` → String: Retrieves contents as text string
- `setText(text)` → Body: Sets contents as plain text
- `replaceText(searchPattern, replacement)` → Element: Replaces text using regex

### CLASS: Paragraph
**PARENT:** ContainerElement  
**METHODS:**
- `appendText(text)` → Text: Creates and appends new text element
- `setText(text)` → void: Sets paragraph contents as text
- `getText()` → String: Gets paragraph text
- `setHeading(heading)` → Paragraph: Sets paragraph heading type
- `setAlignment(alignment)` → Paragraph: Sets horizontal alignment
- `setSpacingBefore(points)` → Paragraph: Sets spacing before element
- `setSpacingAfter(points)` → Paragraph: Sets spacing after element
- `setLineSpacing(multiplier)` → Paragraph: Sets line spacing
- `setIndentStart(points)` → Paragraph: Sets start indentation
- `setIndentEnd(points)` → Paragraph: Sets end indentation

### CLASS: Text
**METHODS:**
- `setText(text)` → Text: Sets text contents
- `getText()` → String: Retrieves text contents
- `appendText(text)` → Text: Adds text to end
- `deleteText(startOffset, endOffset)` → Text: Deletes range of text
- `insertText(offset, text)` → Text: Inserts text at offset
- `setBold(bold)` → Text: Sets bold setting
- `setItalic(italic)` → Text: Sets italic setting
- `setUnderline(underline)` → Text: Sets underline setting
- `setFontSize(size)` → Text: Sets font size in points
- `setForegroundColor(color)` → Text: Sets text color
- `setBackgroundColor(color)` → Text: Sets background color

### CLASS: Table
**PARENT:** ContainerElement  
**METHODS:**
- `appendTableRow()` → TableRow: Creates and appends new row
- `getCell(rowIndex, cellIndex)` → TableCell: Gets cell at indices
- `getNumRows()` → Integer: Gets number of rows
- `getRow(rowIndex)` → TableRow: Gets row at index
- `removeRow(rowIndex)` → TableRow: Removes row at index
- `setBorderColor(color)` → Table: Sets border color
- `setBorderWidth(width)` → Table: Sets border width in points

### CLASS: ListItem
**PARENT:** ContainerElement  
**METHODS:**
- `setText(text)` → void: Sets list item text
- `setGlyphType(glyphType)` → ListItem: Sets bullet/numbering type
- `setNestingLevel(level)` → ListItem: Sets nesting level
- `setListId(listId)` → ListItem: Associates with list ID

## Element Types
- BODY_SECTION
- PARAGRAPH
- TEXT
- TABLE
- TABLE_ROW
- TABLE_CELL
- LIST_ITEM
- INLINE_IMAGE
- HORIZONTAL_RULE
- PAGE_BREAK
- HEADER_SECTION
- FOOTER_SECTION
- FOOTNOTE

## Enumerations

### ENUM: ParagraphHeading
**VALUES:** NORMAL, HEADING1, HEADING2, HEADING3, HEADING4, HEADING5, HEADING6, TITLE, SUBTITLE

### ENUM: HorizontalAlignment
**VALUES:** LEFT, CENTER, RIGHT, JUSTIFY

### ENUM: GlyphType
**VALUES:** BULLET, HOLLOW_BULLET, SQUARE_BULLET, NUMBER, LATIN_UPPER, LATIN_LOWER, ROMAN_UPPER, ROMAN_LOWER

## Common Patterns

### CREATE_DOCUMENT:
```javascript
var doc = DocumentApp.create('Document Name');
```

### OPEN_DOCUMENT:
```javascript
var doc = DocumentApp.openById('DOCUMENT_ID');
```

### ADD_CONTENT:
```javascript
var body = doc.getBody();
body.appendParagraph('Hello World');
```

### FORMAT_TEXT:
```javascript
var text = paragraph.editAsText();
text.setBold(0, 5, true);
```

### CREATE_TABLE:
```javascript
var table = body.appendTable();
table.appendTableRow().appendTableCell('Cell Content');
```

### SEARCH_REPLACE:
```javascript
body.replaceText('oldText', 'newText');
```

### SET_ATTRIBUTES:
```javascript
element.setAttributes({BOLD: true, FONT_SIZE: 14});
```