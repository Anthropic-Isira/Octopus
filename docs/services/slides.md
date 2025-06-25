# Google Slides Service API Reference

[← Back to Main Documentation](../README.md) | [View Examples →](../examples/slides-examples.md)

## Service Overview
**SERVICE_NAME:** SlidesApp  
**DESCRIPTION:** Creates and opens Presentations that can be edited. Allows scripts to create, access, and modify Google Slides files  
**ACCESS_PATTERN:** `SlidesApp.methodName()`

## Core Classes

### CLASS: SlidesApp
**PROPERTIES:**
- `AlignmentPosition` → AlignmentPosition: Enum for alignment positions
- `ShapeType` → ShapeType: Enum for shape types
- `PredefinedLayout` → PredefinedLayout: Enum for predefined layouts

**METHODS:**
- `create(name)` → Presentation: Creates new presentation
- `getActivePresentation()` → Presentation: Gets active presentation
- `getUi()` → Ui: Gets UI environment
- `openById(id)` → Presentation: Opens by ID
- `openByUrl(url)` → Presentation: Opens by URL
- `newAffineTransformBuilder()` → AffineTransformBuilder: Creates transform builder

### CLASS: Presentation
**METHODS:**
- `addEditor(emailAddress)` → Presentation: Adds editor
- `addViewer(emailAddress)` → Presentation: Adds viewer
- `appendSlide()` → Slide: Appends blank slide
- `appendSlide(layout)` → Slide: Appends slide with layout
- `appendSlide(predefinedLayout)` → Slide: Appends with predefined layout
- `getId()` → String: Gets presentation ID
- `getLayouts()` → Layout[]: Gets layouts
- `getMasters()` → Master[]: Gets masters
- `getName()` → String: Gets presentation name
- `getPageHeight()` → Number: Gets page height in points
- `getPageWidth()` → Number: Gets page width in points
- `getSelection()` → Selection: Gets current selection
- `getSlideById(id)` → Slide: Gets slide by ID
- `getSlides()` → Slide[]: Gets all slides
- `getUrl()` → String: Gets presentation URL
- `insertSlide(insertionIndex)` → Slide: Inserts slide at index
- `removeEditor(emailAddress)` → Presentation: Removes editor
- `replaceAllText(findText, replaceText)` → Integer: Replaces all text
- `saveAndClose()` → void: Saves and closes
- `setName(name)` → void: Sets presentation name

### CLASS: Slide
**METHODS:**
- `duplicate()` → Slide: Duplicates slide
- `getBackground()` → PageBackground: Gets background
- `getColorScheme()` → ColorScheme: Gets color scheme
- `getGroups()` → Group[]: Gets groups
- `getImages()` → Image[]: Gets images
- `getLayout()` → Layout: Gets slide layout
- `getNotesPage()` → NotesPage: Gets notes page
- `getObjectId()` → String: Gets unique ID
- `getPageElements()` → PageElement[]: Gets all page elements
- `getShapes()` → Shape[]: Gets shapes
- `getTables()` → Table[]: Gets tables
- `getVideos()` → Video[]: Gets videos
- `insertImage(imageUrl)` → Image: Inserts image from URL
- `insertShape(shapeType)` → Shape: Inserts shape
- `insertTable(numRows, numColumns)` → Table: Inserts table
- `insertTextBox(text)` → Shape: Inserts text box
- `insertVideo(videoUrl)` → Video: Inserts video
- `move(index)` → void: Moves slide to index
- `remove()` → void: Removes slide
- `replaceAllText(findText, replaceText)` → Integer: Replaces text

