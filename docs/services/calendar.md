# Google Apps Script Calendar Service API Reference

[← Back to Main Documentation](../README.md) | [View Examples →](../examples/calendar-examples.md)

## Overview
This service allows a script to access and modify the user's Google Calendar, including additional calendars that the user is subscribed to.

## Classes

### Calendar
Represents a calendar that the user owns or is subscribed to.

Methods:
- `createAllDayEvent(title, date)` → CalendarEvent - Creates a new all-day event
- `createAllDayEvent(title, startDate, endDate)` → CalendarEvent - Creates a new all-day event that can span multiple days
- `createAllDayEvent(title, startDate, endDate, options)` → CalendarEvent - Creates a new all-day event that can span multiple days
- `createAllDayEvent(title, date, options)` → CalendarEvent - Creates a new all-day event
- `createAllDayEventSeries(title, startDate, recurrence)` → CalendarEventSeries - Creates a new all-day event series
- `createAllDayEventSeries(title, startDate, recurrence, options)` → CalendarEventSeries - Creates a new all-day event series
- `createEvent(title, startTime, endTime)` → CalendarEvent - Creates a new event
- `createEvent(title, startTime, endTime, options)` → CalendarEvent - Creates a new event
- `createEventFromDescription(description)` → CalendarEvent - Creates an event from a free-form description
- `createEventSeries(title, startTime, endTime, recurrence)` → CalendarEventSeries - Creates a new event series
- `createEventSeries(title, startTime, endTime, recurrence, options)` → CalendarEventSeries - Creates a new event series
- `deleteCalendar()` → void - Deletes the calendar permanently
- `getColor()` → String - Gets the color of the calendar
- `getDescription()` → String - Gets the description of the calendar
- `getEventById(iCalId)` → CalendarEvent - Gets the event with the given ID
- `getEventSeriesById(iCalId)` → CalendarEventSeries - Gets the event series with the given ID
- `getEvents(startTime, endTime)` → CalendarEvent[] - Gets all events that occur within a given time range
- `getEvents(startTime, endTime, options)` → CalendarEvent[] - Gets all events that occur within a given time range and meet the specified criteria
- `getEventsForDay(date)` → CalendarEvent[] - Gets all events that occur on a given day
- `getEventsForDay(date, options)` → CalendarEvent[] - Gets all events that occur on a given day and meet specified criteria
- `getId()` → String - Gets the ID of the calendar
- `getName()` → String - Gets the name of the calendar
- `getTimeZone()` → String - Gets the time zone of the calendar
- `isHidden()` → Boolean - Determines whether the calendar is hidden in the user interface
- `isMyPrimaryCalendar()` → Boolean - Determines whether the calendar is the primary calendar for the effective user
- `isOwnedByMe()` → Boolean - Determines whether the calendar is owned by you
- `isSelected()` → Boolean - Determines whether the calendar's events are displayed in the user interface
- `setColor(color)` → Calendar - Sets the color of the calendar
- `setDescription(description)` → Calendar - Sets the description of a calendar
- `setHidden(hidden)` → Calendar - Sets whether the calendar is visible in the user interface
- `setName(name)` → Calendar - Sets the name of the calendar
- `setSelected(selected)` → Calendar - Sets whether the calendar's events are displayed in the user interface
- `setTimeZone(timeZone)` → Calendar - Sets the time zone of the calendar
- `unsubscribeFromCalendar()` → void - Unsubscribes the user from a calendar

### CalendarApp
Main entry point for Calendar operations. Allows a script to read and update the user's Google Calendar.

Properties:
- `Color` - An enum representing the named colors available in the Calendar service
- `EventColor` - An enum representing the named event colors available in the Calendar service
- `EventTransparency` - The EventTransparency enumeration
- `EventType` - The EventType enumeration
- `GuestStatus` - An enum representing the statuses a guest can have for an event
- `Month` - An enum representing the months of the year
- `Visibility` - An enum representing the visibility of an event
- `Weekday` - An enum representing the days of the week

