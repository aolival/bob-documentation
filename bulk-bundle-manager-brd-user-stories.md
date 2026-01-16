# Business Requirements Document (BRD)
## Bulk Bundle Manager - User Stories

---

## Document Information

**Project Name**: BoB (Bundle of Bundles) - Bulk Bundle Manager
**Document Type**: Business Requirements Document (BRD)
**Version**: 1.0
**Created Date**: 2025-01-19
**Last Updated**: 2025-01-19
**Status**: Draft - Pending Review
**Product Owner**: TBD
**Business Analyst**: TBD
**Development Team**: TBD

---

## Table of Contents

1. [User Story 1: Manual Loan Entry & Summary Generation](#user-story-1)
2. [User Story 2: Bulk CSV Upload & Loan Parsing](#user-story-2)
3. [User Story 3: Loan Summary Grid Display & Filtering](#user-story-3)
4. [User Story 4: Bundle Build Execution](#user-story-4)
5. [User Story 5: Dashboard History & Bundle Review](#user-story-5)
6. [User Story 6: Error Handling & Validation](#user-story-6)
7. [User Story 7: Refresh Summary & Add Individual Loans](#user-story-7)
8. [User Story 8: Build Status Tracking & Results Display](#user-story-8)
9. [User Story 9: User Authentication & Session Management](#user-story-9)
10. [User Story 10: EPS/BytePro API Integration](#user-story-10)

---

<a name="user-story-1"></a>
## User Story 1: Manual Loan Entry & Summary Generation

### Story ID
**BRD-BULK-001**

### Story Title
Manual Loan Entry & Bulk Summary Generation

### User Story Statement
**As a** loan processor
**I want to** manually enter multiple loan numbers and generate a document summary
**So that** I can quickly see which loans are ready for bundling without uploading a file

---

### BRD Summary

#### Overview
This feature allows users to manually enter loan numbers into a text area and generate a bulk document summary that displays the document status for each loan. This is the primary entry method for creating bundles when users have a small list of loans or want to quickly check status without preparing a CSV file.

#### Business Value
- **Efficiency**: Users can check 10-50 loans in seconds instead of manually looking up each loan
- **Flexibility**: No need to prepare CSV files for small batches
- **User Experience**: Simple text-based entry familiar to users
- **Time Savings**: Eliminates manual document counting and validation

#### Scope
- Manual text entry of loan numbers (comma-separated or line-separated)
- Real-time validation of loan number format
- Duplicate detection and automatic removal
- API integration to retrieve document status for all loans
- Display of summary grid with found/missing document counts

---

### Detailed Requirements

#### Functional Requirements

**FR-001: Bundle Name Selection**
- System MUST provide a dropdown list of available bundle names
- Dropdown MUST contain all investor bundle types (100+ options)
- Bundle names MUST be sorted alphabetically
- System MUST require bundle name selection before generating summary
- Selected bundle name MUST determine stacking order used for validation

**FR-002: Text Area for Manual Entry**
- System MUST provide a text area for manual loan number entry
- Text area MUST accept multiple formats:
  - Line-separated: One loan per line
  - Comma-separated: Loans separated by commas
  - Space-separated: Loans separated by spaces
  - Mixed format: Combination of above
- Text area MUST support copy-paste from Excel/spreadsheet
- Text area MUST display character/line count

**FR-003: Optional Metadata Fields**
- System MUST provide optional "Investor Number" field
- System MUST provide optional "Trade Number" field
- Both fields MUST accept alphanumeric input only
- Fields MUST be included in bundle metadata but not required for processing

**FR-004: Loan Number Validation**
- System MUST validate each loan number before API call
- Validation rules:
  - Alphanumeric characters only
  - No special characters except hyphens
  - Length between 5-20 characters
  - Not empty/whitespace only
- System MUST trim whitespace from all loan numbers
- System MUST convert loan numbers to consistent format (uppercase)

**FR-005: Duplicate Detection**
- System MUST identify duplicate loan numbers
- System MUST automatically remove duplicates
- System MUST display warning message: "X duplicate loan numbers removed"
- System MUST keep only first occurrence of duplicate

**FR-006: Summary Generation**
- User clicks "Generate Bulk Bundle Summary" button
- System validates all inputs
- System locks all input fields (prevents editing during processing)
- System displays loading indicator: "Loading from EPS API..."
- System calls EPS API with loan number array
- System receives document status for each loan
- System displays summary grid (see User Story 3)

**FR-007: Clear Functionality**
- System MUST provide "Clear" button
- Button clears all fields and resets form
- Button unlocks all fields for new entry
- Button available at all times (even during loading)

---

#### Technical Requirements

**TR-001: Input Processing**
- Parse text area input and split by delimiters (newline, comma, space)
- Remove empty strings from array
- Normalize loan numbers (trim, uppercase, remove extra whitespace)
- Store processed loan numbers in state array

**TR-002: Validation Logic**
- Implement regex validation: `/^[A-Z0-9-]{5,20}$/i`
- Implement duplicate detection using Set or array comparison
- Display validation errors inline (below text area)
- Prevent API call if validation fails

**TR-003: API Request Format**
```json
{
  "bundleName": "U.S. Bank",
  "loanNumbers": ["USB12345", "USB12346", ...],
  "investorNumber": "INV001",
  "tradeNumber": "TRADE123",
  "authToken": "Bearer <token>",
  "requestTimestamp": "2025-01-19T10:30:00Z"
}
```

**TR-004: State Management**
- Store bundle name in state
- Store loan numbers array in state
- Store validation errors in state
- Store loading status in state
- Lock fields by setting `fieldsLocked` state to true

**TR-005: Performance**
- Support up to 100 loans in single request
- Display warning if more than 100 loans entered
- Debounce text area input (300ms) to prevent excessive re-renders

---

### Acceptance Criteria

#### AC-001: Bundle Name Selection
- [ ] Given I am on Bundle Builder tab
- [ ] When I click Bundle Name dropdown
- [ ] Then I see a list of 100+ bundle options sorted alphabetically
- [ ] And I can select one bundle name
- [ ] And selected bundle name appears in dropdown field

#### AC-002: Manual Loan Entry
- [ ] Given I have selected a bundle name
- [ ] When I type loan numbers into text area (one per line)
- [ ] Then system accepts my input
- [ ] And I can type or paste up to 100 loan numbers
- [ ] And text area expands to accommodate content

#### AC-003: Mixed Format Support
- [ ] Given I have selected a bundle name
- [ ] When I paste comma-separated loan numbers
- [ ] Then system correctly parses all loan numbers
- [ ] When I paste line-separated loan numbers
- [ ] Then system correctly parses all loan numbers
- [ ] When I paste mixed format (commas and newlines)
- [ ] Then system correctly parses all loan numbers

#### AC-004: Validation Success
- [ ] Given I have entered 10 valid loan numbers
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then system validates all loan numbers successfully
- [ ] And no validation errors appear
- [ ] And system locks all input fields
- [ ] And loading indicator appears
- [ ] And API call is initiated

#### AC-005: Validation Failure - Empty Bundle Name
- [ ] Given I have entered loan numbers but NOT selected bundle name
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then system displays error: "Please select a bundle name first"
- [ ] And fields remain unlocked
- [ ] And no API call is made

#### AC-006: Validation Failure - No Loan Numbers
- [ ] Given I have selected bundle name but entered no loan numbers
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then system displays error: "Please enter at least one loan number"
- [ ] And fields remain unlocked
- [ ] And no API call is made

#### AC-007: Validation Failure - Invalid Format
- [ ] Given I have entered loan numbers with special characters (!@#$%)
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then system displays error: "Invalid loan number format: [list of invalid entries]"
- [ ] And fields remain unlocked
- [ ] And no API call is made

#### AC-008: Duplicate Detection
- [ ] Given I have entered loan numbers with 3 duplicates
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then system displays warning: "3 duplicate loan numbers removed"
- [ ] And system proceeds with unique loan numbers only
- [ ] And duplicates are automatically removed from processing

#### AC-009: Optional Fields
- [ ] Given I have selected bundle name and entered loan numbers
- [ ] When I click "Generate Bulk Bundle Summary" WITHOUT entering Investor/Trade numbers
- [ ] Then system processes successfully
- [ ] And optional fields are not required

#### AC-010: Clear Button
- [ ] Given I have entered data in all fields
- [ ] When I click "Clear" button
- [ ] Then all fields reset to empty
- [ ] And bundle name dropdown resets to placeholder
- [ ] And text area clears
- [ ] And all fields unlock for new entry

---

### Dependencies
- User Story 3: Loan Summary Grid Display (displays results)
- User Story 6: Error Handling & Validation (handles errors)
- User Story 10: EPS/BytePro API Integration (retrieves document status)

---

### Risks & Mitigations
- **Risk**: Users paste extremely large loan lists (500+ loans)
  - **Mitigation**: Display warning at 100 loans, suggest bulk upload method
- **Risk**: Invalid loan number formats from different LOS systems
  - **Mitigation**: Flexible validation regex, clear error messages
- **Risk**: API timeout for large loan lists
  - **Mitigation**: Implement request timeout, display timeout error, allow retry

---

<a name="user-story-2"></a>
## User Story 2: Bulk CSV Upload & Loan Parsing

### Story ID
**BRD-BULK-002**

### Story Title
Bulk CSV Upload & Loan Parsing

### User Story Statement
**As a** loan operations manager
**I want to** upload an ELD Report CSV file and automatically extract loan numbers
**So that** I can process 50-500 loans efficiently without manual data entry

---

### BRD Summary

#### Overview
This feature allows users to upload CSV files (typically ELD Reports from the Loan Origination System) and automatically parse loan numbers based on filter criteria. This is the primary method for processing large batches of loans (50-500+).

#### Business Value
- **Scalability**: Handle hundreds of loans in single operation
- **Accuracy**: Eliminates manual transcription errors
- **Integration**: Leverages existing LOS reports
- **Time Savings**: Processes 500 loans as quickly as 10 loans

#### Scope
- File upload interface (CSV only)
- Filter type selection (e.g., "ELD Report")
- Filter value input for targeted parsing
- CSV parsing and loan number extraction
- Integration with validation and summary generation workflow

---

### Detailed Requirements

#### Functional Requirements

**FR-008: Bulk Upload Tab**
- System MUST provide "Bulk Upload" tab alongside "Manual Entry" tab
- Tab switching MUST preserve bundle name and optional fields
- Tab content MUST change based on selection
- Only one tab can be active at a time

**FR-009: Filter Selection**
- System MUST provide "Select Filter" dropdown
- Default filter type: "ELD Report"
- Additional filter types: TBD based on business needs
- Filter type determines CSV column mapping logic

**FR-010: Filter Value Input**
- System MUST provide "Filter Value" text input field
- Field accepts alphanumeric input
- Field determines which rows to extract from CSV
- Example: Enter "12345" to extract loans matching criteria "12345"

**FR-011: File Upload**
- System MUST provide "Upload File" button
- System MUST accept only CSV files (.csv extension)
- System MUST reject non-CSV files with error message
- System MUST support files up to 10 MB in size
- System MUST display selected filename after upload

**FR-012: CSV Parsing**
- System MUST parse CSV file line-by-line
- System MUST identify header row
- System MUST map columns based on filter type
- For "ELD Report" filter:
  - Extract loan numbers from "Loan Number" or "Subject Property Loan #" column
  - Apply filter value to relevant column (e.g., "Trade Number" column)
  - Extract only rows matching filter criteria
- System MUST handle missing columns gracefully
- System MUST handle empty cells gracefully
- System MUST skip header row

**FR-013: Loan Number Extraction**
- After parsing, system extracts loan numbers into array
- System applies same validation as manual entry (see FR-004)
- System removes duplicates automatically
- System proceeds to summary generation (same as manual entry)

**FR-014: File Size & Error Handling**
- System MUST reject files larger than 10 MB
- System MUST display error if file cannot be parsed
- System MUST display error if no loan numbers found
- System MUST display count of extracted loan numbers: "Found 127 loan numbers"

---

#### Technical Requirements

**TR-006: File Upload Component**
- Use HTML5 file input with accept=".csv"
- Read file contents using FileReader API
- Parse CSV using library (e.g., PapaParse) or custom parser
- Store parsed data in state

**TR-007: CSV Parsing Logic**
```javascript
// Example parsing logic
const parseCSV = (fileContent, filterType, filterValue) => {
  const lines = fileContent.split('\n');
  const headers = lines[0].split(',');

  // Find relevant column indices
  const loanNumberCol = headers.findIndex(h =>
    h.includes('Loan Number') || h.includes('Subject Property Loan')
  );
  const filterCol = headers.findIndex(h =>
    h.includes('Trade Number') // Depends on filter type
  );

  // Extract loan numbers from matching rows
  const loanNumbers = [];
  for (let i = 1; i < lines.length; i++) {
    const cells = lines[i].split(',');
    if (cells[filterCol] === filterValue) {
      loanNumbers.push(cells[loanNumberCol].trim());
    }
  }

  return loanNumbers;
};
```

**TR-008: State Management**
- Store uploaded file object in state
- Store parsed loan numbers in state (same array as manual entry)
- Store parsing errors in state
- Merge with manual entry workflow after parsing

**TR-009: Error Handling**
- Catch CSV parsing errors (malformed CSV)
- Catch file read errors (corrupted file)
- Display errors inline below upload button
- Allow user to retry with different file

---

### Acceptance Criteria

#### AC-011: Bulk Upload Tab Display
- [ ] Given I am on Bundle Builder tab
- [ ] When I click "Bulk Upload" tab
- [ ] Then I see filter selection dropdown
- [ ] And I see filter value input field
- [ ] And I see "Upload File" button
- [ ] And manual entry text area is hidden

#### AC-012: Filter Selection
- [ ] Given I am on Bulk Upload tab
- [ ] When I click "Select Filter" dropdown
- [ ] Then I see filter options including "ELD Report"
- [ ] And I can select one filter type
- [ ] And selected filter appears in dropdown

#### AC-013: Filter Value Entry
- [ ] Given I have selected filter type
- [ ] When I type filter value (e.g., "12345")
- [ ] Then system accepts my input
- [ ] And field displays entered value

#### AC-014: File Upload - Success
- [ ] Given I have selected filter type and entered filter value
- [ ] When I click "Upload File" and select a valid CSV file
- [ ] Then system accepts the file
- [ ] And system displays filename: "example.csv"
- [ ] And system begins parsing

#### AC-015: File Upload - Invalid File Type
- [ ] Given I have selected filter type
- [ ] When I try to upload non-CSV file (e.g., .xlsx, .pdf)
- [ ] Then system rejects the file
- [ ] And system displays error: "Please upload a CSV file"
- [ ] And no parsing occurs

#### AC-016: File Upload - File Too Large
- [ ] Given I have selected filter type
- [ ] When I try to upload CSV file larger than 10 MB
- [ ] Then system rejects the file
- [ ] And system displays error: "File size exceeds 10 MB limit"
- [ ] And no parsing occurs

#### AC-017: CSV Parsing Success
- [ ] Given I have uploaded valid ELD Report CSV
- [ ] When system parses the file with filter value "12345"
- [ ] Then system extracts all loan numbers matching filter criteria
- [ ] And system displays: "Found 127 loan numbers"
- [ ] And system proceeds to validation (same as manual entry)

#### AC-018: CSV Parsing - No Matches
- [ ] Given I have uploaded valid CSV
- [ ] When system parses with filter value that matches no rows
- [ ] Then system displays error: "No loan numbers found matching filter criteria"
- [ ] And no API call is made
- [ ] And fields remain unlocked

#### AC-019: CSV Parsing - Malformed CSV
- [ ] Given I have uploaded malformed CSV (inconsistent columns)
- [ ] When system attempts to parse
- [ ] Then system displays error: "Unable to parse CSV file. Please check file format."
- [ ] And no API call is made
- [ ] And user can retry with different file

#### AC-020: Integration with Manual Entry Workflow
- [ ] Given CSV parsing succeeds and extracts 50 loan numbers
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then system validates loan numbers (same validation as manual entry)
- [ ] And system locks fields
- [ ] And system calls API
- [ ] And system displays summary grid

---

### Dependencies
- User Story 1: Manual Loan Entry (shares validation and generation workflow)
- User Story 3: Loan Summary Grid Display (displays results)
- User Story 6: Error Handling & Validation (handles parsing errors)

---

### Risks & Mitigations
- **Risk**: Different LOS systems generate different CSV formats
  - **Mitigation**: Flexible column mapping, support multiple CSV formats
- **Risk**: Users upload incorrect file type
  - **Mitigation**: File type validation, clear error messages
- **Risk**: CSV contains thousands of loans (performance issue)
  - **Mitigation**: Limit to 500 loans, suggest batching for larger sets

---

<a name="user-story-3"></a>
## User Story 3: Loan Summary Grid Display & Filtering

### Story ID
**BRD-BULK-003**

### Story Title
Loan Summary Grid Display & Filtering

### User Story Statement
**As a** loan processor
**I want to** view document status for all loans in a filterable grid
**So that** I can quickly identify which loans are ready to bundle and which need attention

---

### BRD Summary

#### Overview
After loan numbers are submitted to the API, the system displays a comprehensive grid showing document status for each loan. The grid includes filtering capabilities to view all loans, loans with missing documents, or complete loans. This is the primary interface for reviewing loan readiness.

#### Business Value
- **Visibility**: Clear view of document status across all loans
- **Decision Support**: Quickly identify bottlenecks and priorities
- **Efficiency**: Filter to focus on specific loan categories
- **Actionability**: Enable targeted action on problem loans

#### Scope
- Grid display with multiple columns (loan number, missing docs, found docs, status)
- Filter tabs (All, Missing, Complete)
- Summary statistics header
- Row selection via checkboxes
- Status color coding
- Clickable loan numbers for drill-down (future enhancement)

---

### Detailed Requirements

#### Functional Requirements

**FR-015: Summary Grid Display**
- System MUST display grid after API response is received
- Grid MUST show one row per loan
- Grid MUST include columns:
  1. **Checkbox**: For selecting individual loans
  2. **Loan Number**: Identifier for the loan
  3. **# Missing**: Count of missing documents (red text if > 0)
  4. **# Found**: Count of found documents (gray text)
  5. **Status**: Bundle status badge (color-coded)
- Grid MUST support vertical scrolling for large loan lists
- Grid MUST display "No loans found" if result set is empty

**FR-016: Summary Header**
- System MUST display summary statistics above grid:
  - Total Loans: Count of all loans (gray badge)
  - Missing Docs: Count of loans with missing documents (red badge)
  - Complete: Count of loans with zero missing docs (green badge)
- Header MUST update dynamically when data changes

**FR-017: Filter Tabs**
- System MUST provide three filter tabs:
  1. **All (X)**: Shows all loans
  2. **Missing (X)**: Shows only loans with missing documents > 0
  3. **Complete (X)**: Shows only loans with missing documents = 0
- Active tab MUST be highlighted (teal background)
- Inactive tabs MUST be gray
- Tab counts MUST update dynamically
- Clicking tab MUST filter grid immediately (no API call)

**FR-018: Status Badges**
- System MUST display status badges with color coding:
  - **Not Bundled**: Gray badge - Loan has missing documents
  - **Ready to Bundle**: Green badge - Loan has zero missing documents
  - **In Progress**: Yellow badge - Bundle build in progress for this loan
  - **Completed**: Green badge with checkmark - Bundle successfully built
  - **Bundle Failed**: Red badge with X - Bundle build failed

**FR-019: Row Selection**
- System MUST provide checkbox for each loan
- System MUST provide "Select All" checkbox in header
- "Select All" MUST select all visible loans in current filter
- Loans with status "In Progress" MUST have disabled checkboxes (cannot be selected)
- Selected count MUST display in header: "X loans selected"
- Selection state MUST persist when switching between filter tabs

**FR-020: Loan Number Clickability (Future)**
- Loan numbers SHOULD be clickable links (blue, underlined)
- Clicking loan number SHOULD open loan detail view (future enhancement)
- For current version, loan numbers are plain text

---

#### Technical Requirements

**TR-010: Grid Rendering**
- Use table or grid component (HTML table or React grid library)
- Implement client-side filtering (no API call on tab change)
- Store full loan array in state, filter for display
- Implement virtual scrolling for 100+ loans (performance optimization)

**TR-011: Filter Logic**
```javascript
const getFilteredLoans = () => {
  if (activeTab === 'all') return loans;
  if (activeTab === 'missing') return loans.filter(l => l.missingDocs > 0);
  if (activeTab === 'complete') return loans.filter(l => l.missingDocs === 0);
};
```

**TR-012: Status Determination**
- Status determined by backend API based on:
  - Document count (missing vs. found)
  - Bundle build state (not started, in progress, completed, failed)
- Frontend displays status badge based on status value from API

**TR-013: Selection Management**
- Store selected loan IDs in state array
- Update array when checkbox toggled
- "Select All" sets all visible loan IDs in array
- Deselect all clears array

**TR-014: Responsive Design**
- Grid MUST be horizontally scrollable on small screens
- Grid MUST display all columns without truncation
- Summary header MUST stack vertically on mobile

---

### Acceptance Criteria

#### AC-021: Grid Display After API Success
- [ ] Given API returns data for 27 loans
- [ ] When data is received
- [ ] Then system displays grid with 27 rows
- [ ] And each row shows loan number, missing count, found count, and status
- [ ] And all columns are visible and aligned

#### AC-022: Summary Header Display
- [ ] Given grid displays 27 loans (15 with missing docs, 12 complete)
- [ ] When I view the summary header
- [ ] Then I see "Total Loans: 27" in gray badge
- [ ] And I see "Missing Docs: 15" in red badge
- [ ] And I see "Complete: 12" in green badge

#### AC-023: Filter Tab - All
- [ ] Given I am on "All" tab (default)
- [ ] When I view the grid
- [ ] Then I see all 27 loans
- [ ] And tab shows "All (27)"
- [ ] And tab is highlighted in teal

#### AC-024: Filter Tab - Missing
- [ ] Given I am on "All" tab
- [ ] When I click "Missing" tab
- [ ] Then I see only 15 loans with missing documents > 0
- [ ] And tab shows "Missing (15)"
- [ ] And tab is highlighted in teal
- [ ] And "All" tab is no longer highlighted

#### AC-025: Filter Tab - Complete
- [ ] Given I am on "All" tab
- [ ] When I click "Complete" tab
- [ ] Then I see only 12 loans with missing documents = 0
- [ ] And tab shows "Complete (12)"
- [ ] And tab is highlighted in teal

#### AC-026: Status Badge Colors
- [ ] Given grid displays loans with various statuses
- [ ] When I view status column
- [ ] Then "Not Bundled" appears as gray badge
- [ ] And "Ready to Bundle" appears as green badge
- [ ] And "In Progress" appears as yellow badge
- [ ] And "Completed" appears as green badge with checkmark icon
- [ ] And "Bundle Failed" appears as red badge with X icon

#### AC-027: Missing Docs Red Highlighting
- [ ] Given loan has 3 missing documents
- [ ] When I view the "# Missing" column
- [ ] Then count "3" appears in red text
- [ ] And it stands out visually from found docs count

#### AC-028: Select Individual Loan
- [ ] Given I am viewing grid
- [ ] When I click checkbox next to loan "USB12345"
- [ ] Then checkbox becomes checked
- [ ] And selected count updates: "1 loan selected"

#### AC-029: Select All Loans
- [ ] Given I am viewing grid with 27 loans (none selected)
- [ ] When I click "Select All" checkbox in header
- [ ] Then all 27 checkboxes become checked
- [ ] And selected count updates: "27 loans selected"

#### AC-030: Cannot Select In Progress Loans
- [ ] Given loan "USB12349" has status "In Progress"
- [ ] When I view the grid
- [ ] Then checkbox for "USB12349" is disabled (grayed out)
- [ ] And I cannot check it
- [ ] And clicking "Select All" does not select it

#### AC-031: Selection Persists Across Tabs
- [ ] Given I am on "All" tab and have selected 5 loans
- [ ] When I switch to "Missing" tab
- [ ] Then selected loans that are in "Missing" view remain selected
- [ ] When I switch back to "All" tab
- [ ] Then all 5 originally selected loans are still selected

#### AC-032: Empty Grid Message
- [ ] Given I am on "Complete" tab and there are 0 complete loans
- [ ] When I view the grid
- [ ] Then I see message: "No loans found"
- [ ] And grid displays empty state (no rows)

---

### Dependencies
- User Story 1: Manual Loan Entry (provides data to display)
- User Story 2: Bulk CSV Upload (provides data to display)
- User Story 4: Bundle Build Execution (uses selection data)
- User Story 10: EPS/BytePro API Integration (provides loan status data)

---

### Risks & Mitigations
- **Risk**: Grid performance issues with 500+ loans
  - **Mitigation**: Implement virtual scrolling, pagination if needed
- **Risk**: Confusing status labels
  - **Mitigation**: User testing, clear documentation, tooltips
- **Risk**: Selection state lost when data refreshes
  - **Mitigation**: Preserve selection by loan ID, not index

---

<a name="user-story-4"></a>
## User Story 4: Bundle Build Execution

### Story ID
**BRD-BULK-004**

### Story Title
Bundle Build Execution

### User Story Statement
**As a** loan processor
**I want to** initiate bundle build for selected loans or all ready loans
**So that** I can generate investor-ready PDF bundles according to specified stacking order

---

### BRD Summary

#### Overview
This feature allows users to execute bundle builds in two modes: building all loans with "Ready to Bundle" status, or building only selected loans. The system calls the BytePro API via EPS to merge documents according to the bundle's stacking order, generating a final PDF output.

#### Business Value
- **Automation**: Automated PDF generation eliminates manual document compilation
- **Accuracy**: Ensures correct document order per investor requirements
- **Flexibility**: Build all ready loans or selective subset
- **Efficiency**: Process multiple loans simultaneously

#### Scope
- Two build modes: "Build All Ready" and "Build Selected"
- Build confirmation modal
- Progress tracking during build
- API integration with BytePro build endpoint
- Status updates as loans complete

---

### Detailed Requirements

#### Functional Requirements

**FR-021: Build Mode Selection**
- System MUST provide two build buttons:
  1. **"Build Bundle (Ready)"**: Builds all loans with status "Ready to Bundle"
  2. **"Build Bundle (Selected)"**: Builds only checked loans
- Buttons MUST be displayed in action button row
- Buttons MUST be disabled during API loading or if no valid loans available

**FR-022: Build Validation - Ready Mode**
- When "Build Bundle (Ready)" clicked:
  - System counts loans with status "Ready to Bundle"
  - If count = 0: Display error "No loans ready to bundle"
  - If count > 0: Proceed to confirmation modal

**FR-023: Build Validation - Selected Mode**
- When "Build Bundle (Selected)" clicked:
  - System counts selected loans (checked checkboxes)
  - System excludes loans with status "In Progress"
  - If count = 0: Display error "No loans selected"
  - If all selected loans are "In Progress": Display error "Cannot build loans already in progress"
  - If count > 0: Proceed to confirmation modal

**FR-024: Build Confirmation Modal**
- System MUST display confirmation modal before build starts
- Modal MUST display:
  - Title: "Confirm Bundle Build"
  - Message: "You are about to build X loans"
  - Warning (if applicable): "⚠️ Y loans have missing documents and may fail"
  - Buttons: "Yes, Build Bundle" (green) | "Cancel" (gray)
- Clicking "Yes, Build Bundle" initiates build process
- Clicking "Cancel" closes modal and returns to grid
- Clicking outside modal closes modal (no build)

**FR-025: Build Process Initiation**
- System calls EPS build endpoint with loan number array
- System displays build progress modal:
  - Title: "Building Bundle..."
  - Progress bar (animated or percentage-based)
  - Status text: "Processing 0 of X loans"
  - Modal cannot be dismissed during build (no close button)

**FR-026: Progress Updates**
- System receives progress updates from API (WebSocket or polling)
- Progress modal updates:
  - Progress bar: Percentage completion
  - Status text: "Processing 12 of 27 loans (44%)"
- Individual loan statuses in grid update in real-time:
  - "Ready to Bundle" → "In Progress" (when build starts)
  - "In Progress" → "Completed" (when build succeeds)
  - "In Progress" → "Bundle Failed" (when build fails)

**FR-027: Build Completion**
- When build finishes, close progress modal
- Display build results modal (see User Story 8)
- Update all loan statuses in grid
- Enable action buttons for next operation

---

#### Technical Requirements

**TR-015: Build Mode Logic**
```javascript
const handleBuildReady = () => {
  const readyLoans = loans.filter(l => l.bundleStatus === 'Ready to Bundle');
  if (readyLoans.length === 0) {
    showError('No loans ready to bundle');
    return;
  }
  showConfirmModal(readyLoans);
};

const handleBuildSelected = () => {
  const selectedLoans = loans.filter(l =>
    l.selected && l.bundleStatus !== 'In Progress'
  );
  if (selectedLoans.length === 0) {
    showError('No loans selected');
    return;
  }
  showConfirmModal(selectedLoans);
};
```

**TR-016: API Request Format**
```json
{
  "bundleName": "U.S. Bank",
  "loanNumbers": ["USB12345", "USB12346", ...],
  "buildMode": "ready", // or "selected"
  "authToken": "Bearer <token>",
  "requestTimestamp": "2025-01-19T11:00:00Z"
}
```

**TR-017: API Response Format**
```json
{
  "success": true,
  "jobId": "JOB-12345",
  "totalLoans": 27,
  "message": "Bundle build initiated",
  "estimatedCompletionTime": "2025-01-19T11:05:00Z"
}
```

**TR-018: Progress Tracking**
- Poll build status API every 2 seconds: `GET /api/v1/bundle-status/{jobId}`
- Update progress bar based on completed loan count
- Update individual loan statuses in grid
- Stop polling when all loans complete or fail

**TR-019: State Management**
- Store build job ID in state
- Store progress percentage in state
- Update loan statuses in state as they complete
- Set `isBuilding` flag to true during build (disables buttons)

---

### Acceptance Criteria

#### AC-033: Build Ready Button Display
- [ ] Given I have generated summary with 12 "Ready to Bundle" loans
- [ ] When I view action buttons
- [ ] Then I see "Build Bundle (Ready)" button
- [ ] And button is enabled (not grayed out)

#### AC-034: Build Selected Button Display
- [ ] Given I have selected 5 loans via checkboxes
- [ ] When I view action buttons
- [ ] Then I see "Build Bundle (Selected)" button
- [ ] And button is enabled

#### AC-035: Build Ready - Success Flow
- [ ] Given I have 12 loans with status "Ready to Bundle"
- [ ] When I click "Build Bundle (Ready)"
- [ ] Then confirmation modal appears
- [ ] And modal shows "You are about to build 12 loans"
- [ ] When I click "Yes, Build Bundle"
- [ ] Then progress modal appears with "Building Bundle..."
- [ ] And API call is made with 12 loan numbers

#### AC-036: Build Ready - No Ready Loans
- [ ] Given I have 0 loans with status "Ready to Bundle"
- [ ] When I click "Build Bundle (Ready)"
- [ ] Then error message displays: "No loans ready to bundle"
- [ ] And no modal appears
- [ ] And no API call is made

#### AC-037: Build Selected - Success Flow
- [ ] Given I have selected 5 loans (none are "In Progress")
- [ ] When I click "Build Bundle (Selected)"
- [ ] Then confirmation modal appears
- [ ] And modal shows "You are about to build 5 loans"
- [ ] When I click "Yes, Build Bundle"
- [ ] Then progress modal appears
- [ ] And API call is made with 5 selected loan numbers

#### AC-038: Build Selected - No Selection
- [ ] Given I have NOT selected any loans (no checkboxes checked)
- [ ] When I click "Build Bundle (Selected)"
- [ ] Then error message displays: "No loans selected"
- [ ] And no modal appears
- [ ] And no API call is made

#### AC-039: Build Selected - All In Progress
- [ ] Given I have selected 3 loans, all with status "In Progress"
- [ ] When I click "Build Bundle (Selected)"
- [ ] Then error message displays: "Cannot build loans already in progress"
- [ ] And no modal appears

#### AC-040: Build Selected - Mixed Selection with Missing Docs
- [ ] Given I have selected 8 loans, 3 of which have missing documents
- [ ] When I click "Build Bundle (Selected)"
- [ ] Then confirmation modal appears
- [ ] And modal shows warning: "⚠️ 3 loans have missing documents and may fail"
- [ ] When I click "Yes, Build Bundle"
- [ ] Then build proceeds anyway (user choice)

#### AC-041: Confirmation Modal - Cancel
- [ ] Given confirmation modal is displayed
- [ ] When I click "Cancel" button
- [ ] Then modal closes
- [ ] And I return to grid view
- [ ] And no API call is made

#### AC-042: Progress Modal Display
- [ ] Given build has been initiated
- [ ] When build is in progress
- [ ] Then progress modal displays
- [ ] And modal shows "Building Bundle..."
- [ ] And modal shows progress: "Processing 0 of 27 loans"
- [ ] And modal shows animated progress bar
- [ ] And modal cannot be closed (no close button)

#### AC-043: Progress Updates During Build
- [ ] Given build is processing 27 loans
- [ ] When API returns progress: 12 loans completed
- [ ] Then progress modal updates: "Processing 12 of 27 loans (44%)"
- [ ] And progress bar shows 44% completion
- [ ] And grid updates: 12 loans show "Completed" status
- [ ] And 15 loans show "In Progress" status

#### AC-044: Build Completion
- [ ] Given build has completed for all 27 loans
- [ ] When API returns final status
- [ ] Then progress modal closes
- [ ] And build results modal appears (see User Story 8)
- [ ] And all loan statuses in grid are updated
- [ ] And action buttons become enabled again

---

### Dependencies
- User Story 3: Loan Summary Grid Display (provides loan data and selection)
- User Story 8: Build Status Tracking (displays results)
- User Story 10: EPS/BytePro API Integration (executes build)

---

### Risks & Mitigations
- **Risk**: Build takes very long time (10+ minutes for 100 loans)
  - **Mitigation**: Display estimated time, allow background processing
- **Risk**: Network disconnection during build
  - **Mitigation**: Implement job recovery, allow status check by job ID
- **Risk**: User accidentally clicks "Build" without reviewing loans
  - **Mitigation**: Confirmation modal with clear summary

---

<a name="user-story-5"></a>
## User Story 5: Dashboard History & Bundle Review

### Story ID
**BRD-BULK-005**

### Story Title
Dashboard History & Bundle Review

### User Story Statement
**As a** loan operations manager
**I want to** view history of all completed bundle builds in a dashboard
**So that** I can track bundle activity, review past builds, and audit bundle status

---

### BRD Summary

#### Overview
The Dashboard provides a historical view of all bundle builds completed in the system. Users can view bundle name, build date/time, loan count, and status (Complete, Incomplete, In Progress). Clicking a bundle name loads the full bundle details for review.

#### Business Value
- **Auditability**: Track all bundle builds for compliance
- **Visibility**: See bundle activity at a glance
- **Efficiency**: Quickly access recent bundles for review
- **Training**: Sample bundles for training new users

#### Scope
- Dashboard tab with table of bundle history
- Display recent builds (last 24 hours for demo, 10 business days for production)
- Display 4 sample bundles for training/demo
- Clickable bundle names to reload bundle details
- Auto-cleanup of old entries (24 hours for demo)

---

### Detailed Requirements

#### Functional Requirements

**FR-028: Dashboard Tab**
- System MUST provide "Dashboard" tab in main navigation
- Tab displays table of bundle history
- Table has columns:
  1. **Bundle Name**: Clickable link
  2. **Latest Build Date/Time**: Formatted timestamp
  3. **Number of Loans**: Count of loans in bundle
  4. **Bulk Bundle Status**: Status with color coding

**FR-029: Recent History Display**
- System MUST display bundles created in current session
- Recent entries appear at top of table
- Recent entries highlighted with light blue background (bg-blue-50)
- Most recent entry appears first (reverse chronological order)

**FR-030: Sample Bundles Display**
- System MUST display 4 hardcoded sample bundles below recent history
- Sample bundles always present (not subject to cleanup)
- Sample bundles:
  1. "US Bank" - 27 loans - Incomplete
  2. "American Bank" - 10 loans - In progress
  3. "US Bank" - 11 loans - In progress
  4. "American Bank" - 13 loans - Complete
- Sample bundles have white background (normal rows)

**FR-031: Auto-Cleanup (Demo Behavior)**
- System MUST automatically remove dashboard entries older than 24 hours
- Cleanup runs on component mount (page load)
- Sample bundles NOT subject to cleanup
- Cleanup runs silently (no user notification unless entries removed)

**FR-032: Bundle Name Click Action**
- Clicking bundle name link loads bundle details
- System searches dashboardHistory array for matching entry
- If not found in history, searches sampleBundleData object
- If found: Load bundle data into Bundle Builder
- If not found: Display error "Bundle data not found"

**FR-033: Load Bundle Details**
- When bundle found:
  1. Switch to "Bundle Builder" tab
  2. Populate bundle name dropdown
  3. Load loan numbers into state
  4. Lock all input fields (read-only mode)
  5. Display loan summary grid with all loan statuses
  6. User can review, refresh, or build additional loans

**FR-034: Retention Notice**
- System MUST display notice below table:
  - "Note: Dashboard history entries will be retained for 10 business days and automatically removed thereafter."
  - Text in light gray, small font (text-xs text-gray-400)
  - Note describes production behavior (actual demo is 24 hours)

---

#### Technical Requirements

**TR-020: Dashboard Data Structure**
```javascript
const dashboardHistory = [
  {
    bundleName: "U.S. Bank",
    loanCount: 27,
    timestamp: "2025-01-19T11:15:30Z",
    status: "Complete", // or "Incomplete" or "In progress"
    loans: [
      { id: "USB12345", bundleStatus: "Completed", missingDocs: 0, foundDocs: 58 },
      // ... all loans
    ]
  },
  // ... more entries
];
```

**TR-021: Auto-Cleanup Logic**
```javascript
useEffect(() => {
  const cleanupOldEntries = () => {
    const now = new Date();
    const twentyFourHoursAgo = new Date(now.getTime() - (24 * 60 * 60 * 1000));

    setDashboardHistory(prevHistory => {
      return prevHistory.filter(entry => {
        const entryDate = new Date(entry.timestamp);
        return entryDate > twentyFourHoursAgo;
      });
    });
  };

  cleanupOldEntries();
}, []); // Run on mount
```

**TR-022: Bundle Click Handler**
```javascript
const handleBundleClick = (bundleKey) => {
  // Search dashboard history
  let bundleData = dashboardHistory.find(entry =>
    `${entry.bundleName}-${entry.loanCount}-${entry.status}` === bundleKey
  );

  // If not in history, check sample data
  if (!bundleData) {
    bundleData = sampleBundleData[bundleKey];
  }

  if (bundleData) {
    // Load bundle into builder
    setMainTab('bundleBuilder');
    setBundleName(bundleData.bundleName);
    setLoans(bundleData.loans);
    setFieldsLocked(true);
    setShowSummary(true);
  } else {
    showError('Bundle data not found');
  }
};
```

**TR-023: Date Formatting**
- Format timestamp as: "MM/DD/YY H:MM am/pm EST"
- Example: "01/19/25 11:15am EST"
- Use JavaScript toLocaleString with options:
  ```javascript
  new Date(timestamp).toLocaleString('en-US', {
    month: '2-digit',
    day: '2-digit',
    year: '2-digit',
    hour: 'numeric',
    minute: '2-digit',
    hour12: true
  }) + ' EST';
  ```

---

### Acceptance Criteria

#### AC-045: Dashboard Tab Display
- [ ] Given I am on any tab
- [ ] When I click "Dashboard" tab
- [ ] Then I see dashboard table
- [ ] And table has columns: Bundle Name, Latest Build Date/Time, Number of Loans, Bulk Bundle Status

#### AC-046: Sample Bundles Always Display
- [ ] Given I open the application for first time
- [ ] When I view Dashboard
- [ ] Then I see 4 sample bundles:
  - US Bank (27 loans, Incomplete)
  - American Bank (10 loans, In progress)
  - US Bank (11 loans, In progress)
  - American Bank (13 loans, Complete)
- [ ] And sample bundles have white background

#### AC-047: Recent Bundle Appears in Dashboard
- [ ] Given I have just completed a bundle build for "U.S. Bank" with 27 loans
- [ ] When build completes and I navigate to Dashboard
- [ ] Then I see new entry at top of table
- [ ] And entry shows "U.S. Bank" bundle name
- [ ] And entry shows today's date/time
- [ ] And entry shows 27 loans
- [ ] And entry shows status (Complete, Incomplete, or In progress)
- [ ] And entry has light blue background

#### AC-048: Recent Entry Appears First
- [ ] Given I have completed 3 bundles (Build 1, Build 2, Build 3 in that order)
- [ ] When I view Dashboard
- [ ] Then Build 3 appears at top
- [ ] And Build 2 appears second
- [ ] And Build 1 appears third
- [ ] And sample bundles appear after all recent builds

#### AC-049: Auto-Cleanup After 24 Hours
- [ ] Given I completed a bundle build 25 hours ago
- [ ] When I refresh the application (component remounts)
- [ ] Then old entry is removed from dashboard
- [ ] And only bundles from last 24 hours remain
- [ ] And sample bundles remain (not removed)

#### AC-050: Bundle Name Click - Load Recent Build
- [ ] Given I see recent bundle "U.S. Bank" in dashboard
- [ ] When I click "U.S. Bank" link
- [ ] Then system switches to Bundle Builder tab
- [ ] And bundle name "U.S. Bank" is populated
- [ ] And all 27 loans are loaded in summary grid
- [ ] And all fields are locked (read-only mode)
- [ ] And loan statuses reflect saved state

#### AC-051: Bundle Name Click - Load Sample Build
- [ ] Given I see sample bundle "American Bank" in dashboard
- [ ] When I click "American Bank" link
- [ ] Then system switches to Bundle Builder tab
- [ ] And bundle name "American Bank" is populated
- [ ] And sample loan data is loaded in grid
- [ ] And fields are locked

#### AC-052: Bundle Name Click - Not Found
- [ ] Given a bundle was removed from history (edge case)
- [ ] When I click bundle name link
- [ ] Then system displays error toast: "Bundle data not found"
- [ ] And system remains on Dashboard tab

#### AC-053: Status Color Coding in Dashboard
- [ ] Given dashboard displays bundles with various statuses
- [ ] When I view status column
- [ ] Then "Complete" appears in gray text
- [ ] And "Incomplete" appears in red text
- [ ] And "In progress" appears in gray text

#### AC-054: Retention Notice Display
- [ ] Given I am viewing Dashboard
- [ ] When I scroll to bottom of table
- [ ] Then I see notice: "Note: Dashboard history entries will be retained for 10 business days and automatically removed thereafter."
- [ ] And text is light gray and small font

#### AC-055: No Recent Builds Scenario
- [ ] Given I have NOT completed any bundles in this session
- [ ] When I view Dashboard
- [ ] Then I see only 4 sample bundles
- [ ] And no recent entries appear
- [ ] And dashboard is not empty

---

### Dependencies
- User Story 4: Bundle Build Execution (creates dashboard entries)
- User Story 3: Loan Summary Grid Display (displays loaded bundle data)

---

### Risks & Mitigations
- **Risk**: Dashboard becomes cluttered with many entries
  - **Mitigation**: Auto-cleanup after 24 hours (demo) or 10 days (prod)
- **Risk**: Users confused about demo vs. production retention
  - **Mitigation**: Clear documentation, notice text explains production behavior
- **Risk**: Clicking bundle link loads stale data
  - **Mitigation**: Allow refresh option after loading bundle

---

<a name="user-story-6"></a>
## User Story 6: Error Handling & Validation

### Story ID
**BRD-BULK-006**

### Story Title
Comprehensive Error Handling & Validation

### User Story Statement
**As a** loan processor
**I want to** receive clear error messages and validation feedback
**So that** I can quickly correct issues and successfully complete bundle operations

---

### BRD Summary

#### Overview
Comprehensive error handling and validation ensures users receive immediate, actionable feedback when issues occur. This includes input validation, API error handling, network error recovery, and user-friendly error messages across all system operations.

#### Business Value
- **User Experience**: Clear errors reduce frustration and support tickets
- **Efficiency**: Quick error resolution keeps workflow moving
- **Data Quality**: Validation prevents bad data from entering system
- **Reliability**: Graceful error handling prevents system crashes

#### Scope
- Client-side input validation
- API error handling and display
- Network error recovery with auto-retry
- Validation error messages
- Timeout handling
- Session expiration handling

---

### Detailed Requirements

#### Functional Requirements

**FR-035: Input Validation Errors**
- System MUST validate all inputs before API calls
- Validation errors MUST display inline (near input field)
- Error text MUST be red color
- Error text MUST be specific and actionable
- Examples:
  - "Please select a bundle name first"
  - "Please enter at least one loan number"
  - "Invalid loan number format: [specific issues]"
  - "Invalid loan number - only alphanumeric characters allowed"

**FR-036: API Error Handling**
- System MUST catch all API errors
- System MUST display error modal for critical errors
- Error modal MUST show:
  - Error icon (red X)
  - Error title
  - Error message (user-friendly)
  - Technical details (if available, collapsible)
  - "Close" or "Retry" button
- Examples:
  - "Unable to connect to document service. Please try again."
  - "Bundle build failed: [reason from BytePro]"
  - "Loan USB12345 not found in system"

**FR-037: Network Error Recovery**
- System MUST detect network disconnection
- System MUST auto-retry failed requests 3 times
- System MUST display toast notification during retry:
  - "Connection lost. Retrying... (Attempt 1 of 3)"
- After 3 failed retries:
  - Display error modal with "Retry" button for manual retry
  - "Network error. Please check your connection and try again."

**FR-038: Timeout Handling**
- System MUST implement timeout for API calls (2 minutes default)
- System MUST display timeout modal if timeout occurs:
  - "Request is taking longer than expected."
  - Options: "Continue Waiting" | "Cancel"
  - If Continue: Extend timeout by 2 minutes
  - If Cancel: Cancel request, return to grid

**FR-039: Session Expiration**
- System MUST detect 401 Unauthorized responses (expired token)
- System MUST display session expiration modal:
  - "Your session has expired. Please log in again."
  - Button: "Log In"
- Clicking "Log In" redirects to SSO login
- After re-authentication, return user to previous page (preserve state if possible)

**FR-040: Validation Warning Messages**
- System MAY display warning messages for non-blocking issues
- Warnings displayed as yellow toast notifications
- Examples:
  - "3 duplicate loan numbers removed"
  - "Warning: 5 loans have missing documents and may fail to bundle"
- Warnings do not prevent operation from continuing

**FR-041: Specific Error Scenarios**
- **No Bundle Name**: "Please select a bundle name first"
- **No Loan Numbers**: "Please enter at least one loan number"
- **Invalid Loan Format**: "Invalid loan number format: [list specific issues]"
- **CSV Parse Error**: "Unable to parse CSV file. Please check file format."
- **File Too Large**: "File size exceeds 10 MB limit"
- **No Matches in CSV**: "No loan numbers found matching filter criteria"
- **No Ready Loans**: "No loans ready to bundle"
- **No Selection**: "No loans selected"
- **All In Progress**: "Cannot build loans already in progress"
- **EPS Unavailable**: "Unable to connect to document service. Please try again."
- **BytePro Error**: "Bundle build failed: [specific error]"
- **Storage Full**: "Bundle built but could not be saved. Contact administrator."

---

#### Technical Requirements

**TR-024: Validation Function**
```javascript
const validateInputs = () => {
  const errors = [];

  if (!bundleName) {
    errors.push('Please select a bundle name first');
  }

  if (loanNumbers.length === 0) {
    errors.push('Please enter at least one loan number');
  }

  const invalidLoans = loanNumbers.filter(ln =>
    !/ ^[A-Z0-9-]{5,20}$/i.test(ln)
  );
  if (invalidLoans.length > 0) {
    errors.push(`Invalid loan number format: ${invalidLoans.join(', ')}`);
  }

  return errors;
};
```

**TR-025: API Error Handler**
```javascript
const handleAPIError = (error) => {
  if (error.response) {
    // API returned error response
    const status = error.response.status;
    const message = error.response.data.message || 'An error occurred';

    if (status === 401) {
      // Session expired
      showSessionExpiredModal();
    } else if (status === 404) {
      showError('Resource not found');
    } else if (status >= 500) {
      showError('Server error. Please try again later.');
    } else {
      showError(message);
    }
  } else if (error.request) {
    // Network error (no response received)
    handleNetworkError();
  } else {
    // Other error
    showError('An unexpected error occurred');
  }
};
```

**TR-026: Auto-Retry Logic**
```javascript
const fetchWithRetry = async (url, options, maxRetries = 3) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fetch(url, options);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      showToast(`Connection lost. Retrying... (Attempt ${i + 1} of ${maxRetries})`);
      await delay(2000); // Wait 2 seconds before retry
    }
  }
};
```

**TR-027: Error Display Components**
- **Toast Notification**: Small popup in top-right, auto-dismiss after 5 seconds
- **Inline Error**: Red text below input field, persists until corrected
- **Error Modal**: Center screen overlay, requires user action to dismiss
- **Error Banner**: Full-width banner at top of page (for critical errors)

---

### Acceptance Criteria

#### AC-056: Validation Error - No Bundle Name
- [ ] Given I have NOT selected bundle name
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then inline error displays below bundle name field
- [ ] And error text reads "Please select a bundle name first"
- [ ] And error text is red
- [ ] And no API call is made

#### AC-057: Validation Error - No Loan Numbers
- [ ] Given I have selected bundle name but entered NO loan numbers
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then inline error displays below text area
- [ ] And error text reads "Please enter at least one loan number"
- [ ] And no API call is made

#### AC-058: Validation Error - Invalid Loan Format
- [ ] Given I have entered loan numbers with special characters (!@#$%)
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then inline error displays
- [ ] And error lists specific invalid loan numbers
- [ ] And error reads "Invalid loan number format: [list]"
- [ ] And no API call is made

#### AC-059: Warning - Duplicates Removed
- [ ] Given I have entered 3 duplicate loan numbers
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then yellow toast notification appears
- [ ] And toast reads "3 duplicate loan numbers removed"
- [ ] And toast auto-dismisses after 5 seconds
- [ ] And operation continues with unique loans only

#### AC-060: API Error - Service Unavailable
- [ ] Given EPS API is down or unreachable
- [ ] When I click "Generate Bulk Bundle Summary"
- [ ] Then loading indicator appears briefly
- [ ] And error modal displays with red X icon
- [ ] And modal title reads "Connection Error"
- [ ] And modal message reads "Unable to connect to document service. Please try again."
- [ ] And modal has "Retry" and "Close" buttons

#### AC-061: Network Error - Auto Retry
- [ ] Given network connection is lost during API call
- [ ] When request fails
- [ ] Then toast notification appears: "Connection lost. Retrying... (Attempt 1 of 3)"
- [ ] And system automatically retries request
- [ ] When retry 1 fails
- [ ] Then toast updates: "Connection lost. Retrying... (Attempt 2 of 3)"
- [ ] When retry 2 fails
- [ ] Then toast updates: "Connection lost. Retrying... (Attempt 3 of 3)"
- [ ] When all 3 retries fail
- [ ] Then error modal displays with manual "Retry" button

#### AC-062: Session Expiration
- [ ] Given my auth token has expired
- [ ] When I make any API call
- [ ] Then system receives 401 Unauthorized response
- [ ] And session expiration modal displays
- [ ] And modal reads "Your session has expired. Please log in again."
- [ ] When I click "Log In"
- [ ] Then system redirects to SSO login page

#### AC-063: Timeout - Build Takes Too Long
- [ ] Given build has been running for over 2 minutes
- [ ] When timeout threshold is reached
- [ ] Then timeout modal displays
- [ ] And modal reads "Request is taking longer than expected."
- [ ] And modal shows options: "Continue Waiting" | "Cancel"
- [ ] When I click "Continue Waiting"
- [ ] Then timeout extends by 2 more minutes
- [ ] When I click "Cancel"
- [ ] Then request is cancelled and I return to grid

#### AC-064: CSV Parse Error
- [ ] Given I upload malformed CSV file (inconsistent columns)
- [ ] When system attempts to parse
- [ ] Then error displays below upload button
- [ ] And error reads "Unable to parse CSV file. Please check file format."
- [ ] And I can retry with different file

#### AC-065: Build Error - No Ready Loans
- [ ] Given I have 0 loans with status "Ready to Bundle"
- [ ] When I click "Build Bundle (Ready)"
- [ ] Then toast error displays: "No loans ready to bundle"
- [ ] And no modal appears
- [ ] And no API call is made

#### AC-066: Build Error - Loan Not Found
- [ ] Given loan "USB99999" does not exist in LOS
- [ ] When API returns status for all loans
- [ ] Then loan "USB99999" shows status "Loan not found" in grid
- [ ] And row is highlighted in red
- [ ] And user can remove loan and continue

#### AC-067: Error Modal - Retry Success
- [ ] Given error modal displays with "Retry" button
- [ ] When I click "Retry"
- [ ] Then modal closes
- [ ] And system retries the failed operation
- [ ] When retry succeeds
- [ ] Then operation continues normally

#### AC-068: Error Clears When Input Corrected
- [ ] Given inline error displays: "Please select a bundle name first"
- [ ] When I select a bundle name
- [ ] Then error message disappears immediately
- [ ] And field no longer shows error state

---

### Dependencies
- All other user stories (error handling applies to all features)

---

### Risks & Mitigations
- **Risk**: Too many error messages overwhelm users
  - **Mitigation**: Prioritize errors, show most critical first
- **Risk**: Generic error messages don't help users
  - **Mitigation**: Specific, actionable error text with examples
- **Risk**: Auto-retry loops forever
  - **Mitigation**: Limit to 3 retries, then require manual retry

---

<a name="user-story-7"></a>
## User Story 7: Refresh Summary & Add Individual Loans

### Story ID
**BRD-BULK-007**

### Story Title
Refresh Summary & Add Individual Loans

### User Story Statement
**As a** loan processor
**I want to** refresh document status and add individual loans after initial summary generation
**So that** I can see updated document counts and expand my bundle without starting over

---

### BRD Summary

#### Overview
After generating initial summary, users can refresh document status (useful when documents are uploaded) and add individual loans to the existing summary. This provides flexibility to build and refine bundles incrementally.

#### Business Value
- **Flexibility**: Adapt to changing document status
- **Efficiency**: Add loans without regenerating entire summary
- **Real-time**: See latest document counts as documents are uploaded
- **User Control**: Build bundles iteratively

#### Scope
- Refresh button to re-fetch document status
- Add Loan button to add individual loan
- Add Loan modal with validation
- Re-fetch API call for updated status
- Update grid with fresh data

---

### Detailed Requirements

#### Functional Requirements

**FR-042: Refresh Summary Button**
- System MUST provide refresh icon button in summary header
- Button visible only after summary has been generated
- Button disabled during loading/refreshing
- Clicking button triggers refresh confirmation modal

**FR-043: Refresh Confirmation Modal**
- Modal displays:
  - Title: "Refresh Summary"
  - Message: "This will re-fetch document status from the system. Continue?"
  - Buttons: "Yes" (green) | "No" (gray)
- Clicking "Yes" initiates refresh
- Clicking "No" closes modal, no action

**FR-044: Refresh Process**
- System calls EPS API with same loan list (re-fetches status)
- System displays loading indicator: "Refreshing..."
- System receives updated document counts
- System updates grid with new data
- User selection state preserved (checkboxes remain checked)

**FR-045: Add Loan Button**
- System MUST provide "Add Loan" button in action button row
- Button visible only after summary has been generated
- Button disabled during loading/building
- Clicking button opens Add Loan modal

**FR-046: Add Loan Modal**
- Modal displays:
  - Title: "Add Loan to Summary"
  - Input field: "Loan Number"
  - Buttons: "Add" (green) | "Cancel" (gray)
- Input field accepts alphanumeric loan numbers only
- Input field validates on blur and on submit

**FR-047: Add Loan Validation**
- System validates loan number format (same rules as manual entry)
- System checks for duplicates (loan already in summary)
- Validation errors display inline in modal (red text below input)
- Errors:
  - Empty: "Please enter a loan number"
  - Invalid format: "Invalid loan number format"
  - Duplicate: "This loan is already in the summary"

**FR-048: Add Loan Process**
- On valid loan number:
  - System adds loan to loan list
  - System calls API to fetch status for new loan only (or refresh all)
  - System updates grid with new loan
  - Modal closes automatically
  - Success toast: "Loan USB12345 added successfully"

**FR-049: Grid Update After Add/Refresh**
- New data merges with existing grid
- Sort order maintained (loan number or original order)
- Summary header updates (total count, missing count, complete count)
- Filter tabs update counts
- User remains on current filter tab

---

#### Technical Requirements

**TR-028: Refresh API Call**
- Re-use same API endpoint as initial summary generation
- Send same loan numbers array (no changes)
- Receive updated document counts for all loans
- Update state with new data

**TR-029: Add Loan API Call**
- Option 1: Call API for single loan only
  ```json
  {
    "bundleName": "U.S. Bank",
    "loanNumbers": ["USB12399"], // Only new loan
    "authToken": "Bearer <token>"
  }
  ```
- Option 2: Refresh all loans including new one
  ```json
  {
    "bundleName": "U.S. Bank",
    "loanNumbers": ["USB12345", "USB12346", ..., "USB12399"], // All loans
    "authToken": "Bearer <token>"
  }
  ```
- Merge new loan data into existing loans array in state

**TR-030: State Management**
- Preserve selection state during refresh:
  ```javascript
  const selectedLoanIds = loans.filter(l => l.selected).map(l => l.id);
  // ... refresh data ...
  const updatedLoans = newData.map(loan => ({
    ...loan,
    selected: selectedLoanIds.includes(loan.id)
  }));
  ```

**TR-031: Add Loan Validation**
```javascript
const validateNewLoan = (loanNumber) => {
  if (!loanNumber.trim()) {
    return 'Please enter a loan number';
  }
  if (!/^[A-Z0-9-]{5,20}$/i.test(loanNumber)) {
    return 'Invalid loan number format';
  }
  if (loans.some(l => l.id === loanNumber.trim())) {
    return 'This loan is already in the summary';
  }
  return null; // No error
};
```

---

### Acceptance Criteria

#### AC-069: Refresh Button Display
- [ ] Given I have generated summary with 27 loans
- [ ] When I view summary grid
- [ ] Then I see refresh icon button in header
- [ ] And button is enabled (not grayed out)

#### AC-070: Refresh Confirmation Modal
- [ ] Given I click refresh icon button
- [ ] When modal appears
- [ ] Then modal shows title "Refresh Summary"
- [ ] And modal shows message "This will re-fetch document status from the system. Continue?"
- [ ] And modal shows "Yes" and "No" buttons

#### AC-071: Refresh Confirmation - No
- [ ] Given refresh confirmation modal is open
- [ ] When I click "No"
- [ ] Then modal closes
- [ ] And grid remains unchanged
- [ ] And no API call is made

#### AC-072: Refresh Process
- [ ] Given I click "Yes" on refresh confirmation
- [ ] When refresh begins
- [ ] Then loading indicator appears: "Refreshing..."
- [ ] And API call is made with same 27 loan numbers
- [ ] When API returns updated data
- [ ] Then grid updates with new document counts
- [ ] And loading indicator disappears

#### AC-073: Refresh Preserves Selection
- [ ] Given I have selected 5 loans via checkboxes
- [ ] When I refresh summary
- [ ] Then after refresh completes
- [ ] Then the same 5 loans remain selected
- [ ] And checkboxes are still checked

#### AC-074: Refresh Updates Counts
- [ ] Given loan USB12345 had 3 missing docs before refresh
- [ ] And documents were uploaded for USB12345
- [ ] When I refresh summary
- [ ] Then loan USB12345 now shows 0 missing docs
- [ ] And status updates to "Ready to Bundle"
- [ ] And summary header updates: Complete count increases by 1

#### AC-075: Add Loan Button Display
- [ ] Given I have generated summary with 27 loans
- [ ] When I view action buttons
- [ ] Then I see "Add Loan" button
- [ ] And button is enabled

#### AC-076: Add Loan Modal Display
- [ ] Given I click "Add Loan" button
- [ ] When modal appears
- [ ] Then modal shows title "Add Loan to Summary"
- [ ] And modal shows input field labeled "Loan Number"
- [ ] And modal shows "Add" and "Cancel" buttons
- [ ] And input field is focused (cursor blinking)

#### AC-077: Add Loan - Success
- [ ] Given Add Loan modal is open
- [ ] When I enter valid loan number "USB12399"
- [ ] And I click "Add"
- [ ] Then modal closes
- [ ] And success toast appears: "Loan USB12399 added successfully"
- [ ] And API call is made for new loan
- [ ] And grid updates with 28 loans (was 27, now 28)
- [ ] And new loan appears in grid with document counts

#### AC-078: Add Loan - Empty Input
- [ ] Given Add Loan modal is open
- [ ] When I click "Add" without entering loan number
- [ ] Then error displays in modal: "Please enter a loan number"
- [ ] And modal remains open
- [ ] And no API call is made

#### AC-079: Add Loan - Invalid Format
- [ ] Given Add Loan modal is open
- [ ] When I enter loan number with special characters "USB@12399"
- [ ] And I click "Add"
- [ ] Then error displays in modal: "Invalid loan number format"
- [ ] And modal remains open
- [ ] And no API call is made

#### AC-080: Add Loan - Duplicate
- [ ] Given Add Loan modal is open
- [ ] When I enter loan number "USB12345" (already in summary)
- [ ] And I click "Add"
- [ ] Then error displays in modal: "This loan is already in the summary"
- [ ] And modal remains open
- [ ] And no API call is made

#### AC-081: Add Loan - Cancel
- [ ] Given Add Loan modal is open
- [ ] When I click "Cancel"
- [ ] Then modal closes
- [ ] And no loan is added
- [ ] And grid remains unchanged

#### AC-082: Summary Header Updates After Add
- [ ] Given I add 1 loan to summary of 27 loans
- [ ] When new loan is added successfully
- [ ] Then summary header updates: "Total Loans: 28" (was 27)
- [ ] And other counts update based on new loan's status

---

### Dependencies
- User Story 1: Manual Loan Entry (shares validation logic)
- User Story 3: Loan Summary Grid Display (updates grid)
- User Story 10: EPS/BytePro API Integration (fetches updated status)

---

### Risks & Mitigations
- **Risk**: Refresh takes long time for large loan lists
  - **Mitigation**: Display progress during refresh
- **Risk**: User adds same loan multiple times (duplicate check fails)
  - **Mitigation**: Robust duplicate detection, server-side validation
- **Risk**: Refresh loses user's work (selection, filters)
  - **Mitigation**: Preserve selection state, maintain active filter tab

---

<a name="user-story-8"></a>
## User Story 8: Build Status Tracking & Results Display

### Story ID
**BRD-BULK-008**

### Story Title
Build Status Tracking & Results Display

### User Story Statement
**As a** loan processor
**I want to** see real-time build progress and detailed results after build completes
**So that** I know which loans succeeded, which failed, and what actions to take next

---

### BRD Summary

#### Overview
During bundle build, users see real-time progress updates showing how many loans have been processed. After build completes, a results modal displays success/failure summary with details about any failed loans. This provides transparency and enables quick troubleshooting.

#### Business Value
- **Transparency**: Users know exactly what's happening during build
- **Confidence**: Real-time progress reduces anxiety about long operations
- **Actionability**: Clear failure details enable quick resolution
- **Efficiency**: Immediately identify and address problem loans

#### Scope
- Progress modal during build
- Real-time progress updates (percentage, loan count)
- Success/failure result modal
- Failure details (loan number, reason)
- Close page action to save to dashboard

---

### Detailed Requirements

#### Functional Requirements

**FR-050: Build Progress Modal**
- System MUST display progress modal when build starts
- Modal displays:
  - Title: "Building Bundle..."
  - Progress bar (animated or percentage-based)
  - Status text: "Processing X of Y loans (Z%)"
  - Modal cannot be dismissed (no close button or click-outside)
- Modal remains visible until build completes

**FR-051: Real-Time Progress Updates**
- System receives progress updates from API
- Updates received via WebSocket (preferred) or polling every 2 seconds
- Progress bar updates to reflect completion percentage
- Status text updates with current loan count
- Example: "Processing 12 of 27 loans (44%)"

**FR-052: Grid Status Updates During Build**
- As each loan completes, its status in grid updates:
  - "Ready to Bundle" → "In Progress" (when build starts)
  - "In Progress" → "Completed" (when build succeeds)
  - "In Progress" → "Bundle Failed" (when build fails)
- Status updates happen in real-time without full grid refresh
- Color coding updates automatically

**FR-053: Build Completion - All Success**
- When all loans complete successfully:
  - Close progress modal
  - Display success modal:
    - ✅ Green checkmark icon
    - Title: "Bundle Build Successful"
    - Message: "Successfully built X of X loans"
    - Details: "Bundle name: [bundle name]"
    - Button: "Close Page" (green)

**FR-054: Build Completion - Partial Success**
- When some loans succeed and some fail:
  - Close progress modal
  - Display partial success modal:
    - ⚠️ Yellow warning icon
    - Title: "Bundle Build Partially Completed"
    - Message: "Successfully built X of Y loans"
    - Details: "Z loans failed to build:"
      - List of failed loans with failure reasons
      - USB12349 - Missing required documents
      - USB12352 - PDF merge error
      - USB12358 - Document corrupted
    - Buttons: "Close Page" (green) | "Review Failures" (gray)

**FR-055: Build Completion - All Failed**
- When all loans fail:
  - Close progress modal
  - Display error modal:
    - ❌ Red X icon
    - Title: "Bundle Build Failed"
    - Message: "0 of X loans were successfully built"
    - Details: "Reason: [overall failure reason]"
    - Examples:
      - "BytePro service unavailable. Please try again later."
      - "Storage system error. Contact administrator."
    - Button: "Close" (red)

**FR-056: Close Page Action**
- Clicking "Close Page" button:
  1. Saves bundle to dashboard history with random status (demo behavior)
  2. Shows black screen for 5 seconds (simulates page close)
  3. Resets all form fields
  4. Navigates to Dashboard tab
  5. Displays updated dashboard with new entry

**FR-057: Review Failures Action**
- Available only in partial success modal
- Clicking "Review Failures":
  - Closes modal
  - Remains on Bundle Builder tab
  - Grid shows all loans with updated statuses
  - Failed loans highlighted in red
  - User can fix issues, refresh summary, rebuild failed loans

---

#### Technical Requirements

**TR-032: Progress Tracking**
- Poll build status every 2 seconds:
  ```javascript
  const pollBuildStatus = async (jobId) => {
    const interval = setInterval(async () => {
      const response = await fetch(`/api/v1/bundle-status/${jobId}`);
      const data = await response.json();

      updateProgress(data.completedLoans, data.totalLoans);
      updateLoanStatuses(data.loanStatuses);

      if (data.status === 'completed' || data.status === 'failed') {
        clearInterval(interval);
        showResultsModal(data);
      }
    }, 2000);
  };
  ```

**TR-033: Progress Calculation**
```javascript
const progress = (completedLoans / totalLoans) * 100;
const statusText = `Processing ${completedLoans} of ${totalLoans} loans (${progress}%)`;
```

**TR-034: Results Modal Logic**
```javascript
const showResultsModal = (buildData) => {
  const successCount = buildData.loanStatuses.filter(l => l.status === 'Completed').length;
  const failCount = buildData.loanStatuses.filter(l => l.status === 'Bundle Failed').length;
  const totalCount = buildData.totalLoans;

  if (successCount === totalCount) {
    showSuccessModal(successCount);
  } else if (successCount > 0 && failCount > 0) {
    showPartialSuccessModal(successCount, totalCount, failedLoans);
  } else {
    showErrorModal(buildData.failureReason);
  }
};
```

**TR-035: Dashboard Save**
- See User Story 5 for dashboard save logic
- Random status generation for demo:
  ```javascript
  const statuses = ['Complete', 'Incomplete', 'In progress'];
  const randomStatus = statuses[Math.floor(Math.random() * statuses.length)];
  ```

---

### Acceptance Criteria

#### AC-083: Progress Modal Display
- [ ] Given build has been initiated
- [ ] When build starts
- [ ] Then progress modal appears
- [ ] And modal shows "Building Bundle..."
- [ ] And modal shows progress bar at 0%
- [ ] And modal shows "Processing 0 of 27 loans (0%)"
- [ ] And modal cannot be closed (no close button)

#### AC-084: Progress Updates During Build
- [ ] Given build is processing 27 loans
- [ ] When 10 loans complete
- [ ] Then progress bar updates to 37%
- [ ] And status text updates: "Processing 10 of 27 loans (37%)"
- [ ] When 20 loans complete
- [ ] Then progress bar updates to 74%
- [ ] And status text updates: "Processing 20 of 27 loans (74%)"

#### AC-085: Grid Updates During Build
- [ ] Given build is in progress
- [ ] When loan USB12345 starts processing
- [ ] Then loan status in grid changes to "In Progress" (yellow badge)
- [ ] When loan USB12345 completes
- [ ] Then loan status changes to "Completed" (green badge with checkmark)

#### AC-086: Success Modal - All Loans Successful
- [ ] Given all 27 loans complete successfully
- [ ] When build finishes
- [ ] Then progress modal closes
- [ ] And success modal appears with green checkmark icon
- [ ] And modal title reads "Bundle Build Successful"
- [ ] And modal message reads "Successfully built 27 of 27 loans"
- [ ] And modal shows bundle name
- [ ] And modal has "Close Page" button

#### AC-087: Partial Success Modal
- [ ] Given 23 of 27 loans complete successfully, 4 fail
- [ ] When build finishes
- [ ] Then partial success modal appears with yellow warning icon
- [ ] And modal title reads "Bundle Build Partially Completed"
- [ ] And modal message reads "Successfully built 23 of 27 loans"
- [ ] And modal shows "4 loans failed to build:"
- [ ] And modal lists 4 failed loans with failure reasons:
  - USB12349 - Missing required documents
  - USB12352 - PDF merge error
  - USB12358 - Document corrupted
  - USB12361 - Timeout error
- [ ] And modal has "Close Page" and "Review Failures" buttons

#### AC-088: Error Modal - All Loans Failed
- [ ] Given all 27 loans fail to build
- [ ] When build finishes
- [ ] Then error modal appears with red X icon
- [ ] And modal title reads "Bundle Build Failed"
- [ ] And modal message reads "0 of 27 loans were successfully built"
- [ ] And modal shows failure reason: "BytePro service unavailable. Please try again later."
- [ ] And modal has "Close" button

#### AC-089: Close Page Action
- [ ] Given success modal displays
- [ ] When I click "Close Page"
- [ ] Then modal closes
- [ ] And black screen appears for 5 seconds
- [ ] And bundle is saved to dashboard with random status
- [ ] And all form fields reset to empty
- [ ] And system navigates to Dashboard tab
- [ ] And new entry appears at top of dashboard (light blue background)

#### AC-090: Review Failures Action
- [ ] Given partial success modal displays
- [ ] When I click "Review Failures"
- [ ] Then modal closes
- [ ] And I remain on Bundle Builder tab
- [ ] And grid shows all 27 loans
- [ ] And 4 failed loans are highlighted in red
- [ ] And I can refresh summary or rebuild failed loans

#### AC-091: Failed Loan Details in Grid
- [ ] Given loan USB12349 failed with reason "Missing required documents"
- [ ] When I view grid after build completes
- [ ] Then loan USB12349 shows status "Bundle Failed" (red badge)
- [ ] And hovering over status shows tooltip with failure reason

#### AC-092: Build Timeout
- [ ] Given build is taking longer than 2 minutes
- [ ] When timeout is reached
- [ ] Then timeout modal displays
- [ ] And modal asks "Request is taking longer than expected. Continue waiting or cancel?"
- [ ] When I click "Continue Waiting"
- [ ] Then build continues for 2 more minutes

---

### Dependencies
- User Story 4: Bundle Build Execution (initiates build)
- User Story 5: Dashboard History (saves completed bundle)
- User Story 10: EPS/BytePro API Integration (provides build status)

---

### Risks & Mitigations
- **Risk**: Progress updates lag or freeze
  - **Mitigation**: Implement timeout, display "Still processing..." message
- **Risk**: Too many failure details overwhelm modal
  - **Mitigation**: Limit to first 10 failures, add "View All Failures" option
- **Risk**: User closes browser during build
  - **Mitigation**: Warn user on navigation, provide job ID to check status later

---

<a name="user-story-9"></a>
## User Story 9: User Authentication & Session Management

### Story ID
**BRD-BULK-009**

### Story Title
User Authentication & Session Management

### User Story Statement
**As a** system user
**I want to** securely authenticate via SSO and maintain my session
**So that** I can access the platform securely and my identity is verified

---

### BRD Summary

#### Overview
Users authenticate via company SSO (Single Sign-On) provider, which generates a secure authentication token. The platform maintains user session throughout their workflow and handles session expiration gracefully.

#### Business Value
- **Security**: Centralized authentication reduces security risks
- **Compliance**: Audit trail of user actions tied to identity
- **User Experience**: Single login for multiple company systems
- **Administration**: Centralized user management via SSO

#### Scope
- SSO integration (SAML 2.0 or OAuth 2.0)
- Token-based authentication
- Session management
- Session expiration handling
- User profile display
- Logout functionality

---

### Detailed Requirements

#### Functional Requirements

**FR-058: Initial Access**
- User navigates to platform URL (e.g., https://bob.company.com or localhost:5173)
- System detects no active session
- System redirects to SSO login page

**FR-059: SSO Authentication**
- User enters credentials on SSO page (username/password + MFA if required)
- SSO validates credentials against company directory (Active Directory/LDAP)
- SSO generates authentication token (JWT)
- SSO redirects back to platform with token

**FR-060: Token Validation**
- Platform receives authentication token
- Platform validates token signature and expiration
- Platform calls EPS API to retrieve user profile
- Platform stores token securely (HttpOnly cookie or secure session storage)

**FR-061: User Profile Display**
- System displays user information in header:
  - Avatar with initials (circular badge)
  - Full name
  - Role/title
- Clicking avatar opens user dropdown menu

**FR-062: User Dropdown Menu**
- Dropdown displays:
  - User name
  - Email address
  - Role
  - Divider
  - "My Account" option
  - "Settings" option
  - "Switch Profile" option (if multi-profile support)
  - Divider
  - "Switch to [other app]" options
  - Divider
  - "Log Out" option

**FR-063: Session Maintenance**
- Token remains valid for session duration (e.g., 8 hours)
- Token included in all API requests (Authorization header)
- Token automatically refreshed if refresh token available
- Idle timeout after 30 minutes of inactivity (configurable)

**FR-064: Session Expiration Handling**
- When token expires, API returns 401 Unauthorized
- System displays session expiration modal
- Modal message: "Your session has expired. Please log in again."
- User clicks "Log In" → redirect to SSO login
- After re-authentication, return user to previous page (preserve state if possible)

**FR-065: Logout**
- User clicks "Log Out" in dropdown menu
- System clears stored token
- System calls SSO logout endpoint
- System redirects to SSO logout page
- User receives confirmation: "You have been logged out"

**FR-066: Role-Based Access Control (Future)**
- System checks user role before allowing bundle creation
- Roles examples:
  - Loan Processor: Full access
  - Loan Officer: View-only access
  - Manager: Full access + admin features
- Unauthorized users see error: "You do not have permission to perform this action"

---

#### Technical Requirements

**TR-036: SSO Integration**
- Protocol: SAML 2.0 or OAuth 2.0 (configured per company)
- SSO Provider: Okta, Azure AD, or custom SSO
- Redirect URLs configured in SSO provider settings

**TR-037: Token Format (JWT)**
```json
{
  "sub": "user@company.com",
  "name": "John Doe",
  "role": "Loan Processor",
  "exp": 1642612345,
  "iat": 1642612345,
  "iss": "company-sso"
}
```

**TR-038: Token Storage**
- Option 1: HttpOnly cookie (preferred for security)
  - Cookie name: `auth_token`
  - Flags: HttpOnly, Secure, SameSite=Strict
- Option 2: Secure session storage (fallback)
  - Store token in memory, not localStorage

**TR-039: API Request with Token**
```javascript
const response = await fetch('/api/v1/bulk-loan-documents', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${authToken}`
  },
  body: JSON.stringify({ bundleName, loanNumbers })
});
```

**TR-040: Token Validation**
```javascript
const validateToken = async (token) => {
  try {
    const response = await fetch('/api/v1/validate-token', {
      headers: { 'Authorization': `Bearer ${token}` }
    });
    return response.ok;
  } catch (error) {
    return false;
  }
};
```

**TR-041: Session Expiration Detection**
```javascript
const handleAPIError = (error) => {
  if (error.response && error.response.status === 401) {
    // Token expired
    showSessionExpiredModal();
    redirectToLogin();
  }
};
```

---

### Acceptance Criteria

#### AC-093: Initial Access - No Session
- [ ] Given I have not logged in yet
- [ ] When I navigate to platform URL
- [ ] Then system detects no active session
- [ ] And system redirects me to SSO login page

#### AC-094: SSO Login Success
- [ ] Given I am on SSO login page
- [ ] When I enter valid credentials (username, password, MFA)
- [ ] And I submit login form
- [ ] Then SSO validates my credentials
- [ ] And SSO redirects me back to platform with auth token
- [ ] And platform validates token
- [ ] And platform displays dashboard

#### AC-095: User Profile Display
- [ ] Given I am logged in
- [ ] When I view platform header
- [ ] Then I see circular avatar with my initials (e.g., "JD")
- [ ] And avatar is in top-right corner
- [ ] And avatar has teal background

#### AC-096: User Dropdown Menu
- [ ] Given I am logged in
- [ ] When I click avatar in header
- [ ] Then dropdown menu appears
- [ ] And menu shows my full name, email, and role
- [ ] And menu shows options: "My Account", "Settings", "Log Out"
- [ ] And menu shows "Switch to [app]" options

#### AC-097: Token Included in API Requests
- [ ] Given I am logged in and make API call
- [ ] When request is sent to EPS API
- [ ] Then request includes Authorization header
- [ ] And header contains "Bearer <token>"
- [ ] And API validates token and processes request

#### AC-098: Session Expiration Modal
- [ ] Given my token has expired (8 hours elapsed)
- [ ] When I make any API call
- [ ] Then API returns 401 Unauthorized
- [ ] And session expiration modal appears
- [ ] And modal message reads "Your session has expired. Please log in again."
- [ ] And modal has "Log In" button

#### AC-099: Re-Authentication After Expiration
- [ ] Given session expiration modal is displayed
- [ ] When I click "Log In"
- [ ] Then system redirects to SSO login page
- [ ] When I re-authenticate
- [ ] Then system returns me to platform
- [ ] And I can continue my work

#### AC-100: Logout Success
- [ ] Given I am logged in
- [ ] When I click avatar and select "Log Out"
- [ ] Then system clears my auth token
- [ ] And system redirects to SSO logout page
- [ ] And I see confirmation "You have been logged out"
- [ ] When I try to access platform again
- [ ] Then I must log in again

#### AC-101: Idle Timeout (Future)
- [ ] Given I am logged in but inactive for 30 minutes
- [ ] When idle timeout is reached
- [ ] Then system displays warning: "You will be logged out in 60 seconds due to inactivity"
- [ ] When I click "Stay Logged In"
- [ ] Then timeout resets and session continues

#### AC-102: Role-Based Access (Future)
- [ ] Given I am logged in as "View-Only" user
- [ ] When I try to click "Build Bundle"
- [ ] Then system displays error: "You do not have permission to perform this action"
- [ ] And build does not execute

---

### Dependencies
- All other user stories (authentication required for all operations)
- SSO provider configuration (external dependency)

---

### Risks & Mitigations
- **Risk**: SSO provider unavailable
  - **Mitigation**: Display error message, provide support contact
- **Risk**: Token stolen or compromised
  - **Mitigation**: Use HTTPS, HttpOnly cookies, short token expiration
- **Risk**: User loses work when session expires
  - **Mitigation**: Implement auto-save, preserve form state

---

<a name="user-story-10"></a>
## User Story 10: EPS/BytePro API Integration

### Story ID
**BRD-BULK-010**

### Story Title
EPS/BytePro API Integration

### User Story Statement
**As a** the system
**I want to** integrate with EPS API to leverage BytePro's bundling logic
**So that** I can retrieve document status and generate PDF bundles using proven backend services

---

### BRD Summary

#### Overview
The Bulk Bundle Manager integrates with CMG Financial's EPS (Enterprise Processing System) API, which serves as a gateway to BytePro's proven document bundling logic. All document retrieval, validation, and bundle generation happens through this API layer.

#### Business Value
- **Reliability**: Leverage BytePro's production-proven bundling engine
- **Consistency**: Same logic as existing BytePro system ensures compatibility
- **Scalability**: EPS handles load balancing and scaling
- **Maintainability**: Backend changes don't require frontend updates

#### Scope
- API endpoint definitions
- Request/response formats
- Authentication handling
- Error response handling
- Retry logic
- Performance optimization

---

### Detailed Requirements

#### Functional Requirements

**FR-067: Document Status Endpoint**
- Endpoint: `POST /api/v1/bulk-loan-documents`
- Purpose: Retrieve document status for multiple loans
- Request includes: bundleName, loanNumbers array, authToken
- Response includes: Array of loan document statuses
- Timeout: 2 minutes
- Retry: Up to 3 retries on network failure

**FR-068: Bundle Build Endpoint**
- Endpoint: `POST /api/v1/build-bundle`
- Purpose: Initiate bundle build job
- Request includes: bundleName, loanNumbers, buildMode, authToken
- Response includes: Job ID, estimated completion time
- Timeout: 30 seconds (for job creation, not build completion)
- Retry: Up to 3 retries on network failure

**FR-069: Build Status Endpoint**
- Endpoint: `GET /api/v1/bundle-status/{jobId}`
- Purpose: Check progress of bundle build
- Request includes: jobId, authToken
- Response includes: Status, completed count, loan statuses
- Polling: Every 2 seconds during build
- Timeout: N/A (poll until complete or user cancels)

**FR-070: User Profile Endpoint**
- Endpoint: `GET /api/v1/user/profile`
- Purpose: Retrieve authenticated user's profile
- Request includes: authToken
- Response includes: Name, email, role, permissions
- Called once on login

**FR-071: Token Validation Endpoint**
- Endpoint: `GET /api/v1/validate-token`
- Purpose: Validate authentication token
- Request includes: authToken
- Response: 200 OK (valid) or 401 Unauthorized (invalid/expired)

**FR-072: Error Responses**
- All endpoints return consistent error format:
  ```json
  {
    "success": false,
    "error": {
      "code": "INVALID_LOAN_NUMBER",
      "message": "Loan USB99999 not found in system",
      "details": { ... }
    }
  }
  ```
- HTTP status codes:
  - 200: Success
  - 400: Bad request (validation error)
  - 401: Unauthorized (expired/invalid token)
  - 404: Not found (loan not found, bundle not found)
  - 500: Server error (BytePro unavailable, storage error)
  - 503: Service unavailable (maintenance mode)

---

#### Technical Requirements

**TR-042: Document Status Request**
```json
POST /api/v1/bulk-loan-documents
{
  "bundleName": "U.S. Bank",
  "loanNumbers": ["USB12345", "USB12346", "USB12347"],
  "authToken": "Bearer eyJhbGc...",
  "requestTimestamp": "2025-01-19T10:30:00Z"
}
```

**TR-043: Document Status Response**
```json
{
  "success": true,
  "loans": [
    {
      "loanNumber": "USB12345",
      "foundDocs": 58,
      "missingDocs": 3,
      "bundleStatus": "Not Bundled",
      "missingDocTypes": ["VOE", "VOD", "Closing Disclosure"],
      "lastUpdated": "2025-01-19T10:30:15Z"
    },
    {
      "loanNumber": "USB12346",
      "foundDocs": 61,
      "missingDocs": 0,
      "bundleStatus": "Ready to Bundle",
      "missingDocTypes": [],
      "lastUpdated": "2025-01-19T10:30:16Z"
    }
  ],
  "totalLoans": 2,
  "timestamp": "2025-01-19T10:30:20Z"
}
```

**TR-044: Bundle Build Request**
```json
POST /api/v1/build-bundle
{
  "bundleName": "U.S. Bank",
  "loanNumbers": ["USB12345", "USB12346"],
  "buildMode": "selected", // or "ready"
  "authToken": "Bearer eyJhbGc...",
  "requestTimestamp": "2025-01-19T11:00:00Z"
}
```

**TR-045: Bundle Build Response**
```json
{
  "success": true,
  "jobId": "JOB-12345-67890",
  "totalLoans": 2,
  "message": "Bundle build initiated",
  "estimatedCompletionTime": "2025-01-19T11:05:00Z",
  "timestamp": "2025-01-19T11:00:05Z"
}
```

**TR-046: Build Status Request**
```
GET /api/v1/bundle-status/JOB-12345-67890
Authorization: Bearer eyJhbGc...
```

**TR-047: Build Status Response**
```json
{
  "success": true,
  "jobId": "JOB-12345-67890",
  "status": "in_progress", // or "completed", "failed"
  "totalLoans": 2,
  "completedLoans": 1,
  "failedLoans": 0,
  "loanStatuses": [
    {
      "loanNumber": "USB12345",
      "status": "Completed",
      "bundleUrl": "https://storage.company.com/bundles/USB12345.pdf",
      "fileSize": 25600000, // bytes
      "pageCount": 127
    },
    {
      "loanNumber": "USB12346",
      "status": "In Progress",
      "progress": 45 // percentage
    }
  ],
  "timestamp": "2025-01-19T11:02:30Z"
}
```

**TR-048: API Client Implementation**
```javascript
class EPSAPIClient {
  constructor(baseURL, authToken) {
    this.baseURL = baseURL;
    this.authToken = authToken;
  }

  async getBulkLoanDocuments(bundleName, loanNumbers) {
    const response = await this.post('/bulk-loan-documents', {
      bundleName,
      loanNumbers,
      authToken: this.authToken,
      requestTimestamp: new Date().toISOString()
    });
    return response.data;
  }

  async buildBundle(bundleName, loanNumbers, buildMode) {
    const response = await this.post('/build-bundle', {
      bundleName,
      loanNumbers,
      buildMode,
      authToken: this.authToken,
      requestTimestamp: new Date().toISOString()
    });
    return response.data;
  }

  async getBuildStatus(jobId) {
    const response = await this.get(`/bundle-status/${jobId}`);
    return response.data;
  }

  async post(endpoint, data) {
    return await this.fetchWithRetry(
      `${this.baseURL}${endpoint}`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': this.authToken
        },
        body: JSON.stringify(data)
      }
    );
  }

  async get(endpoint) {
    return await this.fetchWithRetry(
      `${this.baseURL}${endpoint}`,
      {
        method: 'GET',
        headers: {
          'Authorization': this.authToken
        }
      }
    );
  }

  async fetchWithRetry(url, options, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
      try {
        const response = await fetch(url, options);
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        return await response.json();
      } catch (error) {
        if (i === maxRetries - 1) throw error;
        await this.delay(2000);
      }
    }
  }

  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

---

### Acceptance Criteria

#### AC-103: Document Status API Success
- [ ] Given I make API call to /bulk-loan-documents with 27 loan numbers
- [ ] When API processes request
- [ ] Then API returns 200 OK response
- [ ] And response contains array of 27 loan objects
- [ ] And each loan object has foundDocs, missingDocs, bundleStatus fields
- [ ] And response time is under 10 seconds

#### AC-104: Document Status API - Loan Not Found
- [ ] Given I make API call with loan number "USB99999" that doesn't exist
- [ ] When API processes request
- [ ] Then API returns 404 Not Found for that specific loan
- [ ] And response message reads "Loan USB99999 not found in system"
- [ ] And other valid loans still return successful status

#### AC-105: Document Status API - Invalid Token
- [ ] Given I make API call with expired authentication token
- [ ] When API validates token
- [ ] Then API returns 401 Unauthorized
- [ ] And response message reads "Authentication token expired"

#### AC-106: Bundle Build API Success
- [ ] Given I make API call to /build-bundle with 27 loan numbers
- [ ] When API processes request
- [ ] Then API returns 200 OK response
- [ ] And response contains jobId for tracking
- [ ] And response contains estimatedCompletionTime
- [ ] And response time is under 30 seconds (for job creation)

#### AC-107: Bundle Build API - BytePro Unavailable
- [ ] Given BytePro service is down
- [ ] When I make API call to /build-bundle
- [ ] Then API returns 503 Service Unavailable
- [ ] And response message reads "Bundle service temporarily unavailable. Please try again later."

#### AC-108: Build Status API - In Progress
- [ ] Given I poll /bundle-status/{jobId} during active build
- [ ] When API returns status
- [ ] Then status field reads "in_progress"
- [ ] And completedLoans field shows current count (e.g., 12)
- [ ] And totalLoans field shows total count (e.g., 27)
- [ ] And loanStatuses array shows current status for each loan

#### AC-109: Build Status API - Completed
- [ ] Given I poll /bundle-status/{jobId} after build completes
- [ ] When API returns status
- [ ] Then status field reads "completed"
- [ ] And completedLoans equals totalLoans
- [ ] And each loanStatus has bundleUrl, fileSize, pageCount
- [ ] And all loans show status "Completed" or "Bundle Failed"

#### AC-110: API Retry on Network Failure
- [ ] Given network connection is lost during API call
- [ ] When first request fails
- [ ] Then system automatically retries after 2 seconds
- [ ] When second retry fails
- [ ] Then system retries again after 2 seconds
- [ ] When third retry succeeds
- [ ] Then system processes response normally

#### AC-111: API Retry Exhausted
- [ ] Given network connection is lost
- [ ] When all 3 retries fail
- [ ] Then system displays error modal
- [ ] And modal shows "Network error. Please check your connection and try again."
- [ ] And modal has manual "Retry" button

#### AC-112: API Timeout
- [ ] Given API call takes longer than 2 minutes
- [ ] When timeout threshold is reached
- [ ] Then system cancels request
- [ ] And system displays timeout error
- [ ] And user can retry manually

---

### Dependencies
- All other user stories (all features depend on API integration)
- EPS API service (external dependency)
- BytePro service (external dependency)
- Network infrastructure

---

### Risks & Mitigations
- **Risk**: API endpoint changes break frontend
  - **Mitigation**: API versioning (/api/v1/), contract testing
- **Risk**: API performance degrades with large loan lists
  - **Mitigation**: Batch processing, pagination if needed
- **Risk**: BytePro service unavailable
  - **Mitigation**: Graceful error handling, retry logic, user notification

---

## Appendix: Epic Summary

### Epic Overview
**Epic Name**: Bulk Bundle Manager - Core Functionality
**Epic ID**: EPIC-BULK-001
**Epic Owner**: Product Manager
**Target Release**: Q1 2025

### User Stories Summary

| Story ID | Story Title | Priority | Estimate | Dependencies |
|----------|-------------|----------|----------|--------------|
| BRD-BULK-001 | Manual Loan Entry | High | 8 pts | 003, 006, 010 |
| BRD-BULK-002 | Bulk CSV Upload | High | 5 pts | 001, 003, 006 |
| BRD-BULK-003 | Loan Summary Grid | High | 13 pts | 001, 002, 010 |
| BRD-BULK-004 | Bundle Build Execution | Critical | 8 pts | 003, 008, 010 |
| BRD-BULK-005 | Dashboard History | Medium | 8 pts | 004 |
| BRD-BULK-006 | Error Handling | High | 13 pts | All |
| BRD-BULK-007 | Refresh & Add Loans | Low | 5 pts | 001, 003, 010 |
| BRD-BULK-008 | Build Status Tracking | High | 8 pts | 004, 005, 010 |
| BRD-BULK-009 | Authentication | Critical | 13 pts | All (External SSO) |
| BRD-BULK-010 | API Integration | Critical | 21 pts | All |

**Total Estimated Effort**: 102 Story Points

### Implementation Phases

**Phase 1: Foundation (Weeks 1-2)**
- BRD-BULK-009: Authentication
- BRD-BULK-010: API Integration
- BRD-BULK-006: Error Handling

**Phase 2: Core Features (Weeks 3-4)**
- BRD-BULK-001: Manual Loan Entry
- BRD-BULK-003: Loan Summary Grid
- BRD-BULK-002: Bulk CSV Upload

**Phase 3: Build Functionality (Week 5)**
- BRD-BULK-004: Bundle Build Execution
- BRD-BULK-008: Build Status Tracking

**Phase 4: Polish & Enhancements (Week 6)**
- BRD-BULK-005: Dashboard History
- BRD-BULK-007: Refresh & Add Loans
- Testing & Bug Fixes

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Product Owner | TBD | _________ | __/__/__ |
| Business Analyst | TBD | _________ | __/__/__ |
| Development Lead | TBD | _________ | __/__/__ |
| QA Lead | TBD | _________ | __/__/__ |
| Stakeholder | TBD | _________ | __/__/__ |

---

**End of BRD Document**
