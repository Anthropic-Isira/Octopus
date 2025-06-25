# Google Slides Service Examples

[← Back to Slides API](../services/slides.md) | [← Back to Main Documentation](../README.md)

## Basic Operations

### Creating and Opening Presentations

```javascript
// Create a new presentation
var presentation = SlidesApp.create('My New Presentation');
console.log('Created presentation with ID: ' + presentation.getId());

// Open existing presentation by ID
var existingPres = SlidesApp.openById('PRESENTATION_ID_HERE');

// Open by URL
var presByUrl = SlidesApp.openByUrl('https://docs.google.com/presentation/d/PRESENTATION_ID/edit');

// Get active presentation (when running as bound script)
var activePres = SlidesApp.getActivePresentation();
```

### Working with Slides

```javascript
// Append a blank slide
var slide = presentation.appendSlide();

// Append slide with predefined layout
var titleSlide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE);
var contentSlide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE_AND_BODY);

// Get all slides
var slides = presentation.getSlides();
console.log('Total slides: ' + slides.length);

// Get specific slide
var firstSlide = slides[0];

// Insert slide at specific position
var insertedSlide = presentation.insertSlide(2); // Insert at position 2

// Duplicate a slide
var duplicatedSlide = firstSlide.duplicate();

// Remove a slide
slides[slides.length - 1].remove();
```

### Adding Content to Slides

```javascript
// Add text box
var textBox = slide.insertTextBox('Hello World!', 100, 100, 300, 60);
textBox.getText().getTextStyle()
  .setFontSize(24)
  .setFontFamily('Arial')
  .setBold(true);

// Add shapes
var rectangle = slide.insertShape(SlidesApp.ShapeType.RECTANGLE, 50, 50, 200, 100);
rectangle.getFill().setSolidFill('#4285F4');

var circle = slide.insertShape(SlidesApp.ShapeType.ELLIPSE, 300, 50, 100, 100);
circle.getBorder().setWeight(3).getLineFill().setSolidFill('#EA4335');

// Add image
var image = slide.insertImage('https://example.com/image.png');
image.setLeft(100).setTop(200).setWidth(400).setHeight(300);

// Add table
var table = slide.insertTable(3, 4, 50, 50, 500, 200);
table.getCell(0, 0).getText().setText('Header 1');
```

## Advanced Examples

### Creating a Complete Presentation

```javascript
function createCompletePresentatation() {
  var presentation = SlidesApp.create('Company Overview');
  
  // Title Slide
  var titleSlide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE);
  var titleElements = titleSlide.getPageElements();
  titleElements[0].asShape().getText().setText('Company Overview');
  titleElements[1].asShape().getText().setText('2024 Annual Report');
  
  // Agenda Slide
  var agendaSlide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE_AND_BODY);
  var agendaShapes = agendaSlide.getShapes();
  agendaShapes[0].getText().setText('Agenda');
  agendaShapes[1].getText().setText(
    '• Company Performance\n' +
    '• Market Analysis\n' +
    '• Future Strategy\n' +
    '• Q&A Session'
  );
  
  // Data Slide with Chart
  var dataSlide = presentation.appendSlide(SlidesApp.PredefinedLayout.BLANK);
  var title = dataSlide.insertTextBox('Quarterly Revenue', 50, 20, 600, 50);
  title.getText().getTextStyle().setFontSize(32).setBold(true);
  
  // Create data table for chart visualization
  var dataTable = dataSlide.insertTable(5, 2, 50, 100, 300, 200);
  var tableData = [
    ['Quarter', 'Revenue'],
    ['Q1', '$1.2M'],
    ['Q2', '$1.5M'],
    ['Q3', '$1.8M'],
    ['Q4', '$2.1M']
  ];
  
  for (var i = 0; i < tableData.length; i++) {
    for (var j = 0; j < tableData[i].length; j++) {
      dataTable.getCell(i, j).getText().setText(tableData[i][j]);
    }
  }
  
  // Closing Slide
  var closingSlide = presentation.appendSlide(SlidesApp.PredefinedLayout.SECTION_HEADER);
  var closingShapes = closingSlide.getShapes();
  closingShapes[0].getText().setText('Thank You');
  closingShapes[1].getText().setText('Questions?');
  
  return presentation;
}
```

### Formatting and Styling