Methods:
- `createAllDayEvent(title, date)` → CalendarEvent - Creates a new all-day event
- `createAllDayEvent(title, startDate, endDate)` → CalendarEvent - Creates a new all-day event that can span multiple days
- `createAllDayEvent(title, startDate, endDate, options)` → CalendarEvent - Creates a new all-day event that can span multiple days
- `createAllDayEvent(title, date, options)` → CalendarEvent - Creates a new all-day event
- `createAllDayEventSeries(title, startDate, recurrence)` → CalendarEventSeries - Creates a new all-day event series
- `createAllDayEventSeries(title, startDate, recurrence, options)` → CalendarEventSeries - Creates a new all-day event series
- `createCalendar(name)` → Calendar - Creates a new calendar, owned by the user
- `createCalendar(name, options)` → Calendar - Creates a new calendar, owned by the user
- `createEvent(title, startTime, endTime)` → CalendarEvent - Creates a new event
- `createEvent(title, startTime, endTime, options)` → CalendarEvent - Creates a new event
- `createEventFromDescription(description)` → CalendarEvent - Creates an event from a free-form description
- `createEventSeries(title, startTime, endTime, recurrence)` → CalendarEventSeries - Creates a new event series
- `createEventSeries(title, startTime, endTime, recurrence, options)` → CalendarEventSeries - Creates a new event series
- `getAllCalendars()` → Calendar[] - Gets all calendars that the user owns or is subscribed to
- `getAllOwnedCalendars()` → Calendar[] - Gets all calendars that the user owns
- `getCalendarById(id)` → Calendar - Gets the calendar with the given ID
- `getCalendarsByName(name)` → Calendar[] - Gets all calendars with a given name that the user owns or is subscribed to
- `getColor()` → String - Gets the color of the calendar
- `getDefaultCalendar()` → Calendar - Gets the user's default calendar
- `getDescription()` → String - Gets the description of the calendar
- `getEventById(iCalId)` → CalendarEvent - Gets the event with the given ID
- `getEventSeriesById(iCalId)` → CalendarEventSeries - Gets the event series with the given ID
- `getEvents(startTime, endTime)` → CalendarEvent[] - Gets all events that occur within a given time range
- `getEvents(startTime, endTime, options)` → CalendarEvent[] - Gets all events that occur within a given time range and meet the specified criteria
- `getEventsForDay(date)` → CalendarEvent[] - Gets all events that occur on a given day
- `getEventsForDay(date, options)` → CalendarEvent[] - Gets all events that occur on a given day and meet specified criteria
- `getId()` → String - Gets the ID of the calendar
- `getName()` → String - Gets the name of the calendar
- `getOwnedCalendarById(id)` → Calendar - Gets the calendar with the given ID, if the user owns it
- `getOwnedCalendarsByName(name)` → Calendar[] - Gets all calendars with a given name that the user owns
- `getTimeZone()` → String - Gets the time zone of the calendar
- `isHidden()` → Boolean - Determines whether the calendar is hidden in the user interface
- `isMyPrimaryCalendar()` → Boolean - Determines whether the calendar is the primary calendar for the effective user
- `isOwnedByMe()` → Boolean - Determines whether the calendar is owned by you
- `isSelected()` → Boolean - Determines whether the calendar's events are displayed in the user interface
- `newRecurrence()` → EventRecurrence - Creates a new recurrence object, which can be used to create rules for event recurrence
- `setColor(color)` → Calendar - Sets the color of the calendar
- `setDescription(description)` → Calendar - Sets the description of a calendar
- `setHidden(hidden)` → Calendar - Sets whether the calendar is visible in the user interface
- `setName(name)` → Calendar - Sets the name of the calendar
- `setSelected(selected)` → Calendar - Sets whether the calendar's events are displayed in the user interface
- `setTimeZone(timeZone)` → Calendar - Sets the time zone of the calendar
- `subscribeToCalendar(id)` → Calendar - Subscribes the user to the calendar with the given ID, if the user is allowed to subscribe
- `subscribeToCalendar(id, options)` → Calendar - Subscribes the user to the calendar with the given ID, if the user is allowed to subscribe

