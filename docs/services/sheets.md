# Google Sheets Service API Reference

[← Back to Main Documentation](../README.md) | [View Examples →](../examples/sheets-examples.md)

## Service Overview
**SERVICE_NAME:** SpreadsheetApp  
**DESCRIPTION:** Access and create Google Sheets files. Allows scripts to create, access, and modify Google Sheets files  
**ACCESS_PATTERN:** `SpreadsheetApp.methodName()`  
**NOTE:** Operations are bundled for performance. Use `SpreadsheetApp.flush()` to force immediate changes

## Core Classes

### CLASS: SpreadsheetApp
**PROPERTIES:**
- `BorderStyle` → BorderStyle: Enum for border styles
- `DataValidationCriteria` → DataValidationCriteria: Enum for validation criteria
- `ProtectionType` → ProtectionType: Enum for protection types

**METHODS:**
- `create(name)` → Spreadsheet: Creates new spreadsheet
- `create(name, rows, columns)` → Spreadsheet: Creates with dimensions
- `flush()` → void: Applies all pending changes
- `getActive()` → Spreadsheet: Gets active spreadsheet
- `getActiveRange()` → Range: Gets selected range
- `getActiveSheet()` → Sheet: Gets active sheet
- `openById(id)` → Spreadsheet: Opens by ID
- `openByUrl(url)` → Spreadsheet: Opens by URL
- `newConditionalFormatRule()` → ConditionalFormatRuleBuilder: Creates format rule builder
- `newDataValidation()` → DataValidationBuilder: Creates validation builder

### CLASS: Spreadsheet
**METHODS:**
- `addEditor(emailAddress)` → Spreadsheet: Adds editor
- `addViewer(emailAddress)` → Spreadsheet: Adds viewer
- `appendRow(rowContents)` → Sheet: Appends row with data
- `copy(name)` → Spreadsheet: Copies spreadsheet
- `deleteSheet(sheet)` → void: Deletes sheet
- `duplicateActiveSheet()` → Sheet: Duplicates active sheet
- `getActiveSheet()` → Sheet: Gets active sheet
- `getDataRange()` → Range: Gets range with data
- `getId()` → String: Gets spreadsheet ID
- `getName()` → String: Gets spreadsheet name
- `getNumSheets()` → Integer: Gets sheet count
- `getRange(a1Notation)` → Range: Gets range by notation
- `getSheetByName(name)` → Sheet: Gets sheet by name
- `getSheets()` → Sheet[]: Gets all sheets
- `getUrl()` → String: Gets spreadsheet URL
- `insertSheet()` → Sheet: Inserts new sheet
- `insertSheet(sheetName)` → Sheet: Inserts named sheet
- `moveActiveSheet(pos)` → void: Moves active sheet
- `removeEditor(emailAddress)` → Spreadsheet: Removes editor
- `rename(newName)` → void: Renames spreadsheet
- `setActiveSheet(sheet)` → Sheet: Sets active sheet

### CLASS: Sheet
**METHODS:**
- `activate()` → Sheet: Activates sheet
- `appendRow(rowContents)` → Sheet: Appends row
- `autoResizeColumn(columnPosition)` → Sheet: Auto-sizes column
- `clear()` → Sheet: Clears all content
- `clearContents()` → Sheet: Clears content only
- `copyTo(spreadsheet)` → Sheet: Copies to spreadsheet
- `deleteColumn(columnPosition)` → Sheet: Deletes column
- `deleteRow(rowPosition)` → Sheet: Deletes row
- `getActiveCell()` → Range: Gets active cell
- `getDataRange()` → Range: Gets data range
- `getLastColumn()` → Integer: Gets last column with data
- `getLastRow()` → Integer: Gets last row with data
- `getMaxColumns()` → Integer: Gets total columns
- `getMaxRows()` → Integer: Gets total rows
- `getName()` → String: Gets sheet name
- `getRange(row, column)` → Range: Gets cell
- `getRange(row, column, numRows, numColumns)` → Range: Gets range
- `getRange(a1Notation)` → Range: Gets range by notation
- `hideColumn(column)` → void: Hides column
- `hideRow(row)` → void: Hides row
- `insertColumnAfter(afterPosition)` → Sheet: Inserts column
- `insertRowAfter(afterPosition)` → Sheet: Inserts row
- `protect()` → Protection: Creates protection
- `setColumnWidth(columnPosition, width)` → Sheet: Sets column width
- `setName(name)` → Sheet: Sets sheet name
- `setRowHeight(rowPosition, height)` → Sheet: Sets row height
- `sort(columnPosition)` → Sheet: Sorts by column

### CLASS: Range
**METHODS:**
- `activate()` → Range: Sets as active range
- `clear()` → Range: Clears content and format
- `clearContent()` → Range: Clears content only
- `clearFormat()` → Range: Clears format only
- `copyTo(destination)` → void: Copies to destination
- `getA1Notation()` → String: Gets A1 notation
- `getBackground()` → String: Gets background color
- `getCell(row, column)` → Range: Gets cell in range
- `getColumn()` → Integer: Gets start column
- `getDataValidation()` → DataValidation: Gets validation rule
- `getFormula()` → String: Gets formula
- `getFormulas()` → String[][]: Gets all formulas
- `getLastColumn()` → Integer: Gets end column
- `getLastRow()` → Integer: Gets end row
- `getNumColumns()` → Integer: Gets column count
- `getNumRows()` → Integer: Gets row count
- `getValue()` → Object: Gets cell value
- `getValues()` → Object[][]: Gets all values
- `merge()` → Range: Merges cells
- `offset(rowOffset, columnOffset)` → Range: Gets offset range
- `setBackground(color)` → Range: Sets background
- `setBorder(top, left, bottom, right, vertical, horizontal)` → Range: Sets borders
- `setDataValidation(rule)` → Range: Sets validation
- `setFontColor(color)` → Range: Sets font color
- `setFontSize(size)` → Range: Sets font size
- `setFormula(formula)` → Range: Sets formula
- `setFormulas(formulas)` → Range: Sets formulas grid
- `setHorizontalAlignment(alignment)` → Range: Sets alignment
- `setNumberFormat(format)` → Range: Sets number format
- `setValue(value)` → Range: Sets cell value
- `setValues(values)` → Range: Sets values grid
- `setWrap(isWrapEnabled)` → Range: Sets text wrap

