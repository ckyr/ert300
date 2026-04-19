# ERT300 Converter — Changelog

---

## v1 — Converter only
**File:** `index_v1_converter.html`

Initial version. Single-screen DT → kg/lbs converter.

- DT stepper (+/−) from 20 to 65
- Head size selector: MID / MID+ / OVER
- Result card showing kg and lbs
- Playing style zone indicator (Lo / Mid / Hi / Hi+)
- Conversion table calibrated directly from ERT300 disc values (DT 26–53), full 28-point lookup per head size
- Out-of-range display for DT values outside 26–53
- iPhone home screen app meta tags (black-translucent status bar)
- Dark mode support

---

## v2 — My Rackets tab + localStorage
*(reconstructed — intermediate version)*

Added racket tracking with local browser storage.

- Two tabs: Converter / My Rackets
- Each racket stores: label name, racket model, head size, baseline DT, machine tension, stringing date, string + notes
- Tension % loss bar with colour-coded status (green / amber / red)
- Restring threshold warning at 20% tension loss
- DT history per racket with add / edit / delete
- Baseline entry shown as read-only with blue "baseline" badge
- History sorted newest-first
- localStorage persistence between sessions
- Date bug fix: dates parsed in local timezone to prevent off-by-one-day error

---

## v3 — Firebase cloud sync + Google Sign-in
*(reconstructed — intermediate version)*

Added Firebase authentication and Firestore cloud sync.

- Google Sign-in button in nav bar
- Signed-out mode: localStorage (amber dot indicator)
- Signed-in mode: Firestore real-time sync across all devices (green dot indicator)
- User data isolated by UID via Firestore security rules
- Modal width constrained to 430px on desktop browsers (centered overlay)
- Firebase config block with setup instructions in comments

---

## v4 — Stringing Jobs tab
**File:** `index_v4_string_library.html`

Added a third tab for logging stringing jobs as a reference database.

- Three tabs: My Rackets / Stringing Jobs / Converter
- Stringing Jobs stores: racket model, head size, string + gauge, machine tension, baseline DT, date, notes
- DT value shown as colour-coded badge matching zone colours
- Live search/filter by racket model or string name (fixed focus-loss bug on each keystroke)
- DT vs tension scatter chart with trend line (show/hide toggle)
- Chart filterable by racket model; auto-refreshes when filter changes
- Chart x-axis labelled "Tension (kg / lbs)" to accommodate both units
- Stringing Jobs data synced to Firestore under `users/{uid}/library/{id}`
- Firestore rules updated to `{collection}` wildcard covering all collections

---

## v5 — Linked rackets + stringing jobs data model
*(reconstructed — intermediate version)*

Major data model redesign eliminating double data entry.

- **My Rackets** becomes a pure equipment registry (name, model, head size only)
- **Stringing Jobs** is the single source of truth for all stringing data
- Each stringing job has an "Assign to racket" dropdown
- Assigning a job auto-fills and locks model + head size from the racket definition
- Racket card baseline DT, tension, string, and date all flow from the most recent assigned job automatically
- Editing a stringing job updates the linked racket card in real time
- Follow-up DT history preserved on job deletion/reassignment — filtered dynamically by baseline date, never deleted
- Unassigned jobs appear in Stringing Jobs and chart but not in My Rackets
- Stringing Jobs sorted by date descending (most recent first)
- Machine tension field changed to number-only input
- Firestore collections: `rackets` and `jobs`

---

## v6 — UI polish, legend redesign, DT colour coding
**File:** `index_v6_current.html` *(from previous zip — now superseded by v7)*

Visual consistency pass and quality-of-life improvements.

### Legend
- DT zone legend redesigned from 4-column compact tiles to 2-column layout with full descriptions
- Full zone descriptions: "DT 28–34 · Defensive baseline", "DT 35–41 · Dynamic & offensive", etc.
- Legend added to all three tabs (My Rackets, Stringing Jobs, Converter)
- Redundant "Playing style" zone pill removed from Converter tab
- Legend tiles in Converter highlight the active zone as DT is adjusted