### CLASS: PageElement
**METHODS:**
- `alignOnPage(alignmentPosition)` → PageElement: Aligns element
- `asGroup()` → Group: Returns as Group
- `asImage()` → Image: Returns as Image
- `asLine()` → Line: Returns as Line
- `asShape()` → Shape: Returns as Shape
- `asTable()` → Table: Returns as Table
- `asVideo()` → Video: Returns as Video
- `bringForward()` → PageElement: Brings forward
- `bringToFront()` → PageElement: Brings to front
- `duplicate()` → PageElement: Duplicates element
- `getHeight()` → Number: Gets height in points
- `getLeft()` → Number: Gets left position
- `getObjectId()` → String: Gets unique ID
- `getPageElementType()` → PageElementType: Gets element type
- `getRotation()` → Number: Gets rotation angle
- `getTop()` → Number: Gets top position
- `getTransform()` → AffineTransform: Gets transform
- `getWidth()` → Number: Gets width in points
- `remove()` → void: Removes element
- `scaleHeight(ratio)` → PageElement: Scales height
- `scaleWidth(ratio)` → PageElement: Scales width
- `select()` → void: Selects element
- `sendBackward()` → PageElement: Sends backward
- `sendToBack()` → PageElement: Sends to back
- `setHeight(height)` → PageElement: Sets height
- `setLeft(left)` → PageElement: Sets left position
- `setRotation(angle)` → PageElement: Sets rotation
- `setTop(top)` → PageElement: Sets top position
- `setWidth(width)` → PageElement: Sets width

### CLASS: Shape
**METHODS:**
- `getAutofit()` → Autofit: Gets autofit settings
- `getBorder()` → Border: Gets border
- `getContentAlignment()` → ContentAlignment: Gets text alignment
- `getFill()` → Fill: Gets fill
- `getLink()` → Link: Gets link
- `getPlaceholderType()` → PlaceholderType: Gets placeholder type
- `getShapeType()` → ShapeType: Gets shape type
- `getText()` → TextRange: Gets text content
- `removeLink()` → void: Removes link
- `replaceWithImage(imageUrl)` → Image: Replaces with image
- `setContentAlignment(alignment)` → Shape: Sets text alignment
- `setLinkSlide(slideIndex)` → Link: Sets link to slide
- `setLinkUrl(url)` → Link: Sets link URL

### CLASS: TextRange
**METHODS:**
- `appendParagraph(text)` → Paragraph: Appends paragraph
- `appendText(text)` → TextRange: Appends text
- `clear()` → void: Clears text
- `getEndIndex()` → Integer: Gets end index
- `getLength()` → Integer: Gets character count
- `getListStyle()` → ListStyle: Gets list style
- `getParagraphStyle()` → ParagraphStyle: Gets paragraph style
- `getStartIndex()` → Integer: Gets start index
- `getTextStyle()` → TextStyle: Gets text style
- `insertText(startOffset, text)` → TextRange: Inserts text
- `replaceAllText(findText, replaceText)` → Integer: Replaces text
- `select()` → void: Selects text
- `setText(newText)` → TextRange: Sets text

### CLASS: Table
**METHODS:**
- `appendColumn()` → TableColumn: Appends column
- `appendRow()` → TableRow: Appends row
- `getCell(rowIndex, columnIndex)` → TableCell: Gets cell
- `getColumn(columnIndex)` → TableColumn: Gets column
- `getNumColumns()` → Integer: Gets column count
- `getNumRows()` → Integer: Gets row count
- `getRow(rowIndex)` → TableRow: Gets row
- `insertColumn(index)` → TableColumn: Inserts column
- `insertRow(index)` → TableRow: Inserts row

### CLASS: Image
**METHODS:**
- `getAs(contentType)` → Blob: Gets as blob
- `getBlob()` → Blob: Gets image blob
- `getBorder()` → Border: Gets border
- `getContentUrl()` → String: Gets content URL
- `getLink()` → Link: Gets link
- `getSourceUrl()` → String: Gets source URL
- `replace(imageUrl)` → Image: Replaces image
- `setLinkSlide(slideIndex)` → Link: Sets link to slide
- `setLinkUrl(url)` → Link: Sets link URL

## Enumerations

