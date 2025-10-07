# DrupalCon Vienna 2025 - Custom Schedule Builder

## Project Overview

This is a privacy-first, single-page web application that allows DrupalCon Vienna 2025 attendees to build personalized conference schedules. The app runs entirely client-side with no backend, tracking, cookies, or data collection.

**Built by**: aboros (Drupal community member)  
**Not affiliated with**: DrupalCon Vienna 2025 (unofficial tool)

## Technology Stack

- **Frontend**: Vanilla JavaScript (ES6+), no frameworks
- **Styling**: Tailwind CSS (CDN)
- **Icons**: Font Awesome 6.0.0
- **Analytics**: Simple Analytics (privacy-first, optional)
- **Storage**: Browser localStorage only
- **Server**: Simple Python server (`server.py`) for local development

## File Structure

```
/
├── index.html              # Main SPA with embedded JavaScript
├── data/
│   └── events.json        # Event data (manually updated)
├── source/
│   ├── convert/
│   │   └── convert_schedule.py  # Script to parse source HTML
│   └── drupalcon-schedule-source-data.html  # Source data
├── server.py              # Simple HTTP server
├── requirements.txt       # Python dependencies (beautifulsoup4)
├── README.md             # Project documentation
└── venv/                 # Python virtual environment
```

## Core Features

### 1. Event Filtering

- **By Date**: Filter events by conference day (Oct 14-17, 2025)
- **By Track**: Filter by session track (drupal cms, coding & site building, etc.)
- **By Keywords**: Real-time search across title, description, track, location
- **By Selection Status**: Show all, selected only, or unselected only

### 2. Event Selection

- Click anywhere on event card to toggle selection
- Checkbox also toggles selection
- Selected events highlighted with Drupal blue theme
- Selection state persisted to localStorage
- Unsaved changes tracked and "Save" button enabled accordingly

### 3. Calendar Export

- **Download ICS**: Creates `.ics` file for selected events
- **Add to Calendar**: Opens calendar app with selected events
- **Timezone**: Europe/Vienna (CET/CEST) with proper VTIMEZONE definitions
- **Event Details**: Includes title, location, description, and session URL

### 4. Selection Overview Panel

- Fixed bottom panel that slides up when events are selected
- Shows total events and duration
- Expandable details with statistics by track and by day
- Only visible when user has selections

### 5. Privacy & Storage

- No server-side storage
- No cookies
- No user tracking (except optional Simple Analytics)
- All data stored in browser's localStorage under key: `drupalconSelectedEvents`

## Data Structure

### Event Object (from events.json)

```javascript
{
  "startTime": "2025-10-14T09:30:00",     // ISO 8601 local time (Vienna)
  "endTime": "2025-10-14T18:00:00",       // ISO 8601 local time (Vienna)
  "duration": "PT8H30M",                   // ISO 8601 duration format
  "summary": "Session Title",              // Event name
  "location": "Room Name",                 // Venue/room
  "description": "Speaker Name(s)",        // Speaker info
  "link": "https://...",                   // Session URL
  "track": "drupal cms"                    // Category/track
}
```

### Generated Properties

- `id`: Generated as `${startTime}-${location}-${summary}` with sanitization
- `clean_title`: Sanitized version of summary for internal use

### Track Categories

- `drupal cms`
- `coding & site building`
- `agency & business`
- `infosec & devops`
- `community health`
- `open web`
- `clients & industry experiences`
- `contribution topic`
- `sponsor talks`
- `keynote`
- `coffee & lunch break time`
- `bof` (Birds of a Feather)
- `other`

## Key JavaScript Architecture

### Global State

```javascript
window.selectedEvents      // Set of selected event IDs
window.lastSavedState      // Set of last saved state (for dirty checking)
window.allEvents           // Array of all events (reference)
```

### Core Functions

| Function | Purpose |
|----------|---------|
| `fetchEvents()` | Loads and processes events.json |
| `displayEvents(events)` | Renders event list |
| `filterEvents(events, filterName, skipAnalytics)` | Applies all active filters |
| `toggleEventSelection(eventId)` | Handles selection toggle |
| `generateIcsContent(events)` | Creates ICS calendar format |
| `updateSelectionOverview(events)` | Updates bottom panel stats |
| `saveSelections()` | Persists to localStorage |