```javascript
function applyCustomFormatting(slide) {
  // Create a styled text box
  var textBox = slide.insertTextBox('Styled Text', 100, 100, 400, 100);
  var textRange = textBox.getText();
  
  // Apply paragraph styling
  var paragraphStyle = textRange.getParagraphStyle();
  paragraphStyle.setParagraphAlignment(SlidesApp.ParagraphAlignment.CENTER);
  paragraphStyle.setLineSpacing(1.5);
  paragraphStyle.setSpaceAbove(10);
  paragraphStyle.setSpaceBelow(10);
  
  // Apply text styling
  var textStyle = textRange.getTextStyle();
  textStyle.setFontSize(24)
    .setFontFamily('Georgia')
    .setForegroundColor('#1A73E8')
    .setBold(true)
    .setItalic(false)
    .setUnderline(false);
  
  // Apply partial text formatting
  textRange.insertText(0, 'Important: ');
  textRange.getRange(0, 10).getTextStyle()
    .setForegroundColor('#EA4335')
    .setFontSize(28);
  
  // Create gradient background
  var shape = slide.insertShape(SlidesApp.ShapeType.RECTANGLE, 50, 250, 500, 100);
  shape.getFill().setSolidFill('#E8F0FE');
  shape.getBorder().setTransparent();
  
  // Add styled bullet list
  var bulletBox = slide.insertTextBox('', 100, 400, 400, 150);
  var bulletText = bulletBox.getText();
  bulletText.setText(
    '• First point with emphasis\n' +
    '• Second point with detail\n' +
    '• Third point for consideration'
  );
  bulletText.getListStyle().applyListPreset(SlidesApp.ListPreset.DISC_CIRCLE_SQUARE);
}
```

### Working with Tables

```javascript
function createComplexTable(slide) {
  var table = slide.insertTable(5, 4, 50, 50, 600, 300);
  
  // Header row
  var headers = ['Product', 'Q1 Sales', 'Q2 Sales', 'Total'];
  for (var i = 0; i < headers.length; i++) {
    var headerCell = table.getCell(0, i);
    headerCell.getText().setText(headers[i]);
    headerCell.getFill().setSolidFill('#4285F4');
    headerCell.getText().getTextStyle()
      .setForegroundColor('#FFFFFF')
      .setBold(true);
  }
  
  // Data rows
  var data = [
    ['Product A', '$50,000', '$55,000', '$105,000'],
    ['Product B', '$30,000', '$35,000', '$65,000'],
    ['Product C', '$20,000', '$25,000', '$45,000'],
    ['Total', '$100,000', '$115,000', '$215,000']
  ];
  
  for (var row = 0; row < data.length; row++) {
    for (var col = 0; col < data[row].length; col++) {
      var cell = table.getCell(row + 1, col);
      cell.getText().setText(data[row][col]);
      
      // Style the total row
      if (row === data.length - 1) {
        cell.getFill().setSolidFill('#E8F0FE');
        cell.getText().getTextStyle().setBold(true);
      }
    }
  }
  
  // Apply borders to entire table
  for (var r = 0; r < table.getNumRows(); r++) {
    for (var c = 0; c < table.getNumColumns(); c++) {
      table.getCell(r, c).getBorder()
        .setWeight(1)
        .getLineFill()
        .setSolidFill('#DADCE0');
    }
  }
}
```

## Practical Use Cases

### Template-Based Presentation Generator

```javascript
function generateFromTemplate(templateId, data) {
  // Copy template presentation
  var template = SlidesApp.openById(templateId);
  var newPresentation = template.makeCopy('Generated - ' + data.title);
  
  // Replace all placeholders
  newPresentation.replaceAllText('{{title}}', data.title);
  newPresentation.replaceAllText('{{date}}', data.date);
  newPresentation.replaceAllText('{{presenter}}', data.presenter);
  
  // Process each slide
  var slides = newPresentation.getSlides();
  slides.forEach(function(slide, index) {
    // Replace slide-specific content
    if (data.slideContent[index]) {
      var content = data.slideContent[index];
      for (var key in content) {
        slide.replaceAllText('{{' + key + '}}', content[key]);
      }
    }
  });
  
  return newPresentation;
}

// Usage
var presentationData = {
  title: 'Q4 2024 Results',
  date: new Date().toLocaleDateString(),
  presenter: 'John Doe',
  slideContent: [
    {}, // Title slide
    {revenue: '$2.1M', growth: '15%'},
    {customers: '500', retention: '95%'}
  ]
};

var generated = generateFromTemplate('TEMPLATE_ID', presentationData);
```

### Bulk Slide Creator