### CLASS: Protection
**METHODS:**
- `addEditor(emailAddress)` → Protection: Adds editor
- `canEdit()` → Boolean: Checks edit permission
- `getDescription()` → String: Gets description
- `getEditors()` → User[]: Gets editors list
- `getProtectionType()` → ProtectionType: Gets type
- `getRange()` → Range: Gets protected range
- `remove()` → void: Removes protection
- `removeEditor(emailAddress)` → Protection: Removes editor
- `setDescription(description)` → Protection: Sets description
- `setWarningOnly(warningOnly)` → Protection: Sets warning mode

### CLASS: DataValidation
**METHODS:**
- `copy()` → DataValidationBuilder: Creates builder copy
- `getAllowInvalid()` → Boolean: Gets allow invalid setting
- `getCriteriaType()` → DataValidationCriteria: Gets criteria type
- `getCriteriaValues()` → Object[]: Gets criteria values
- `getHelpText()` → String: Gets help text

### CLASS: ConditionalFormatRule
**METHODS:**
- `copy()` → ConditionalFormatRuleBuilder: Creates builder copy
- `getBooleanCondition()` → BooleanCondition: Gets boolean condition
- `getGradientCondition()` → GradientCondition: Gets gradient condition
- `getRanges()` → Range[]: Gets applied ranges

## Enumerations

### ENUM: BorderStyle
**VALUES:**
- **DOTTED**: Dotted line borders
- **DASHED**: Dashed line borders
- **SOLID**: Thin solid line borders
- **SOLID_MEDIUM**: Medium solid line borders
- **SOLID_THICK**: Thick solid line borders
- **DOUBLE**: Two solid line borders

### ENUM: DataValidationCriteria
**VALUES:**
- **VALUE_IN_LIST**: Requires value from list
- **VALUE_IN_RANGE**: Requires value from range
- **NUMBER_BETWEEN**: Number between values
- **NUMBER_GREATER_THAN**: Number greater than value
- **NUMBER_LESS_THAN**: Number less than value
- **DATE_BETWEEN**: Date between values
- **DATE_AFTER**: Date after value
- **DATE_BEFORE**: Date before value
- **TEXT_CONTAINS**: Text contains value
- **TEXT_EQUAL_TO**: Text equals value
- **CUSTOM_FORMULA**: Custom formula validation
- **CHECKBOX**: Checkbox validation

### ENUM: ProtectionType
**VALUES:**
- **RANGE**: Protection for a range
- **SHEET**: Protection for a sheet

### ENUM: HorizontalAlignment
**VALUES:**
- **LEFT**: Left alignment
- **CENTER**: Center alignment
- **RIGHT**: Right alignment

## Common Patterns

### CREATE_SPREADSHEET:
```javascript
var ss = SpreadsheetApp.create('New Spreadsheet');
```

### OPEN_SPREADSHEET:
```javascript
var ss = SpreadsheetApp.openById('SPREADSHEET_ID');
var ss = SpreadsheetApp.openByUrl('https://docs.google.com/spreadsheets/d/...');
```

### GET_RANGE_VALUES:
```javascript
var sheet = SpreadsheetApp.getActiveSheet();
var range = sheet.getRange('A1:C10');
var values = range.getValues();
```

### SET_RANGE_VALUES:
```javascript
var values = [
  ['Row 1 Col 1', 'Row 1 Col 2'],
  ['Row 2 Col 1', 'Row 2 Col 2']
];
range.setValues(values);
```

### APPEND_ROW:
```javascript
var sheet = SpreadsheetApp.getActiveSheet();
sheet.appendRow(['Value1', 'Value2', 'Value3']);
```

### FORMAT_RANGE:
```javascript
var range = sheet.getRange('A1:B10');
range.setBackground('#ffffff')
     .setFontColor('#000000')
     .setFontSize(12)
     .setBorder(true, true, true, true, false, false);
```

### DATA_VALIDATION:
```javascript
var cell = sheet.getRange('A1');
var rule = SpreadsheetApp.newDataValidation()
    .requireValueInList(['Option1', 'Option2', 'Option3'])
    .setAllowInvalid(false)
    .build();
cell.setDataValidation(rule);
```

### CONDITIONAL_FORMATTING:
```javascript
var range = sheet.getRange('A1:A10');
var rule = SpreadsheetApp.newConditionalFormatRule()
    .whenNumberGreaterThan(100)
    .setBackground('#00FF00')
    .setRanges([range])
    .build();
var rules = sheet.getConditionalFormatRules();
rules.push(rule);
sheet.setConditionalFormatRules(rules);
```

### PROTECT_RANGE:
```javascript
var range = sheet.getRange('A1:C10');
var protection = range.protect().setDescription('Protected Range');
protection.addEditor('user@example.com');
protection.setWarningOnly(true);
```

### SORT_DATA:
```javascript
var range = sheet.getRange('A1:C20');
range.sort({column: 1, ascending: true});
```