### DT number colour
- Large DT number in Converter changes colour dynamically to match active zone
- DT 55 and 56 now correctly shown as red (Hi+); grey only at DT 57+
- Separated conversion table range (DT 26–53) from zone colour range (DT 28–56)

### Drag-to-reorder rackets
- Six-dot grip handle (⠿) on each racket card
- Desktop: drag any card to reorder; iPhone: press and hold grip handle
- Sort order persisted as `sortOrder` field on each racket document in Firestore
- Syncs across all devices; new rackets added at end of list

### iOS standalone fix
- Fixed layout issue where content rendered behind Dynamic Island
- Added `sb-spacer` div that measures actual safe area inset at runtime

### Other
- Placeholder text updated: "e.g. Racket 1", "e.g. Wilson Pro Staff"
- Chart filter bug fixed: chart auto-refreshes when racket model filter changes

---

## v7 — Responsive desktop + tablet layout
**File:** `index_v7_current.html`

Full responsive layout supporting mobile, tablet, and desktop with appropriate designs for each.

### Three-breakpoint responsive system

| Screen width | Layout |
|---|---|
| < 768px | Mobile — full width, top nav + tab bar, bottom sheet modals |
| 768px – 1289px | Tablet — fluid full-width, sidebar 25% / content 75% |
| 1290px+ | Desktop — fixed 430px sidebar / 860px content, centered with background |

### Desktop layout (1290px+)
- Fixed 1290px shell centered in the browser window
- Left sidebar (430px): app title, Google sign-in button, vertical tab navigation with dot + left-border active indicator
- Content area (860px): full content with generous padding
- Modals become centered dialogs instead of bottom sheets
- DT zone legend displays in 4 columns (one row) instead of 2
- Background colour outside the shell (light grey / dark charcoal) gives visual separation

### Tablet layout (768px – 1289px)
- Shell fills full browser window width
- Sidebar at 25% of window width, content at 75%
- Same sidebar navigation style as desktop
- Handles iPad portrait (768px), iPad landscape (1024px), and small laptops

### Mobile layout (< 768px) — unchanged
- Full-width shell up to 430px max
- Top nav bar + horizontal tab bar
- Bottom sheet modals with drag handle
- iOS standalone safe area fix for Dynamic Island devices

---

## Firebase setup summary

All versions from v3 onward require Firebase configuration. See `README.md` for full instructions.

