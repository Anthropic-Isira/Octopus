# Google Calendar Service Examples

[← Back to Calendar API](../services/calendar.md) | [← Back to Main Documentation](../README.md)

## Basic Operations

### Creating Events

```javascript
// Create a simple event
var event = CalendarApp.createEvent('Team Meeting',
  new Date('December 15, 2024 10:00:00'),
  new Date('December 15, 2024 11:00:00'));

// Create an all-day event
var allDayEvent = CalendarApp.createAllDayEvent('Company Holiday',
  new Date('December 25, 2024'));

// Create an event with options
var detailedEvent = CalendarApp.createEvent('Project Review',
  new Date('December 20, 2024 14:00:00'),
  new Date('December 20, 2024 15:30:00'),
  {
    description: 'Quarterly project review meeting',
    location: 'Conference Room A',
    guests: 'colleague@example.com,manager@example.com',
    sendInvites: true
  });
```

### Managing Calendars

```javascript
// Get the default calendar
var defaultCalendar = CalendarApp.getDefaultCalendar();
console.log('Default calendar: ' + defaultCalendar.getName());

// Create a new calendar
var newCalendar = CalendarApp.createCalendar('Project Calendar', {
  summary: 'Calendar for project events',
  color: CalendarApp.Color.BLUE
});

// Get a specific calendar by ID
var calendar = CalendarApp.getCalendarById('calendar-id@example.com');

// Get all calendars
var calendars = CalendarApp.getAllCalendars();
calendars.forEach(function(cal) {
  console.log(cal.getName() + ' - ' + cal.getId());
});
```

### Retrieving Events

```javascript
// Get events for today
var today = new Date();
var events = CalendarApp.getEventsForDay(today);

events.forEach(function(event) {
  console.log(event.getTitle() + ' at ' + event.getStartTime());
});

// Get events in a date range
var startDate = new Date('December 1, 2024');
var endDate = new Date('December 31, 2024');
var rangeEvents = CalendarApp.getEvents(startDate, endDate);

// Search for specific events
var searchEvents = CalendarApp.getEvents(startDate, endDate, {
  search: 'meeting'
});
```

## Advanced Examples

### Recurring Events

```javascript
// Create a weekly recurring event
var recurrence = CalendarApp.newRecurrence()
  .addWeeklyRule()
  .onlyOnWeekdays([CalendarApp.Weekday.MONDAY, CalendarApp.Weekday.WEDNESDAY])
  .until(new Date('March 31, 2024'));

var recurringEvent = CalendarApp.createEventSeries('Team Standup',
  new Date('January 1, 2024 09:00:00'),
  new Date('January 1, 2024 09:30:00'),
  recurrence);

// Create a monthly recurring event
var monthlyRecurrence = CalendarApp.newRecurrence()
  .addMonthlyRule()
  .onlyOnMonthDay(15);

var monthlyEvent = CalendarApp.createEventSeries('Monthly Review',
  new Date('January 15, 2024 14:00:00'),
  new Date('January 15, 2024 15:00:00'),
  monthlyRecurrence);
```

### Event Modifications

```javascript
// Get and modify an event
var events = CalendarApp.getEventsForDay(new Date());
if (events.length > 0) {
  var event = events[0];
  
  // Update event details
  event.setTitle('Updated Title')
       .setLocation('New Location')
       .setDescription('Updated description');
  
  // Add reminders
  event.addEmailReminder(60);  // 60 minutes before
  event.addPopupReminder(15);  // 15 minutes before
  
  // Manage guests
  event.addGuest('newguest@example.com');
  event.setGuestsCanModify(true);
  event.setGuestsCanSeeGuests(true);
  
  // Set event color
  event.setColor(CalendarApp.EventColor.PALE_BLUE);
}
```

### Working with Event Series

```javascript
// Get an event series
var eventSeries = CalendarApp.getEventSeriesById('series-id');

// Modify the entire series
eventSeries.setLocation('New Default Location');

// Change recurrence pattern
var newRecurrence = CalendarApp.newRecurrence()
  .addWeeklyRule()
  .onlyOnWeekday(CalendarApp.Weekday.THURSDAY)
  .times(10);  // Only 10 occurrences

eventSeries.setRecurrence(newRecurrence, 
  new Date('January 1, 2024 10:00:00'),
  new Date('January 1, 2024 11:00:00'));
```

## Practical Use Cases

### Meeting Scheduler