### CalendarEvent
Represents a single calendar event.

Methods:
- `addEmailReminder(minutesBefore)` → CalendarEvent - Adds a new email reminder to the event
- `addGuest(email)` → CalendarEvent - Adds a guest to the event
- `addPopupReminder(minutesBefore)` → CalendarEvent - Adds a new pop-up notification to the event
- `addSmsReminder(minutesBefore)` → CalendarEvent - Adds a new SMS reminder to the event
- `anyoneCanAddSelf()` → Boolean - Determines whether people can add themselves as guests to a Calendar event
- `deleteEvent()` → void - Deletes a Calendar event
- `deleteTag(key)` → CalendarEvent - Deletes a key/value tag from the event
- `getAllDayEndDate()` → Date - Gets the date on which this all-day calendar event ends
- `getAllDayStartDate()` → Date - Gets the date on which this all-day calendar event begins
- `getAllTagKeys()` → String[] - Gets all keys for tags that have been set on the event
- `getColor()` → String - Returns the color of the calendar event
- `getCreators()` → String[] - Gets the creators of an event
- `getDateCreated()` → Date - Gets the date the event was created
- `getDescription()` → String - Gets the description of the event
- `getEmailReminders()` → Integer[] - Gets the minute values for all email reminders for the event
- `getEndTime()` → Date - Gets the date and time at which this calendar event ends
- `getEventSeries()` → CalendarEventSeries - Gets the series of recurring events that this event belongs to
- `getEventType()` → EventType - Gets the EventType of this event
- `getGuestByEmail(email)` → EventGuest - Gets a guest by email address
- `getGuestList()` → EventGuest[] - Gets the guests for the event, not including the event owner
- `getGuestList(includeOwner)` → EventGuest[] - Gets the guests for the event, potentially including the event owners
- `getId()` → String - Gets the unique iCalUID of the event
- `getLastUpdated()` → Date - Gets the date the event was last updated
- `getLocation()` → String - Gets the location of the event
- `getMyStatus()` → GuestStatus - Gets the event status (such as attending or invited) of the effective user
- `getOriginalCalendarId()` → String - Get the ID of the calendar where this event was originally created
- `getPopupReminders()` → Integer[] - Gets the minute values for all pop-up reminders for the event
- `getSmsReminders()` → Integer[] - Gets the minute values for all SMS reminders for the event
- `getStartTime()` → Date - Gets the date and time at which this calendar event begins
- `getTag(key)` → String - Gets a tag value of the event
- `getTitle()` → String - Gets the title of the event
- `getTransparency()` → EventTransparency - Gets the transparency of the event
- `getVisibility()` → Visibility - Gets the visibility of the event
- `guestsCanInviteOthers()` → Boolean - Determines whether guests can invite other guests
- `guestsCanModify()` → Boolean - Determines whether guests can modify the event
- `guestsCanSeeGuests()` → Boolean - Determines whether guests can see other guests
- `isAllDayEvent()` → Boolean - Determines whether this is an all-day event
- `isOwnedByMe()` → Boolean - Determines whether you're the owner of the event
- `isRecurringEvent()` → Boolean - Determines whether the event is part of an event series
- `removeAllReminders()` → CalendarEvent - Removes all reminders from the event
- `removeGuest(email)` → CalendarEvent - Removes a guest from the event
- `resetRemindersToDefault()` → CalendarEvent - Resets the reminders using the calendar's default settings
- `setAllDayDate(date)` → CalendarEvent - Sets the date of the event
- `setAllDayDates(startDate, endDate)` → CalendarEvent - Sets the dates of the event
- `setAnyoneCanAddSelf(anyoneCanAddSelf)` → CalendarEvent - Sets whether non-guests can add themselves to the event
- `setColor(color)` → CalendarEvent - Sets the color of the calendar event
- `setDescription(description)` → CalendarEvent - Sets the description of the event
- `setGuestsCanInviteOthers(guestsCanInviteOthers)` → CalendarEvent - Sets whether guests can invite other guests
- `setGuestsCanModify(guestsCanModify)` → CalendarEvent - Sets whether guests can modify the event
- `setGuestsCanSeeGuests(guestsCanSeeGuests)` → CalendarEvent - Sets whether guests can see other guests
- `setLocation(location)` → CalendarEvent - Sets the location of the event
- `setMyStatus(status)` → CalendarEvent - Sets the event status (such as attending or invited) of the effective user
- `setTag(key, value)` → CalendarEvent - Sets a key/value tag on the event, for storing custom metadata
- `setTime(startTime, endTime)` → CalendarEvent - Sets the dates and times for the start and end of the event
- `setTitle(title)` → CalendarEvent - Sets the title of the event
- `setTransparency(transparency)` → CalendarEvent - Sets the transparency of the event
- `setVisibility(visibility)` → CalendarEvent - Sets the visibility of the event

