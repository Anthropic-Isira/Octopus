# Google Sheets Service Examples

[← Back to Sheets API](../services/sheets.md) | [← Back to Main Documentation](../README.md)

## Basic Operations

### Creating and Opening Spreadsheets

```javascript
// Create a new spreadsheet
var ss = SpreadsheetApp.create('My New Spreadsheet');
console.log('Created spreadsheet with ID: ' + ss.getId());

// Create with specific dimensions
var customSS = SpreadsheetApp.create('Custom Sheet', 50, 10); // 50 rows, 10 columns

// Open existing spreadsheet by ID
var existingSS = SpreadsheetApp.openById('SPREADSHEET_ID_HERE');

// Open by URL
var ssByUrl = SpreadsheetApp.openByUrl('https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit');

// Get active spreadsheet (when running as bound script)
var activeSS = SpreadsheetApp.getActiveSpreadsheet();
```

### Working with Sheets

```javascript
// Get active sheet
var sheet = SpreadsheetApp.getActiveSheet();

// Create new sheet
var newSheet = ss.insertSheet('Data');

// Get sheet by name
var namedSheet = ss.getSheetByName('Sheet1');

// Rename sheet
sheet.setName('Updated Name');

// Delete sheet
ss.deleteSheet(namedSheet);

// Duplicate sheet
var duplicated = sheet.copyTo(ss).setName('Copy of ' + sheet.getName());
```

### Reading and Writing Data

```javascript
// Get single cell value
var cellValue = sheet.getRange('A1').getValue();

// Get range of values
var rangeValues = sheet.getRange('A1:C10').getValues();

// Set single cell value
sheet.getRange('A1').setValue('Hello World');

// Set multiple values
var values = [
  ['Name', 'Age', 'Email'],
  ['John Doe', 30, 'john@example.com'],
  ['Jane Smith', 25, 'jane@example.com']
];
sheet.getRange(1, 1, values.length, values[0].length).setValues(values);

// Append row
sheet.appendRow(['New', 'Row', 'Data']);

// Get last row with data
var lastRow = sheet.getLastRow();
var lastColumn = sheet.getLastColumn();
```

## Advanced Examples

### Data Formatting

```javascript
function formatSpreadsheet() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var headerRange = sheet.getRange('A1:E1');
  
  // Format header row
  headerRange
    .setBackground('#4285F4')
    .setFontColor('#FFFFFF')
    .setFontSize(12)
    .setFontWeight('bold')
    .setHorizontalAlignment('center');
  
  // Apply borders
  var dataRange = sheet.getRange('A1:E10');
  dataRange.setBorder(true, true, true, true, true, true);
  
  // Number formatting
  sheet.getRange('B2:B10').setNumberFormat('$#,##0.00'); // Currency
  sheet.getRange('C2:C10').setNumberFormat('0.00%'); // Percentage
  sheet.getRange('D2:D10').setNumberFormat('MM/dd/yyyy'); // Date
  
  // Conditional formatting
  var rule = SpreadsheetApp.newConditionalFormatRule()
    .whenNumberGreaterThan(100)
    .setBackground('#B7E1CD')
    .setRanges([sheet.getRange('B2:B10')])
    .build();
  
  var rules = sheet.getConditionalFormatRules();
  rules.push(rule);
  sheet.setConditionalFormatRules(rules);
}
```

### Working with Formulas

```javascript
// Set formulas
sheet.getRange('D2').setFormula('=SUM(A2:C2)');
sheet.getRange('E2').setFormula('=AVERAGE(A2:C2)');

// Set array formula
sheet.getRange('F2:F10').setFormula('=ARRAYFORMULA(A2:A10 * B2:B10)');

// Copy formulas down
var formulaRange = sheet.getRange('D2:E2');
var targetRange = sheet.getRange('D3:E10');
formulaRange.copyTo(targetRange);

// Get formulas
var formulas = sheet.getRange('D2:E10').getFormulas();
console.log('Formulas:', formulas);

// R1C1 notation
sheet.getRange('G2').setFormulaR1C1('=R[0]C[-6]+R[0]C[-5]'); // Adds A2+B2
```