**Firestore security rules** (covers all collections):
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{collection}/{docId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

**Collections used:**
- `users/{uid}/rackets` — racket registry entries, follow-up DT history, sort order
- `users/{uid}/jobs` — stringing job records

---

## DT conversion reference

Calibrated from ERT300 disc values. Valid range DT 26–53 for kg/lbs output.

| DT range | Zone | Playing style |
|----------|------|--------------|
| Below 28 | — | Out of range |
| 28–34 | Lo | Defensive baseline play |
| 35–41 | Mid | Dynamic and offensive |
| 42–46 | Hi | Fast and aggressive |
| 47–56 | Hi+ | Very fast and aggressive |
| Above 56 | — | Out of range |

---

## v8 — Settings panel, JSON backup/restore, Help tab, chart PNG button
**File:** `index_v8_current.html`

### Settings panel (⚙)
- Gear icon added to the nav bar next to the sign-in button
- Settings panel opens as a bottom sheet (mobile) or centered dialog (desktop)
- Three backup/restore options:
  1. **Export data (JSON)** — full backup of all rackets, jobs, DT history, sort order to a timestamped JSON file
  2. **Import data (JSON)** — merge backup into existing data; records with matching IDs are updated, new ones added; warning shown about Firestore taking precedence on sign-in
  3. **Restore from JSON (clean)** — deletes all existing data (locally and from Firestore if signed in), then imports backup fresh; no conflict possible; confirmation dialog with clear irreversibility warning

### Help tab
- Fourth tab added: **Help**
- Contains full in-app documentation including: how the app works, DT zones reference table, cloud sync instructions, settings guide, responsive layout reference, and home screen installation steps
- Replaces the need to refer to README.md for common questions

### Chart PNG export
- Removed the labelled "Save chart (PNG)" button from the bottom export bar
- Added a compact image icon button (🖼) directly in the filter row next to "Show chart / Hide chart"
- Saves the current chart view with the active racket filter reflected in the filename

### Cleanup
- Removed individual CSV export buttons from both My Rackets and Stringing Jobs tabs (full JSON export via Settings covers all data)
- Removed now-unused export bar CSS and button styles
- All data export/import consolidated in the Settings panel

### Responsive
- Settings panel and Help tab both respect all three breakpoints (mobile bottom sheet, desktop centered dialog)
- Desktop sidebar now shows 4 tabs: My Rackets, Stringing Jobs, Converter, Help

---

## v9 — App renamed, delete database, description update
**File:** `index_v9_current.html`

### App renamed
- Title: `ERT300 Converter` → `Tennis Stringing Companion`
- Nav bar title: `ERT300` → `Stringing`
- Nav bar subtitle: `String Tension Converter` → `Tennis Stringing Companion`
- iPhone home screen name: `ERT300` → `Stringing`
- Help section heading and description updated to reflect the app's primary focus on racket and job management, with the ERT300 converter as a supporting feature
- New description: *"Keeps records of your rackets, maintains a historical log of stringing jobs, tracks string tension over time, and converts ERT300 Dynamic Tension (DT) readings to kg/lbs."*

### Delete database
- New button added to Settings panel below "Restore from JSON (clean)"
- Red/danger styling matching the Restore button
- Trash can icon
- Deletes all rackets, jobs, and DT history — locally and from Firestore if signed in
- Confirmation dialog shows record counts and explicit "cannot be undone" warning
- Useful for clearing test data before beginning real use
- Documented in Help tab and README

---

## v10 — String autocomplete, string summary, nav title update
**File:** `index_v10_current.html`

### String autocomplete (datalist)
- String field in New Stringing Job modal now uses HTML `<datalist>` for autocomplete
- Dropdown suggestions populate automatically from all previously used string names in existing jobs — no setup needed, works immediately for existing users
- Suggestions appear as you type (prefix matching)
- Typing a new string not in the list is fully supported
- String field label updated from "String name + gauge" to "String (manufacturer — name — gauge)" with placeholder "e.g. Luxilon — ALU Power — 1.25" to encourage consistent naming format

### Strings used summary
- New "Strings used" button added below the "+ New stringing job" button in the Stringing Jobs tab
- Opens a modal listing all unique strings logged, sorted by usage count (most used first), with a count of jobs per string
- Useful quick reference without needing to scroll through all jobs

### App title / nav update
- iPhone home screen name updated from "Stringing" to "Tennis Stringing Companion"
- Nav bar simplified: removed separate subtitle line
- Title now reads "Tennis Stringing Companion" as a single element
- Mobile: single line, compact
- Desktop sidebar: wraps to two lines ("Tennis Stringing" / "Companion") with slightly larger font

---

## v11 — Date and time for stringing jobs and DT readings
**File:** `index_v11_current.html`

### Date + time capture
- All date fields changed from date-only (`YYYY-MM-DD`) to date + time (`datetime-local`, `YYYY-MM-DDTHH:MM`)
- Affected fields: stringing job date, DT history add row, DT history edit row
- All fields auto-populate with the current date and time when opened, editable if needed
- Time is displayed alongside the date in history rows (e.g. "Apr 14, 2026 · 2:35 PM")

### Backwards compatibility
- Legacy records with date-only strings (`YYYY-MM-DD`) are parsed as `00:00` local time
- All sorting and filtering works correctly against both old and new formats
- No re-entry of historical data required

### Same-day measurement fix (extends v10 fix)
- The previous `>` date filter (introduced in v10) now compares full datetime values
- Multiple measurements on the same day are correctly ordered by time
- Scenario now handled correctly: string racket at 10am (baseline DT = 40), measure at 2pm (DT = 38 after stabilisation) — both on the same day, correctly distinguished