### CalendarEventSeries
Represents a series of events (a recurring event).

Methods:
- `addEmailReminder(minutesBefore)` → CalendarEventSeries - Adds a new email reminder to the event
- `addGuest(email)` → CalendarEventSeries - Adds a guest to the event
- `addPopupReminder(minutesBefore)` → CalendarEventSeries - Adds a new pop-up notification to the event
- `addSmsReminder(minutesBefore)` → CalendarEventSeries - Adds a new SMS reminder to the event
- `anyoneCanAddSelf()` → Boolean - Determines whether people can add themselves as guests to a Calendar event
- `deleteEventSeries()` → void - Deletes the event series
- `deleteTag(key)` → CalendarEventSeries - Deletes a key/value tag from the event
- `getAllTagKeys()` → String[] - Gets all keys for tags that have been set on the event
- `getColor()` → String - Returns the color of the calendar event
- `getCreators()` → String[] - Gets the creators of an event
- `getDateCreated()` → Date - Gets the date the event was created
- `getDescription()` → String - Gets the description of the event
- `getEmailReminders()` → Integer[] - Gets the minute values for all email reminders for the event
- `getEventType()` → EventType - Gets the EventType of this event
- `getGuestByEmail(email)` → EventGuest - Gets a guest by email address
- `getGuestList()` → EventGuest[] - Gets the guests for the event, not including the event owner
- `getGuestList(includeOwner)` → EventGuest[] - Gets the guests for the event, potentially including the event owners
- `getId()` → String - Gets the unique iCalUID of the event
- `getLastUpdated()` → Date - Gets the date the event was last updated
- `getLocation()` → String - Gets the location of the event
- `getMyStatus()` → GuestStatus - Gets the event status (such as attending or invited) of the effective user
- `getOriginalCalendarId()` → String - Get the ID of the calendar where this event was originally created
- `getPopupReminders()` → Integer[] - Gets the minute values for all pop-up reminders for the event
- `getSmsReminders()` → Integer[] - Gets the minute values for all SMS reminders for the event
- `getTag(key)` → String - Gets a tag value of the event
- `getTitle()` → String - Gets the title of the event
- `getTransparency()` → EventTransparency - Gets the transparency of the event
- `getVisibility()` → Visibility - Gets the visibility of the event
- `guestsCanInviteOthers()` → Boolean - Determines whether guests can invite other guests
- `guestsCanModify()` → Boolean - Determines whether guests can modify the event
- `guestsCanSeeGuests()` → Boolean - Determines whether guests can see other guests
- `isOwnedByMe()` → Boolean - Determines whether you're the owner of the event
- `removeAllReminders()` → CalendarEventSeries - Removes all reminders from the event
- `removeGuest(email)` → CalendarEventSeries - Removes a guest from the event
- `resetRemindersToDefault()` → CalendarEventSeries - Resets the reminders using the calendar's default settings
- `setAnyoneCanAddSelf(anyoneCanAddSelf)` → CalendarEventSeries - Sets whether non-guests can add themselves to the event
- `setColor(color)` → CalendarEventSeries - Sets the color of the calendar event
- `setDescription(description)` → CalendarEventSeries - Sets the description of the event
- `setGuestsCanInviteOthers(guestsCanInviteOthers)` → CalendarEventSeries - Sets whether guests can invite other guests
- `setGuestsCanModify(guestsCanModify)` → CalendarEventSeries - Sets whether guests can modify the event
- `setGuestsCanSeeGuests(guestsCanSeeGuests)` → CalendarEventSeries - Sets whether guests can see other guests
- `setLocation(location)` → CalendarEventSeries - Sets the location of the event
- `setMyStatus(status)` → CalendarEventSeries - Sets the event status (such as attending or invited) of the effective user
- `setRecurrence(recurrence, startDate)` → CalendarEventSeries - Sets the recurrence rules for an all-day event series
- `setRecurrence(recurrence, startTime, endTime)` → CalendarEventSeries - Sets the recurrence rules for this event series
- `setTag(key, value)` → CalendarEventSeries - Sets a key/value tag on the event, for storing custom metadata
- `setTitle(title)` → CalendarEventSeries - Sets the title of the event
- `setTransparency(transparency)` → CalendarEventSeries - Sets the transparency of the event
- `setVisibility(visibility)` → CalendarEventSeries - Sets the visibility of the event