### Data Validation

```javascript
function setupDataValidation() {
  var sheet = SpreadsheetApp.getActiveSheet();
  
  // Dropdown from list
  var dropdownRule = SpreadsheetApp.newDataValidation()
    .requireValueInList(['Option 1', 'Option 2', 'Option 3'])
    .setAllowInvalid(false)
    .setHelpText('Please select from the dropdown')
    .build();
  sheet.getRange('A2:A10').setDataValidation(dropdownRule);
  
  // Number range validation
  var numberRule = SpreadsheetApp.newDataValidation()
    .requireNumberBetween(0, 100)
    .setHelpText('Enter a number between 0 and 100')
    .build();
  sheet.getRange('B2:B10').setDataValidation(numberRule);
  
  // Date validation
  var dateRule = SpreadsheetApp.newDataValidation()
    .requireDateAfter(new Date())
    .setHelpText('Enter a future date')
    .build();
  sheet.getRange('C2:C10').setDataValidation(dateRule);
  
  // Checkbox
  var checkboxRule = SpreadsheetApp.newDataValidation()
    .requireCheckbox()
    .build();
  sheet.getRange('D2:D10').setDataValidation(checkboxRule);
}
```

## Practical Use Cases

### Data Import and Processing

```javascript
function importAndProcessData(csvUrl) {
  var response = UrlFetchApp.fetch(csvUrl);
  var csvData = response.getContentText();
  var data = Utilities.parseCsv(csvData);
  
  // Create new sheet for imported data
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.insertSheet('Imported Data');
  
  // Import data
  if (data.length > 0) {
    sheet.getRange(1, 1, data.length, data[0].length).setValues(data);
  }
  
  // Process data
  var headers = data[0];
  var processedData = [];
  
  for (var i = 1; i < data.length; i++) {
    var row = data[i];
    var processedRow = [];
    
    // Example processing
    processedRow.push(row[0].toUpperCase()); // Uppercase first column
    processedRow.push(parseFloat(row[1]) * 1.1); // Add 10% to second column
    processedRow.push(new Date(row[2])); // Parse date
    
    processedData.push(processedRow);
  }
  
  // Create processed sheet
  var processedSheet = ss.insertSheet('Processed Data');
  processedSheet.getRange(1, 1, 1, 3).setValues([['Name', 'Adjusted Value', 'Date']]);
  processedSheet.getRange(2, 1, processedData.length, 3).setValues(processedData);
  
  return processedSheet.getName();
}
```

### Dynamic Dashboard

```javascript
function createDashboard() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var dataSheet = ss.getSheetByName('Sales Data');
  var dashboardSheet = ss.insertSheet('Dashboard');
  
  // Summary statistics
  dashboardSheet.getRange('A1').setValue('Sales Dashboard');
  dashboardSheet.getRange('A1').setFontSize(18).setFontWeight('bold');
  
  // Create summary table
  var summaryData = [
    ['Metric', 'Value'],
    ['Total Sales', '=SUM(\'Sales Data\'!B:B)'],
    ['Average Sale', '=AVERAGE(\'Sales Data\'!B:B)'],
    ['Max Sale', '=MAX(\'Sales Data\'!B:B)'],
    ['Total Customers', '=COUNTA(\'Sales Data\'!A:A)-1'],
    ['Date Range', '=MIN(\'Sales Data\'!C:C)&" to "&MAX(\'Sales Data\'!C:C)']
  ];
  
  dashboardSheet.getRange('A3:B8').setValues(summaryData);
  
  // Format summary table
  dashboardSheet.getRange('A3:B3').setBackground('#E8E8E8').setFontWeight('bold');
  dashboardSheet.getRange('B4:B8').setNumberFormat('$#,##0.00');
  
  // Add charts
  var chartBuilder = dashboardSheet.newChart()
    .setChartType(Charts.ChartType.COLUMN)
    .addRange(dataSheet.getRange('A1:B100'))
    .setPosition(10, 1, 0, 0)
    .setOption('title', 'Sales by Customer')
    .setOption('width', 600)
    .setOption('height', 400);
  
  dashboardSheet.insertChart(chartBuilder.build());
  
  // Auto-resize columns
  dashboardSheet.autoResizeColumns(1, 2);
}
```