### ENUM: ShapeType
**VALUES:**
- **TEXT_BOX**: Text box shape
- **RECTANGLE**: Rectangle shape
- **ROUND_RECTANGLE**: Rounded rectangle
- **ELLIPSE**: Ellipse shape
- **ARC**: Arc shape
- **BENT_ARROW**: Bent arrow
- **BLOCK_ARC**: Block arc
- **BRACE_PAIR**: Brace pair
- **CAN**: Can shape
- **CHEVRON**: Chevron shape
- **CLOUD**: Cloud shape
- **CORNER**: Corner shape
- **CUBE**: Cube shape
- **DIAMOND**: Diamond shape
- **DONUT**: Donut shape
- **HEART**: Heart shape
- **HEXAGON**: Hexagon shape
- **HOME_PLATE**: Home plate shape
- **OCTAGON**: Octagon shape
- **PARALLELOGRAM**: Parallelogram
- **PENTAGON**: Pentagon shape
- **PIE**: Pie shape
- **PLAQUE**: Plaque shape
- **PLUS**: Plus shape
- **RIGHT_ARROW**: Right arrow
- **STAR_5**: Five pointed star
- **TRAPEZOID**: Trapezoid shape
- **TRIANGLE**: Triangle shape

### ENUM: PredefinedLayout
**VALUES:**
- **BLANK**: Blank layout
- **CAPTION_ONLY**: Caption at bottom
- **TITLE**: Title and subtitle
- **TITLE_AND_BODY**: Title and body
- **TITLE_AND_TWO_COLUMNS**: Title and two columns
- **TITLE_ONLY**: Only title
- **SECTION_HEADER**: Section title
- **ONE_COLUMN_TEXT**: Single column text
- **MAIN_POINT**: Main point
- **BIG_NUMBER**: Big number heading

### ENUM: AlignmentPosition
**VALUES:**
- **CENTER**: Align to center
- **HORIZONTAL_CENTER**: Horizontal center
- **VERTICAL_CENTER**: Vertical center

### ENUM: PageElementType
**VALUES:**
- **SHAPE**: Generic shape
- **IMAGE**: Image
- **VIDEO**: Video
- **TABLE**: Table
- **GROUP**: Group of elements
- **LINE**: Line
- **WORD_ART**: Word art
- **SHEETS_CHART**: Sheets chart
- **SPEAKER_SPOTLIGHT**: Speaker spotlight

## Common Patterns

### CREATE_PRESENTATION:
```javascript
var presentation = SlidesApp.create('New Presentation');
```

### OPEN_PRESENTATION:
```javascript
var presentation = SlidesApp.openById('PRESENTATION_ID');
```

### ADD_SLIDE:
```javascript
var slide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE_AND_BODY);
```

### INSERT_SHAPE:
```javascript
var shape = slide.insertShape(SlidesApp.ShapeType.RECTANGLE);
shape.setLeft(100).setTop(100).setWidth(200).setHeight(100);
```

### INSERT_TEXT_BOX:
```javascript
var textBox = slide.insertTextBox('Hello World');
textBox.setLeft(50).setTop(50).setWidth(300).setHeight(60);
```

### INSERT_IMAGE:
```javascript
var image = slide.insertImage('https://example.com/image.png');
image.setLeft(100).setTop(100).setWidth(400).setHeight(300);
```

### INSERT_TABLE:
```javascript
var table = slide.insertTable(3, 4); // 3 rows, 4 columns
table.setLeft(50).setTop(50).setWidth(500).setHeight(200);
```

### FORMAT_TEXT:
```javascript
var textRange = shape.getText();
var textStyle = textRange.getTextStyle();
textStyle.setBold(true)
         .setFontSize(18)
         .setForegroundColor('#FF0000');
```

### REPLACE_TEXT:
```javascript
var numReplacements = presentation.replaceAllText('{{name}}', 'John Doe');
```

### ALIGN_ELEMENTS:
```javascript
shape.alignOnPage(SlidesApp.AlignmentPosition.CENTER);
```

### GROUP_ELEMENTS:
```javascript
var group = slide.group([shape1, shape2, shape3]);
```

### NAVIGATE_SLIDES:
```javascript
var slides = presentation.getSlides();
var firstSlide = slides[0];
var lastSlide = slides[slides.length - 1];
```