```javascript
function createSlidesFromData(spreadsheetId) {
  var ss = SpreadsheetApp.openById(spreadsheetId);
  var sheet = ss.getSheets()[0];
  var data = sheet.getDataRange().getValues();
  
  var presentation = SlidesApp.create('Auto-Generated Slides');
  
  // Skip header row
  for (var i = 1; i < data.length; i++) {
    var rowData = data[i];
    var slide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE_AND_BODY);
    
    var shapes = slide.getShapes();
    shapes[0].getText().setText(rowData[0]); // Title
    shapes[1].getText().setText(rowData[1]); // Body content
    
    // Add image if URL provided
    if (rowData[2]) {
      try {
        var image = slide.insertImage(rowData[2]);
        image.setLeft(400).setTop(100).setWidth(200).setHeight(150);
      } catch (e) {
        console.log('Failed to insert image: ' + e.toString());
      }
    }
    
    // Add notes
    if (rowData[3]) {
      slide.getNotesPage().getSpeakerNotesShape()
        .getText().setText(rowData[3]);
    }
  }
  
  return presentation;
}
```

### Interactive Menu System

```javascript
function createInteractivePresentation() {
  var presentation = SlidesApp.create('Interactive Presentation');
  
  // Create menu slide
  var menuSlide = presentation.appendSlide(SlidesApp.PredefinedLayout.BLANK);
  menuSlide.insertTextBox('Main Menu', 200, 50, 400, 60)
    .getText().getTextStyle().setFontSize(36).setBold(true);
  
  // Create content slides
  var topics = ['Introduction', 'Features', 'Pricing', 'Contact'];
  var contentSlides = [];
  
  topics.forEach(function(topic, index) {
    var slide = presentation.appendSlide(SlidesApp.PredefinedLayout.TITLE_AND_BODY);
    slide.getShapes()[0].getText().setText(topic);
    slide.getShapes()[1].getText().setText('Content for ' + topic);
    contentSlides.push(slide);
    
    // Add back button
    var backButton = slide.insertShape(SlidesApp.ShapeType.RECTANGLE, 20, 20, 100, 40);
    backButton.getText().setText('← Menu');
    backButton.setLinkSlide(0); // Link back to menu
  });
  
  // Create menu buttons
  topics.forEach(function(topic, index) {
    var button = menuSlide.insertShape(
      SlidesApp.ShapeType.ROUND_RECTANGLE, 
      200, 150 + (index * 70), 
      400, 50
    );
    button.getText().setText(topic);
    button.getFill().setSolidFill('#4285F4');
    button.getText().getTextStyle().setForegroundColor('#FFFFFF').setFontSize(20);
    button.setLinkSlide(index + 1); // Link to content slide
  });
  
  return presentation;
}
```

### Slide Layout Analyzer

```javascript
function analyzeSlideLayouts(presentationId) {
  var presentation = SlidesApp.openById(presentationId);
  var slides = presentation.getSlides();
  var analysis = {
    totalSlides: slides.length,
    layouts: {},
    elements: {
      textBoxes: 0,
      images: 0,
      shapes: 0,
      tables: 0,
      videos: 0,
      groups: 0
    },
    slidesWithNotes: 0
  };
  
  slides.forEach(function(slide, index) {
    // Get layout
    var layout = slide.getLayout();
    if (layout) {
      var layoutName = layout.getLayoutName();
      analysis.layouts[layoutName] = (analysis.layouts[layoutName] || 0) + 1;
    }
    
    // Count elements
    var elements = slide.getPageElements();
    elements.forEach(function(element) {
      var type = element.getPageElementType();
      switch (type) {
        case SlidesApp.PageElementType.SHAPE:
          if (element.asShape().getShapeType() === SlidesApp.ShapeType.TEXT_BOX) {
            analysis.elements.textBoxes++;
          } else {
            analysis.elements.shapes++;
          }
          break;
        case SlidesApp.PageElementType.IMAGE:
          analysis.elements.images++;
          break;
        case SlidesApp.PageElementType.TABLE:
          analysis.elements.tables++;
          break;
        case SlidesApp.PageElementType.VIDEO:
          analysis.elements.videos++;
          break;
        case SlidesApp.PageElementType.GROUP:
          analysis.elements.groups++;
          break;
      }
    });
    
    // Check for speaker notes
    var notes = slide.getNotesPage().getSpeakerNotesShape().getText().asString();
    if (notes.trim().length > 0) {
      analysis.slidesWithNotes++;
    }
  });
  
  return analysis;
}
```

### Batch Image Replacer