### Automated Report Generator

```javascript
function generateMonthlyReport() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var dataSheet = ss.getSheetByName('Raw Data');
  var templateSheet = ss.getSheetByName('Report Template');
  
  // Create new report sheet
  var reportDate = new Date();
  var reportName = 'Report_' + Utilities.formatDate(reportDate, 'GMT', 'yyyy_MM');
  var reportSheet = templateSheet.copyTo(ss).setName(reportName);
  
  // Get data for current month
  var allData = dataSheet.getDataRange().getValues();
  var currentMonth = reportDate.getMonth();
  var currentYear = reportDate.getFullYear();
  
  var monthData = allData.filter(function(row, index) {
    if (index === 0) return false; // Skip header
    var rowDate = new Date(row[0]); // Assuming date is in first column
    return rowDate.getMonth() === currentMonth && 
           rowDate.getFullYear() === currentYear;
  });
  
  // Calculate metrics
  var totalSales = monthData.reduce(function(sum, row) {
    return sum + row[2]; // Assuming sales amount in column 3
  }, 0);
  
  var avgSale = totalSales / monthData.length;
  
  // Update report with calculated values
  reportSheet.getRange('B2').setValue(reportName);
  reportSheet.getRange('B3').setValue(totalSales);
  reportSheet.getRange('B4').setValue(avgSale);
  reportSheet.getRange('B5').setValue(monthData.length);
  
  // Insert detailed data
  if (monthData.length > 0) {
    reportSheet.getRange(10, 1, monthData.length, monthData[0].length)
      .setValues(monthData);
  }
  
  // Email report
  var pdf = ss.getAs('application/pdf');
  GmailApp.sendEmail('manager@example.com', 
    'Monthly Report - ' + reportName,
    'Please find attached the monthly report.',
    {attachments: [pdf]});
  
  return reportSheet.getName();
}
```

### Data Sync Between Sheets

```javascript
function syncSheets(sourceId, targetId, sheetName) {
  var sourceSpreadsheet = SpreadsheetApp.openById(sourceId);
  var targetSpreadsheet = SpreadsheetApp.openById(targetId);
  
  var sourceSheet = sourceSpreadsheet.getSheetByName(sheetName);
  var targetSheet = targetSpreadsheet.getSheetByName(sheetName) || 
                   targetSpreadsheet.insertSheet(sheetName);
  
  // Clear target sheet
  targetSheet.clear();
  
  // Copy all data
  var sourceData = sourceSheet.getDataRange();
  var values = sourceData.getValues();
  var formulas = sourceData.getFormulas();
  var backgrounds = sourceData.getBackgrounds();
  var fontColors = sourceData.getFontColors();
  
  // Set values
  if (values.length > 0) {
    targetSheet.getRange(1, 1, values.length, values[0].length)
      .setValues(values);
  }
  
  // Copy formulas
  for (var i = 0; i < formulas.length; i++) {
    for (var j = 0; j < formulas[i].length; j++) {
      if (formulas[i][j]) {
        targetSheet.getRange(i + 1, j + 1).setFormula(formulas[i][j]);
      }
    }
  }
  
  // Copy formatting
  targetSheet.getRange(1, 1, backgrounds.length, backgrounds[0].length)
    .setBackgrounds(backgrounds)
    .setFontColors(fontColors);
  
  // Copy column widths
  for (var col = 1; col <= sourceSheet.getLastColumn(); col++) {
    var width = sourceSheet.getColumnWidth(col);
    targetSheet.setColumnWidth(col, width);
  }
  
  // Log sync
  var logSheet = targetSpreadsheet.getSheetByName('Sync Log') || 
                targetSpreadsheet.insertSheet('Sync Log');
  logSheet.appendRow([new Date(), 'Synced from ' + sourceId, sheetName]);
}
```

### Pivot Table Creator

