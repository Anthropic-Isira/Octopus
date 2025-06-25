# Google Apps Script Installation & Usage Guide

[← Back to Main Documentation](README.md)

## Overview

This guide walks you through installing and using Google Apps Script projects step-by-step. Whether you're new to Google Apps Script or need a refresher, this guide will help you get scripts running quickly.

## Table of Contents

1. [Understanding Google Apps Script](#understanding-google-apps-script)
2. [Installation Methods](#installation-methods)
3. [Step-by-Step Installation](#step-by-step-installation)
4. [Running Your First Script](#running-your-first-script)
5. [Using the Scripts in This Repository](#using-the-scripts-in-this-repository)
6. [Common Setup Issues](#common-setup-issues)
7. [Best Practices](#best-practices)

## Understanding Google Apps Script

Google Apps Script (GAS) is JavaScript that runs in Google's cloud, not on your computer. Think of it as macros for Google Workspace.

**Key Points:**
- Scripts run on Google's servers
- No software installation needed
- Access through browser only
- Automatic integration with Google services

## Installation Methods

### Method 1: Copy from Repository (Recommended)

Best for using scripts from this repository.

1. **Open Google Apps Script Editor**
   - Go to [script.google.com](https://script.google.com)
   - Click "New project"

2. **Copy the Script**
   - Copy the entire `.gs` file content from this repository
   - Delete the default `function myFunction()` in the editor
   - Paste the copied code

3. **Save and Name**
   - Click the untitled project name
   - Enter a descriptive name (e.g., "Clockwise Clone")
   - Press Ctrl+S (or Cmd+S on Mac) to save

### Method 2: Create in Google Sheets/Docs

Best for sheet-specific scripts.

1. **Open Your Google Sheet/Doc**
   - Go to your Google Sheet or Doc
   - Click Extensions → Apps Script

2. **Add the Code**
   - The script editor opens
   - Paste your code
   - Save with Ctrl+S

### Method 3: Shared Script Link

If someone shares a script link:

1. Click the shared link
2. Click "Make a copy"
3. The script opens in your account

## Step-by-Step Installation

Let's install the Clockwise Clone script as an example:

### Step 1: Access the Script Editor

```
1. Go to script.google.com
2. Sign in with your Google account
3. Click "New project"
```

### Step 2: Prepare the Script

```
1. Delete all default code in the editor
2. Copy all content from ClockwiseClone.gs in this repository
3. Paste into the editor
```

### Step 3: Configure the Script

Look for configuration sections at the top:

```javascript
// Global Configuration
const ClockwiseConfig = {
  FOCUS_TIME: {
    MIN_DURATION_HOURS: 2,     // Change these values
    PREFERRED_TIME: 'morning', // Options: 'morning', 'afternoon', 'any'
  },
  WORK_HOURS: {
    START: 9,  // 9 AM - adjust to your schedule
    END: 17,   // 5 PM
  }
};
```

### Step 4: Save and Authorize

```
1. Press Ctrl+S to save
2. Name your project (e.g., "My Calendar Optimizer")
3. Click the "Run" button or press Ctrl+R
4. Choose a function to run (usually the install function)
```

### Step 5: Handle Authorization

When you first run a script:

```
1. A dialog appears: "Authorization required"
2. Click "Review permissions"
3. Choose your Google account
4. You may see "This app isn't verified"
5. Click "Advanced" → "Go to [Script Name] (unsafe)"
6. Review permissions and click "Allow"
```

**Why these permissions?** The script needs access to:
- Your calendar (to optimize it)
- Your email (to send summaries)
- Time-based triggers (to run automatically)

## Running Your First Script

### Manual Execution

1. **Select a Function**
   ```
   In the editor toolbar, dropdown shows available functions
   Select the function you want to run
   ```

2. **Run the Function**
   ```
   Click the "Run" button (▶️)
   Or press Ctrl+R (Cmd+R on Mac)
   ```

3. **Check Results**
   ```
   View logs: View → Logs (or Ctrl+Enter)
   Check your Google services for changes
   ```

### Setting Up Automatic Execution

Most scripts include an install function:

```javascript
// Look for a function like this:
function installClockwiseClone() {
  // Sets up automatic triggers
}
```

Run this once to enable automatic execution.

## Using the Scripts in This Repository

### Clockwise Clone (Calendar Optimization)

**Purpose:** Automatically optimizes your calendar by adding focus time and managing meetings.

**Installation:**
1. Copy `ClockwiseClone.gs` to Apps Script editor
2. Adjust configuration values
3. Run `installClockwiseClone()`
4. Check your calendar for "Focus Time" blocks

**Daily Use:**
- Runs automatically at 7 AM
- Manual run: Execute `optimizeCalendar()`
- Weekly reports sent to your email

### Email Campaign Manager

**Purpose:** Manages email campaigns with scheduling and tracking.

**Installation:**
1. Copy `EmailCampaignManager.gs` to Apps Script editor
2. Create a Google Sheet for campaign data
3. Update `SPREADSHEET_ID` in the config
4. Run `initializeCampaignSheet()`

**Usage:**
```javascript
// Create a campaign
createCampaign('Holiday Sale', 'promotional');

// Add recipients  
addRecipientsToCampaign('campaign-id', ['email1@example.com']);

// Schedule sending
scheduleCampaign('campaign-id', new Date('2024-12-01'));
```

### Time Blocker

**Purpose:** Blocks time slots in your calendar based on rules.

**Installation:**
1. Copy `TimeBlocker.gs` to Apps Script editor  
2. Configure blocking rules
3. Run `setupTimeBlocker()`

**Usage:**
- Automatically blocks lunch time
- Protects focus hours
- Prevents back-to-back meetings

## Common Setup Issues

### "This app isn't verified"

**Solution:**
```
1. This appears for personal scripts
2. Click "Advanced"
3. Click "Go to [Script Name] (unsafe)"
4. It's safe - you're running your own code
```

### "Exceeded maximum execution time"

**Solution:**
```javascript
// Add batch processing
function processManyItems() {
  var items = getItems();
  var batchSize = 100;
  
  for (var i = 0; i < items.length; i += batchSize) {
    var batch = items.slice(i, i + batchSize);
    processBatch(batch);
    
    // Avoid timeout
    if (i + batchSize < items.length) {
      Utilities.sleep(1000); // Pause 1 second
    }
  }
}
```

### "You do not have permission"

**Common Causes:**
1. Trying to access files you don't own
2. Script needs reauthorization
3. Accessing deleted resources

**Solution:**
```javascript
// Add permission checks
function safeAccessFile(fileId) {
  try {
    var file = DriveApp.getFileById(fileId);
    return file;
  } catch (e) {
    console.log('No access to file: ' + fileId);
    return null;
  }
}
```

### Script Doesn't Run Automatically

**Check Triggers:**
```
1. Go to Apps Script editor
2. Click clock icon (Triggers) in left sidebar
3. Verify triggers are set up
4. Check failure notifications
```

## Best Practices

### 1. Always Test First

```javascript
// Create test functions
function testEmailFunction() {
  // Test with your own email first
  sendEmail('your-email@gmail.com', 'Test', 'Testing script');
}
```

### 2. Use Configuration Objects

```javascript
// Easy to modify without changing code
const CONFIG = {
  EMAIL_RECIPIENT: 'user@example.com',
  MAX_RETRIES: 3,
  DEBUG_MODE: true
};
```

### 3. Add Safety Checks

```javascript
// Prevent accidental mass operations
function deleteOldFiles() {
  var files = DriveApp.getFilesByName('temp');
  var count = 0;
  
  while (files.hasNext() && count < 10) { // Limit to 10
    var file = files.next();
    console.log('Would delete: ' + file.getName());
    // file.setTrashed(true); // Uncomment when ready
    count++;
  }
}
```

### 4. Enable Logging

```javascript
// Track what your script does
function enhancedLogging(message) {
  console.log(new Date() + ': ' + message);
  
  // Also log to a sheet
  var sheet = SpreadsheetApp.openById('LOG_SHEET_ID');
  sheet.appendRow([new Date(), message]);
}
```

### 5. Handle Errors Gracefully

```javascript
function robustFunction() {
  try {
    // Main code
    riskyOperation();
  } catch (error) {
    console.error('Error occurred:', error);
    // Send notification
    GmailApp.sendEmail(
      Session.getActiveUser().getEmail(),
      'Script Error',
      'Error in robustFunction: ' + error
    );
  }
}
```

## Quick Reference Card

### Essential Commands

| Action | How To |
|--------|--------|
| Open Script Editor | [script.google.com](https://script.google.com) |
| Save Script | Ctrl+S (Cmd+S on Mac) |
| Run Function | Ctrl+R or click ▶️ |
| View Logs | Ctrl+Enter |
| Open Triggers | Click ⏰ in sidebar |
| Test Function | Create test function, run it |

### Common Functions to Remember

```javascript
// Install/Setup functions - run once
installScriptName()
setupTriggers()
initialize()

// Test functions - run to test
testFunction()
debugMode()

// Main functions - regular use
processData()
optimizeCalendar()
sendReport()
```

## Getting Help

1. **Check Logs First**
   - View → Logs shows what happened
   - Look for error messages

2. **Read Error Messages**
   - Google provides helpful error descriptions
   - Common errors explained in [Common Errors Guide](troubleshooting/common-errors.md)

3. **Start Simple**
   - Test with small data sets
   - Run functions manually before automating

4. **Use Debug Functions**
   ```javascript
   function debugInfo() {
     console.log('Current user:', Session.getActiveUser().getEmail());
     console.log('Time zone:', Session.getScriptTimeZone());
     console.log('Quotas:', MailApp.getRemainingDailyQuota());
   }
   ```

## Next Steps

1. Start with a simple script like Time Blocker
2. Read [GAS Nuances & Gotchas](core-concepts/gas-nuances-gotchas.md) to avoid common pitfalls
3. Explore the [Examples](examples/) directory for more use cases
4. Check [Performance Guide](troubleshooting/performance-guide.md) for optimization tips

Remember: Google Apps Script is powerful but has [unique limitations](core-concepts/gas-nuances-gotchas.md). Start small, test often, and gradually build complexity.