### EventGuest
Represents a guest of an event.

Methods:
- `getAdditionalGuests()` → Integer - Gets the number of additional people that this guest has said are attending
- `getEmail()` → String - Gets the email address of the guest
- `getGuestStatus()` → GuestStatus - Gets the status of the guest for the event
- `getName()` → String - Gets the name of the guest

### EventRecurrence
Represents the recurrence settings for an event series.

Methods:
- `addDailyExclusion()` → RecurrenceRule - Adds a rule that excludes occurrences on a daily basis
- `addDailyRule()` → RecurrenceRule - Adds a rule that causes the event to recur on a daily basis
- `addDate(date)` → EventRecurrence - Adds a rule that causes the event to recur on a specific date
- `addDateExclusion(date)` → EventRecurrence - Adds a rule that excludes an occurrence for a specific date
- `addMonthlyExclusion()` → RecurrenceRule - Adds a rule that excludes occurrences on a monthly basis
- `addMonthlyRule()` → RecurrenceRule - Adds a rule that causes the event to recur on a monthly basis
- `addWeeklyExclusion()` → RecurrenceRule - Adds a rule that excludes occurrences on a weekly basis
- `addWeeklyRule()` → RecurrenceRule - Adds a rule that causes the event to recur on a weekly basis
- `addYearlyExclusion()` → RecurrenceRule - Adds a rule that excludes occurrences on a yearly basis
- `addYearlyRule()` → RecurrenceRule - Adds a rule that causes the event to recur on a yearly basis
- `setTimeZone(timeZone)` → EventRecurrence - Sets the time zone for this recurrence

### RecurrenceRule
Represents a recurrence rule for an event series.