```javascript
function scheduleMeetings(attendees, duration, subject) {
  var calendar = CalendarApp.getDefaultCalendar();
  var now = new Date();
  var oneWeekFromNow = new Date(now.getTime() + 7 * 24 * 60 * 60 * 1000);
  
  // Find free time slots
  var events = calendar.getEvents(now, oneWeekFromNow);
  var busyTimes = events.map(function(e) {
    return {start: e.getStartTime(), end: e.getEndTime()};
  });
  
  // Create meeting in first available slot
  var meetingStart = findNextFreeSlot(busyTimes, duration);
  var meetingEnd = new Date(meetingStart.getTime() + duration * 60 * 1000);
  
  var meeting = calendar.createEvent(subject, meetingStart, meetingEnd, {
    guests: attendees.join(','),
    sendInvites: true
  });
  
  return meeting;
}
```

### Calendar Sync

```javascript
function syncCalendars(sourceCalendarId, targetCalendarId) {
  var sourceCalendar = CalendarApp.getCalendarById(sourceCalendarId);
  var targetCalendar = CalendarApp.getCalendarById(targetCalendarId);
  
  var startDate = new Date();
  var endDate = new Date(startDate.getTime() + 30 * 24 * 60 * 60 * 1000); // 30 days
  
  var sourceEvents = sourceCalendar.getEvents(startDate, endDate);
  
  sourceEvents.forEach(function(event) {
    // Check if event already exists in target
    var existingEvents = targetCalendar.getEvents(
      event.getStartTime(), 
      event.getEndTime(), 
      {search: event.getTitle()}
    );
    
    if (existingEvents.length === 0) {
      // Copy event to target calendar
      targetCalendar.createEvent(
        event.getTitle(),
        event.getStartTime(),
        event.getEndTime(),
        {
          description: event.getDescription(),
          location: event.getLocation()
        }
      );
    }
  });
}
```

### Event Report Generator

```javascript
function generateEventReport(calendarId, startDate, endDate) {
  var calendar = CalendarApp.getCalendarById(calendarId);
  var events = calendar.getEvents(startDate, endDate);
  
  var report = {
    totalEvents: events.length,
    totalHours: 0,
    eventsByType: {},
    attendeesList: new Set()
  };
  
  events.forEach(function(event) {
    // Calculate duration
    var duration = (event.getEndTime() - event.getStartTime()) / (1000 * 60 * 60);
    report.totalHours += duration;
    
    // Categorize events
    var eventType = event.getTitle().split(':')[0] || 'Other';
    report.eventsByType[eventType] = (report.eventsByType[eventType] || 0) + 1;
    
    // Collect attendees
    var guests = event.getGuestList();
    guests.forEach(function(guest) {
      report.attendeesList.add(guest.getEmail());
    });
  });
  
  report.uniqueAttendees = report.attendeesList.size;
  return report;
}
```

## Error Handling

```javascript
function safeCreateEvent(title, startTime, endTime, options) {
  try {
    var event = CalendarApp.createEvent(title, startTime, endTime, options);
    console.log('Event created successfully: ' + event.getId());
    return event;
  } catch (e) {
    console.error('Failed to create event: ' + e.toString());
    
    // Handle specific errors
    if (e.toString().includes('Invalid date')) {
      console.error('Please check the date format');
    } else if (e.toString().includes('Permission denied')) {
      console.error('You do not have permission to create events');
    }
    
    return null;
  }
}
```

## Best Practices

1. **Batch Operations**: When creating multiple events, consider using batch operations to improve performance.

2. **Time Zones**: Always be aware of time zone differences when creating events:
```javascript
var event = CalendarApp.createEvent('International Meeting',
  new Date('2024-01-15 15:00:00 GMT'),
  new Date('2024-01-15 16:00:00 GMT'));
event.setTimeZone('America/New_York');
```

3. **Guest Management**: Always check guest permissions before adding them:
```javascript
if (event.guestsCanInviteOthers()) {
  event.addGuest('additional@example.com');
}
```

4. **Event Cleanup**: Remove old events to keep calendars organized:
```javascript
function cleanupOldEvents(daysToKeep) {
  var cutoffDate = new Date();
  cutoffDate.setDate(cutoffDate.getDate() - daysToKeep);
  
  var oldEvents = CalendarApp.getEvents(new Date(2020, 0, 1), cutoffDate);
  oldEvents.forEach(function(event) {
    event.deleteEvent();
  });
}
```