### Utility Functions

| Function | Purpose |
|----------|---------|
| `formatDate(dateString)` | Human-readable date/time |
| `formatDuration(duration)` | Converts PT1H30M to "1 hour 30 minutes" |
| `highlightKeywords(text, keywords)` | Adds highlighting spans |
| `debounce(func, wait)` | Debounces rapid function calls |
| `areSetsEqual(set1, set2)` | Compares two Sets |

## Styling Conventions

### Drupal Brand Colors

- **Primary Blue**: `rgb(0, 106, 169)`
- **Hover Blue**: `rgb(0, 85, 135)`
- **Light Blue BG**: `rgb(240, 248, 255)`

### Custom CSS Classes

| Class | Purpose |
|-------|---------|
| `.drupal-blue` | Background color |
| `.drupal-blue-hover` | Hover state |
| `.drupal-blue-text` | Text color |
| `.drupal-blue-border` | Border color |
| `.drupal-blue-focus` | Focus state |
| `.drupal-blue-bg-light` | Light background |
| `.keyword-highlight` | Yellow highlight for search matches |

## Important Implementation Details

### ICS File Generation

- Uses **Europe/Vienna** timezone (TZID)
- Includes VTIMEZONE definition with DST rules
- Preserves local times (no UTC conversion)
- Escapes special characters in descriptions (commas, newlines)
- Includes session URL at start of description

### Keyword Search

- Searches across: `summary`, `description`, `track`, `location`
- Case-insensitive matching
- Highlights matches in real-time
- Debounced for analytics (2 second delay)
- Immediate filtering for UX responsiveness

### LocalStorage Schema

```javascript
// Key: 'drupalconSelectedEvents'
// Value: JSON array of event IDs
["eventId1", "eventId2", ...]
```

## Analytics Events (Simple Analytics)

| Event Name | Trigger |
|------------|---------|
| `addSession` | When event selected |
| `removeSession` | When event deselected |
| `addToTrack` | Track added to selection |
| `removeFromTrack` | Track removed from selection |
| `dateFilter` | Date filter changed |
| `trackFilter` | Track filter changed |
| `keywordsFilter` | Keywords entered (debounced) |
| `selectionFilter` | Selection filter changed |
| `download_ics` | ICS file downloaded |
| `add_to_calendar` | Add to calendar clicked |
| `save_selections` | Selections saved |
| `selection_details_opened` | Details panel expanded |
| `selection_details_closed` | Details panel collapsed |

## Development Workflow

### Data Updates

1. Source HTML placed in `source/drupalcon-schedule-source-data.html`
2. Run `source/convert/convert_schedule.py` to parse and generate JSON
3. Output written to `data/events.json`
4. Commit both source and generated files

### Local Testing

```bash
python server.py
# Then open http://localhost:8000
```

## Code Quality Guidelines

### When Modifying This Project

1. ✅ **No dependencies**: Keep vanilla JS, no npm/build process
2. 🔒 **Privacy first**: No tracking, no external API calls (except CDNs)
3. ♿ **Accessibility**: Maintain ARIA labels, keyboard navigation
4. ⚡ **Performance**: Debounce expensive operations, use event delegation
5. 📱 **Mobile-first**: Ensure responsive design works on all devices
6. 🎨 **Drupal branding**: Use official Drupal blue colors
7. 📊 **Analytics**: Use Simple Analytics event naming convention

### Testing Checklist

- [ ] Filter combinations work correctly
- [ ] LocalStorage saves/loads properly
- [ ] ICS files download with correct timezone
- [ ] Selection overview shows accurate statistics
- [ ] Keyword highlighting works across all fields
- [ ] Responsive design works on mobile
- [ ] Clear button appears/disappears correctly
- [ ] Unsaved changes detection works

## Known Limitations

- Client-side only (requires browser with JavaScript enabled)
- No offline support (requires internet for CDN resources)
- No user accounts or cloud sync
- Manual data updates required
- No conflict detection for overlapping sessions

## Future Enhancement Ideas (Not Implemented)

- Conflict detection for overlapping time slots
- Share schedule via URL
- Multiple schedule profiles
- Print-friendly view
- Offline PWA support
- Dark mode toggle