```javascript
function replaceImagesInPresentation(presentationId, imageMapping) {
  var presentation = SlidesApp.openById(presentationId);
  var slides = presentation.getSlides();
  
  slides.forEach(function(slide) {
    var images = slide.getImages();
    
    images.forEach(function(image) {
      var sourceUrl = image.getSourceUrl();
      
      // Check if this image should be replaced
      if (imageMapping[sourceUrl]) {
        var newImageUrl = imageMapping[sourceUrl];
        
        // Get current position and size
        var left = image.getLeft();
        var top = image.getTop();
        var width = image.getWidth();
        var height = image.getHeight();
        var rotation = image.getRotation();
        
        // Remove old image
        image.remove();
        
        // Insert new image
        var newImage = slide.insertImage(newImageUrl);
        newImage.setLeft(left)
          .setTop(top)
          .setWidth(width)
          .setHeight(height)
          .setRotation(rotation);
      }
    });
  });
}

// Usage
var replacements = {
  'https://old-domain.com/logo.png': 'https://new-domain.com/logo.png',
  'https://old-domain.com/banner.jpg': 'https://new-domain.com/banner.jpg'
};

replaceImagesInPresentation('PRESENTATION_ID', replacements);
```

### Presentation Merger

```javascript
function mergePresentations(masterPresentationId, presentationIdsToMerge) {
  var masterPresentation = SlidesApp.openById(masterPresentationId);
  
  presentationIdsToMerge.forEach(function(presId) {
    var sourcePresentation = SlidesApp.openById(presId);
    var sourceSlides = sourcePresentation.getSlides();
    
    // Add separator slide
    var separatorSlide = masterPresentation.appendSlide();
    separatorSlide.insertTextBox(
      'Merged from: ' + sourcePresentation.getName(),
      100, 200, 600, 100
    ).getText().getTextStyle().setFontSize(24).setForegroundColor('#666666');
    
    // Copy each slide
    sourceSlides.forEach(function(sourceSlide) {
      // Note: Direct slide copying is limited, so we recreate content
      var newSlide = masterPresentation.appendSlide();
      
      // Copy text boxes
      var shapes = sourceSlide.getShapes();
      shapes.forEach(function(shape) {
        if (shape.getShapeType() === SlidesApp.ShapeType.TEXT_BOX) {
          var newShape = newSlide.insertTextBox(
            shape.getText().asString(),
            shape.getLeft(),
            shape.getTop(),
            shape.getWidth(),
            shape.getHeight()
          );
          
          // Copy text styling (simplified)
          newShape.getText().getTextStyle()
            .setFontSize(shape.getText().getTextStyle().getFontSize())
            .setFontFamily(shape.getText().getTextStyle().getFontFamily());
        }
      });
      
      // Copy images
      var images = sourceSlide.getImages();
      images.forEach(function(image) {
        if (image.getSourceUrl()) {
          newSlide.insertImage(image.getSourceUrl())
            .setLeft(image.getLeft())
            .setTop(image.getTop())
            .setWidth(image.getWidth())
            .setHeight(image.getHeight());
        }
      });
    });
  });
  
  return masterPresentation;
}
```

## Error Handling

```javascript
function safeSlidesOperation(operation) {
  try {
    return operation();
  } catch (e) {
    console.error('Slides operation failed: ' + e.toString());
    
    if (e.toString().includes('Invalid image URL')) {
      console.error('The image URL is not accessible or invalid');
    } else if (e.toString().includes('Limit exceeded')) {
      console.error('Presentation size or element limit exceeded');
    } else if (e.toString().includes('No item with the given ID')) {
      console.error('Presentation or slide not found');
    }
    
    return null;
  }
}

// Usage
var result = safeSlidesOperation(function() {
  var pres = SlidesApp.openById('PRESENTATION_ID');
  return pres.getName();
});
```

## Best Practices

1. **Optimize Element Positioning**:
```javascript
function alignElements(slide) {
  var elements = slide.getPageElements();
  var slideWidth = 720; // Standard width in points
  var slideHeight = 405; // Standard height in points
  
  elements.forEach(function(element) {
    // Center horizontally
    var elementWidth = element.getWidth();
    var centerX = (slideWidth - elementWidth) / 2;
    element.setLeft(centerX);
  });
}
```

2. **Use Master Slides for Consistency**:
```javascript
function applyMasterFormatting(presentation) {
  var masters = presentation.getMasters();
  if (masters.length > 0) {
    var master = masters[0];
    // Apply consistent formatting through master
  }
}
```

3. **Handle Large Presentations Efficiently**:
```javascript
function processLargePresentation(presentationId, batchSize) {
  var presentation = SlidesApp.openById(presentationId);
  var slides = presentation.getSlides();
  
  for (var i = 0; i < slides.length; i += batchSize) {
    var batch = slides.slice(i, i + batchSize);
    
    // Process batch
    batch.forEach(function(slide) {
      // Perform operations
    });
    
    // Pause to avoid timeout
    if (i + batchSize < slides.length) {
      Utilities.sleep(1000);
    }
  }
}
```