Methods:
- `addDailyExclusion()` → RecurrenceRule - Adds a rule that excludes occurrences on a daily basis
- `addDailyRule()` → RecurrenceRule - Adds a rule that causes the event to recur on a daily basis
- `addDate(date)` → EventRecurrence - Adds a rule that causes the event to recur on a specific date
- `addDateExclusion(date)` → EventRecurrence - Adds a rule that excludes an occurrence for a specific date
- `addMonthlyExclusion()` → RecurrenceRule - Adds a rule that excludes occurrences on a monthly basis
- `addMonthlyRule()` → RecurrenceRule - Adds a rule that causes the event to recur on a monthly basis
- `addWeeklyExclusion()` → RecurrenceRule - Adds a rule that excludes occurrences on a weekly basis
- `addWeeklyRule()` → RecurrenceRule - Adds a rule that causes the event to recur on a weekly basis
- `addYearlyExclusion()` → RecurrenceRule - Adds a rule that excludes occurrences on a yearly basis
- `addYearlyRule()` → RecurrenceRule - Adds a rule that causes the event to recur on a yearly basis
- `interval(interval)` → RecurrenceRule - Configures the rule to only apply at this interval of the rule's time unit
- `onlyInMonth(month)` → RecurrenceRule - Configures the rule to only apply to a specific month
- `onlyInMonths(months)` → RecurrenceRule - Configures the rule to only apply to specific months
- `onlyOnMonthDay(day)` → RecurrenceRule - Configures the rule to only apply to a specific day of the month
- `onlyOnMonthDays(days)` → RecurrenceRule - Configures the rule to only apply to specific days of the month
- `onlyOnWeek(week)` → RecurrenceRule - Configures the rule to only apply to a specific week of the year
- `onlyOnWeekday(day)` → RecurrenceRule - Configures the rule to only apply to a specific day of the week
- `onlyOnWeekdays(days)` → RecurrenceRule - Configures the rule to only apply to specific days of the week
- `onlyOnWeeks(weeks)` → RecurrenceRule - Configures the rule to only apply to specific weeks of the year
- `onlyOnYearDay(day)` → RecurrenceRule - Configures the rule to only apply to a specific day of the year
- `onlyOnYearDays(days)` → RecurrenceRule - Configures the rule to only apply to specific days of the year
- `setTimeZone(timeZone)` → EventRecurrence - Sets the time zone for this recurrence
- `times(times)` → RecurrenceRule - Configures the rule to end after a given number of occurrences
- `until(endDate)` → RecurrenceRule - Configures the rule to end on a given date (inclusive)
- `weekStartsOn(day)` → RecurrenceRule - Configures which day a week starts on, for the purposes of applying the rule

## Enumerations

### Color
Named colors available in the Calendar service:
- BLUE (#2952A3)
- BROWN (#8D6F47)
- CHARCOAL (#4E5D6C)
- CHESTNUT (#865A5A)
- GRAY (#5A6986)
- GREEN (#0D7813)
- INDIGO (#5229A3)
- LIME (#528800)
- MUSTARD (#88880E)
- OLIVE (#6E6E41)
- ORANGE (#BE6D00)
- PINK (#B1365F)
- PLUM (#705770)
- PURPLE (#7A367A)
- RED (#A32929)
- RED_ORANGE (#B1440E)
- SEA_BLUE (#29527A)
- SLATE (#4A716C)
- TEAL (#28754E)
- TURQOISE (#1B887A)
- YELLOW (#AB8B00)

### EventColor
Named event colors available in the Calendar service:
- PALE_BLUE ("1") - "Peacock" in Calendar UI
- PALE_GREEN ("2") - "Sage" in Calendar UI
- MAUVE ("3") - "Grape" in Calendar UI
- PALE_RED ("4") - "Flamingo" in Calendar UI
- YELLOW ("5") - "Banana" in Calendar UI
- ORANGE ("6") - "Tangerine" in Calendar UI
- CYAN ("7") - "Lavender" in Calendar UI
- GRAY ("8") - "Graphite" in Calendar UI
- BLUE ("9") - "Blueberry" in Calendar UI
- GREEN ("10") - "Basil" in Calendar UI
- RED ("11") - "Tomato" in Calendar UI

### EventTransparency
- OPAQUE - The event does block time on the calendar
- TRANSPARENT - The event does not block time on the calendar

### EventType
- DEFAULT - The event is a regular event
- BIRTHDAY - The event is a special all-day event with an annual recurrence
- FOCUS_TIME - The event is a focus-time event
- FROM_GMAIL - The event is an event from Gmail
- OUT_OF_OFFICE - The event is an out-of-office event
- WORKING_LOCATION - The event is a working location event

### GuestStatus
- INVITED - The guest has been invited, but has not indicated whether they are attending
- MAYBE - The guest has indicated they might attend
- NO - The guest has indicated they are not attending
- OWNER - The guest is the owner of the event
- YES - The guest has indicated they are attending

### Visibility
- CONFIDENTIAL - The event is private
- DEFAULT - Uses the default visibility for events on the calendar
- PRIVATE - The event is private and only event attendees may view event details
- PUBLIC - The event is public and event details are visible to all readers of the calendar