```javascript
function createPivotTable() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sourceSheet = ss.getSheetByName('Sales Data');
  var pivotSheet = ss.insertSheet('Pivot Analysis');
  
  // Assuming data structure: Date, Product, Category, Amount, Quantity
  var sourceRange = sourceSheet.getDataRange();
  
  // Create pivot table using formulas
  // Get unique categories
  pivotSheet.getRange('A1').setValue('Category Summary');
  pivotSheet.getRange('A2').setFormula(
    '=UNIQUE(\'Sales Data\'!C2:C)'
  );
  
  // Sum by category
  pivotSheet.getRange('B1').setValue('Total Sales');
  pivotSheet.getRange('B2').setFormula(
    '=ARRAYFORMULA(IF(A2:A="",,SUMIF(\'Sales Data\'!C:C,A2:A,\'Sales Data\'!D:D)))'
  );
  
  // Count by category
  pivotSheet.getRange('C1').setValue('Transaction Count');
  pivotSheet.getRange('C2').setFormula(
    '=ARRAYFORMULA(IF(A2:A="",,COUNTIF(\'Sales Data\'!C:C,A2:A)))'
  );
  
  // Average by category
  pivotSheet.getRange('D1').setValue('Average Sale');
  pivotSheet.getRange('D2').setFormula(
    '=ARRAYFORMULA(IF(A2:A="",,B2:B/C2:C))'
  );
  
  // Format the pivot table
  pivotSheet.getRange('A1:D1').setBackground('#4285F4')
    .setFontColor('#FFFFFF')
    .setFontWeight('bold');
  
  pivotSheet.getRange('B:B').setNumberFormat('$#,##0.00');
  pivotSheet.getRange('D:D').setNumberFormat('$#,##0.00');
  
  // Auto-resize columns
  pivotSheet.autoResizeColumns(1, 4);
}
```

### Batch Operations

```javascript
function batchUpdateCells() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var range = sheet.getDataRange();
  var values = range.getValues();
  
  // Process all values in memory
  var updatedValues = values.map(function(row, rowIndex) {
    return row.map(function(cell, colIndex) {
      // Skip header row
      if (rowIndex === 0) return cell;
      
      // Example transformations
      if (typeof cell === 'string') {
        return cell.trim().toUpperCase();
      } else if (typeof cell === 'number') {
        return Math.round(cell * 100) / 100; // Round to 2 decimals
      }
      return cell;
    });
  });
  
  // Write all values at once
  range.setValues(updatedValues);
  
  // Force the changes to be written
  SpreadsheetApp.flush();
}
```

## Error Handling

```javascript
function safeSheetOperation(operation) {
  try {
    return operation();
  } catch (e) {
    console.error('Sheet operation failed: ' + e.toString());
    
    if (e.toString().includes('exceeds the limit')) {
      console.error('Cell limit exceeded. Try reducing data size.');
    } else if (e.toString().includes('No item with the given ID')) {
      console.error('Spreadsheet or sheet not found.');
    } else if (e.toString().includes('does not have permission')) {
      console.error('Insufficient permissions to access the spreadsheet.');
    }
    
    return null;
  }
}

// Usage
var result = safeSheetOperation(function() {
  var ss = SpreadsheetApp.openById('SPREADSHEET_ID');
  return ss.getName();
});
```

## Best Practices

1. **Batch Operations**: Always batch read/write operations:
```javascript
// Good - single operation
var values = sheet.getRange('A1:Z1000').getValues();

// Bad - multiple operations
for (var i = 1; i <= 1000; i++) {
  var value = sheet.getRange('A' + i).getValue(); // Avoid this!
}
```

2. **Use flush() for immediate updates**:
```javascript
sheet.getRange('A1').setValue('Important update');
SpreadsheetApp.flush(); // Forces the update immediately
```

3. **Optimize range references**:
```javascript
// Use getDataRange() for all data
var allData = sheet.getDataRange().getValues();

// Use getLastRow() and getLastColumn() for dynamic ranges
var lastRow = sheet.getLastRow();
var dataRange = sheet.getRange(1, 1, lastRow, 5);
```