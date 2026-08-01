# Member Management

## SCREEN 1 : MEMBER LIST

### 1. Overview
The **Member List** screen serves as the centralized command center for managing the complete lifecycle of all PG members. By eliminating the siloed approach of separate rent and member modules, this unified view displays comprehensive tenant profiles alongside their real-time financial standing. Administrators can monitor accommodation allocations, track upcoming rent dues, verify pending payments, and perform essential actions (View, Edit, Deactivate, Payment History, Payment Verification) directly from a single, highly filterable, and sortable data table.

### 2. Screen Preview
```text
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Member Management                                                                                       Add New Member                                                  |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Search [_________________________]        Filters        Export        Refresh                                                                                            |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ID | Member | Mobile | PG | Room | Bed | Monthly Rent | Due Date | Rent Status | Payment Status | Member Status | Actions                                             |
|----|--------|--------|----|------|-----|--------------|----------|-------------|----------------|---------------|-----------------------------------------------------|
| M01| Rahul  | 9876.. | PG1| 101  | A   | 6000.00      | 05-Aug   | Pending     | Pending        | Active        | View | Edit | History | Verify | Deactivate       |
| M02| Amit   | 9123.. | PG2| 202  | B   | 7500.00      | 01-Aug   | Overdue     | In Review      | Notice Period | View | Edit | History | Verify | Deactivate       |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Showing 1-10 of 150 Records                                                                    Previous | Next                                                          |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table
| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Add New Member** | Button | N/A | Redirects to Add Member screen. | N/A | Primary CTA. |
| **Search** | Text Input | No | Accepts alphanumeric input. Supports partial match. | `Rahul` | Searches across ID, Name, Mobile, PG, Room. |
| **Filters** | Button | N/A | Opens advanced filter modal. | N/A | Controls the Filter Configuration constraints. |
| **Export** | Button | N/A | Exports current filtered view to CSV/Excel. | N/A | - |
| **Refresh** | Button | N/A | Reloads data from server. | N/A | Useful for live status updates. |
| **ID** | Data Column | N/A | Sortable. | `M01` | Unique system identifier. |
| **Member** | Data Column | N/A | Sortable. | `Rahul Patel` | Full Name. |
| **Mobile** | Data Column | N/A | Sortable. 10 digits. | `9876543210` | Formatted for readability. |
| **PG** | Data Column | N/A | Sortable. | `Sunshine PG` | Property name. |
| **Room** | Data Column | N/A | Sortable. | `101` | - |
| **Bed** | Data Column | N/A | Sortable. | `A` | - |
| **Monthly Rent** | Data Column | N/A | Sortable. Currency format. | `6000.00` | Current base rent. |
| **Due Date** | Data Column | N/A | Sortable. Date format. | `05-Aug` | Next rent due date. |
| **Rent Status** | Data Column | N/A | Sortable. (Paid, Pending, Overdue). | `Pending` | Operational status of the current billing cycle. |
| **Payment Status** | Data Column | N/A | Sortable. (Pending, In Review, Paid, Rejected). | `In Review` | Verification status of the submitted transaction. |
| **Member Status** | Data Column | N/A | Sortable. (Active, Notice Period, Inactive). | `Active` | Current occupancy phase. |
| **View** | Action Link | N/A | Redirects to View Member screen. | N/A | - |
| **Edit** | Action Link | N/A | Redirects to Edit Member screen. | N/A | - |
| **History** | Action Link | N/A | Redirects to Payment History screen. | N/A | - |
| **Verify** | Action Link | N/A | Redirects to Payment Verification screen. | N/A | Disabled/Hidden if Payment Status != 'In Review'. |
| **Deactivate** | Action Link | N/A | Opens Deactivate Member modal. | N/A | Disabled if Member Status is already Inactive. |
| **Records Per Page** | Dropdown | No | Valid integer selection (10, 20, 50). | `10` | Controls page size. |
| **Pagination (Prev/Next)** | Buttons | N/A | Disabled if on the first/last page. | N/A | - |

### 4. Validations
*   **Search Input Validation:** The search field must gracefully handle leading/trailing spaces and ignore case sensitivity. It must execute search across combined member and financial data fields.
*   **Action Button States:** The "Verify" action button must only be active (clickable) for records where the `Payment Status` is strictly 'In Review'. Otherwise, it should be disabled or hidden.
*   **Deactivate Constraint:** The "Deactivate" action must be disabled for members whose status is already 'Inactive'.
*   **Export Constraints:** The export function must strictly respect applied search queries, sorting, and active filters, ensuring the downloaded file matches the user's current visual state.
*   **Pagination Reset:** Whenever a new Search query or Filter is applied, the pagination must automatically reset to Page 1 to prevent out-of-bounds errors.

---

## SCREEN 2 : ADD MEMBER

### 1. Overview
The **Add Member** screen facilitates the comprehensive registration of a new PG tenant. This unified form simultaneously captures personal demographics, identity verification documents, emergency contacts, property allocation details, and the core financial configuration (rent amount, security deposit, due dates). Successfully submitting this form establishes the member's profile and automatically initializes their primary financial ledger in the system.

### 2. Screen Preview
```text
+-------------------------------------------------------------------------------------------------+
| Add New Member                                                                                  |
|-------------------------------------------------------------------------------------------------|
| Personal Information                                                                            |
| Member ID *                  Full Name *                  Mobile Number *                       |
| [ Auto-generated ]           [______________________]     [______________________]              |
| Alternate Mobile Number      Email                        Occupation *                          |
| [______________________]     [______________________]     [ Dropdown v ]                        |
| Date of Birth *              Gender *                     Company / College Name *              |
| [ DD/MM/YYYY ]               [ Dropdown v ]               [______________________]              |
|                                                                                                 |
| Identity Verification                                                                           |
| Aadhaar Card *                                                                                  |
| [ Text Input ] [ Choose File ]                                                                  |
| PAN                                                                                             |
| [ Text Input ] [ Choose File ]                                                                  |
| Driving Licence                                                                                 |
| [ Text Input ] [ Choose File ]                                                                  |
|                                                                                                 |
| Emergency Contact                                                                               |
| Contact Person Name *        Relationship *                                                     |
| [______________________]     [ Dropdown v ]                                                     |
|                                                                                                 |
| Address                                                                                         |
| Address Line 1 *             Address Line 2               City *                                |
| [______________________]     [______________________]     [______________________]              |
| State *                      Pincode *                    Country                               |
| [______________________]     [______________________]     [______________________]              |
|                                                                                                 |
| Stay Details                                                                                    |
| PG Name *                    Room Number *                Bed *                                 |
| [ Dropdown v ]               [ Dropdown v ]               [ Dropdown v ]                        |
|                                                                                                 |
| Rent Details                                                                                    |
| Monthly Rent *               Security Deposit *           Maintenance Charge *                  |
| [______________________]     [______________________]     [______________________]              |
| Rent Due Date *              Notice Period *                                                    |
| [ Dropdown (1-31) v ]        [ Dropdown (Days) v ]                                              |
|                                                                                                 |
| Member Status                                                                                   |
| Status *                                                                                        |
| [ Dropdown v ]                                                                                  |
|                                                                                                 |
|                            [ Cancel ] [ Save ]                                                  |
+-------------------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table
| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Member ID** | Text Input | Yes | System-generated, read-only. Alphanumeric. | `M1042` | Unique key. |
| **Full Name** | Text Input | Yes | Max 100 chars. Alphabetic. | `Rahul Patel` | - |
| **Mobile Number** | Text Input | Yes | Exactly 10 digits. Numeric. | `9876543210` | - |
| **Alternate Mobile Number**| Text Input | No | Exactly 10 digits if provided. | `9123456780` | - |
| **Email** | Text Input | No | Valid email format. Max 100 chars. | `rahul@email.com`| - |
| **Occupation** | Dropdown | Yes | Student, Employee, Business, Freelancer, Other. | `Student` | - |
| **Date of Birth** | Date Picker| Yes | Valid past date. Minimum age threshold (e.g. 16+).| `15/08/2000` | - |
| **Gender** | Dropdown | Yes | Male, Female, Other. | `Male` | Used for room assignment logic. |
| **Company/College Name** | Text Input | Yes | Max 100 chars. Alphanumeric. | `ABC University` | - |
| **Aadhaar Card (Text)** | Text Input | Yes | Exactly 12 digits. Numeric. | `123456789012` | - |
| **Aadhaar Card (Upload)** | File Input | Yes | `.jpg`, `.png`, `.pdf`. Max 5MB. | `aadhaar.pdf` | - |
| **PAN (Text)** | Text Input | No | Valid 10 char alphanumeric PAN format. | `ABCDE1234F` | - |
| **PAN (Upload)** | File Input | No | `.jpg`, `.png`, `.pdf`. Max 5MB. | `pan.jpg` | - |
| **Driving Licence (Text)** | Text Input | No | Valid alphanumeric format. | `DL142011001` | - |
| **Driving Licence (Upload)**| File Input | No | `.jpg`, `.png`, `.pdf`. Max 5MB. | `dl.png` | - |
| **Contact Person Name** | Text Input | Yes | Max 100 chars. Alphabetic. | `Ajay Patel` | - |
| **Relationship** | Dropdown | Yes | Father, Mother, Brother, Sister, Friend, Guardian, Other. | `Father` | - |
| **Address Line 1** | Text Input | Yes | Max 255 chars. | `Plot 42, Sector 1` | - |
| **Address Line 2** | Text Input | No | Max 255 chars. | `Opposite Park` | - |
| **City** | Text Input | Yes | Max 100 chars. | `Bengaluru` | - |
| **State** | Text Input | Yes | Max 100 chars. | `Karnataka` | - |
| **Pincode** | Text Input | Yes | Exactly 6 digits. Numeric. | `560034` | - |
| **Country** | Text Input | No | Max 100 chars. Default 'India'. | `India` | - |
| **PG Name** | Dropdown | Yes | Pre-populated list of Active PGs. | `Sunshine PG` | - |
| **Room Number** | Dropdown | Yes | Pre-populated list based on selected PG. | `102` | - |
| **Bed** | Dropdown | Yes | Pre-populated available beds based on selected Room. | `A` | - |
| **Monthly Rent** | Number Input| Yes | Positive numeric value. | `6000` | - |
| **Security Deposit** | Number Input| Yes | Positive numeric value. | `12000` | - |
| **Maintenance Charge** | Number Input| Yes | Positive numeric value or zero. | `500` | - |
| **Rent Due Date** | Dropdown | Yes | Integer from 1 to 31. | `5` | Day of the month rent is expected. |
| **Notice Period** | Dropdown | Yes | Predefined options (e.g., 15 Days, 30 Days). | `30 Days` | - |
| **Status** | Dropdown | Yes | Active, Notice Period, Inactive. | `Active` | - |
| **Save** | Button | N/A | Submits the form data. | N/A | Disabled during API call. |
| **Cancel** | Button | N/A | Discards form and returns to Member List. | N/A | - |

### 4. Validations
*   **Mandatory Fields:** Ensure all fields marked with an asterisk (*) are strictly validated on form submission. Display inline field-level errors for missing data.
*   **Mobile Number Uniqueness:** The `Mobile Number` must be validated against the database to ensure no duplicate active tenant exists with the same number.
*   **Age Verification:** The `Date of Birth` must be validated to ensure the tenant meets the minimum age requirement (e.g., 16 years or older).
*   **Document Format & Size:** Upload fields must strictly reject non-supported MIME types and files exceeding the 5MB size limit.
*   **Cascading Dropdowns (Stay Details):** 
    *   `Room Number` dropdown must remain disabled until a `PG Name` is selected.
    *   `Bed` dropdown must remain disabled until a `Room Number` is selected. It must only display unoccupied/available beds.
*   **Financial Formats:** Rent, Security Deposit, and Maintenance Charge must not accept negative numbers or non-numeric characters.
*   **Conditional Display (Status):** The `Reason` text area must remain completely hidden unless the `Status` dropdown is changed to 'Notice Period'. If visible, it becomes a mandatory field.

---

## SCREEN 3 : VIEW MEMBER

### 1. Overview
The **View Member** screen provides a comprehensive, read-only 360-degree profile of a specific tenant. Utilizing a tabbed interface, it categorizes the vast array of member data into logical segments—personal details, documentation, allocation, and complete financial history. This screen empowers administrators to rapidly access critical information and audit a member's entire lifecycle without the risk of accidental data modification.

### 2. Screen Preview
```text
+-------------------------------------------------------------------------------------------------+
| View Member Details : Rahul Patel (M01)                                                         |
|-------------------------------------------------------------------------------------------------|
| [ Personal Information ] [ Identity Verification ] [ Address ] [ Stay Details ] [ Rent Details ]|
| [ Payment History ] [ Documents ]                                                               |
|-------------------------------------------------------------------------------------------------|
| Personal Information                                                                            |
|                                                                                                 |
| Member ID                 Full Name                    Mobile Number                            |
| M01                       Rahul Patel                  9876543210                               |
| Alternate Mobile Number   Email                        Occupation                               |
| 9123456780                rahul@email.com              Student                                  |
| Date of Birth             Gender                       Company / College Name                   |
| 15/08/2000                Male                         ABC University                           |
|                                                                                                 |
| Emergency Contact                                                                               |
| Contact Person Name       Relationship                                                          |
| Ajay Patel                Father                                                                |
|                                                                                                 |
|-------------------------------------------------------------------------------------------------|
|                            [ Back ] [ Edit ]                                                    |
+-------------------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table
| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **All Personal Information Fields** | Label | N/A | Read-only mirror of Add Member fields. | `Rahul Patel` | Disables all input controls. |
| **All Identity Verification Fields**| Label / Link | N/A | Read-only. Uploads render as downloadable links. | `aadhaar.pdf` | - |
| **All Address Fields** | Label | N/A | Read-only. | `Bengaluru` | - |
| **All Stay Details Fields** | Label | N/A | Read-only. | `Room 102` | - |
| **All Rent Details Fields** | Label | N/A | Read-only. Formatted as currency/dates. | `₹6,000` | - |
| **All Member Status Fields** | Label | N/A | Read-only. | `Active` | - |
| **Payment History (Tab Content)** | Data Table | N/A | Read-only list of past transactions. | N/A | Simplified view of SCREEN 6. |
| **Documents (Tab Content)** | List / Links | N/A | Aggregated list of all uploaded files. | N/A | Downloadable artifacts. |
| **Back** | Button | N/A | Returns to Member List. | N/A | - |
| **Edit** | Button | N/A | Redirects to Edit Member screen. | N/A | Hidden if member is Inactive. |

### 4. Validations
*   **Strict Read-Only Enforcement:** No data field on this screen can be modified. All UI inputs (text boxes, dropdowns, file pickers) must be replaced with static text labels or disabled input elements.
*   **URL Parameter Integrity:** The system must validate the Member ID passed in the URL. If the ID is invalid or the member does not exist, display an appropriate error (e.g., "Member not found") and redirect to the Member List.
*   **File Access Authorization:** Secure the document download links generated in the 'Identity Verification' and 'Documents' tabs to ensure they cannot be accessed without an active administrator session.

### Payment History Tab

#### 1. Overview
The Payment History tab provides a dedicated, read-only view of all monthly rent transactions for the selected member directly within their profile. This unified view enables the administrator to quickly review the member's complete payment history, track due dates, verify payment statuses, and inspect uploaded proofs without navigating away from the View Member page, streamlining the auditing process.

#### 2. Screen Preview
```text
+------------------------------------------------------------------------------------------------------------------------------------------------+
| View Member                                                                                                                                    |
|------------------------------------------------------------------------------------------------------------------------------------------------|
| Personal Information | Identity | Address | Stay Details | Rent Details | Payment History | Documents                                         |
|------------------------------------------------------------------------------------------------------------------------------------------------|
| Search : [_________________________]                                                                                                           |
|                                                                                                                                                |
| Month | Rent Amount | Due Date | Payment Date | Payment Status | Transaction ID | Screenshot | Verified By | Verified On | Actions         |
|-------|-------------|----------|--------------|----------------|----------------|------------|-------------|-------------|-----------------|
| Jul   | ₹6,000      | 05-Jul   | 04-Jul       | Paid           | TXN12345       | View       | Admin       | 04-Jul      | View Screenshot |
| Jun   | ₹6,000      | 05-Jun   | 06-Jun       | Paid           | TXN12312       | View       | Admin       | 06-Jun      | View Screenshot |
| May   | ₹6,000      | 05-May   | --           | Pending        | --             | --         | --          | --          | --              |
|                                                                                                                                                |
| Showing 1-10 of XX Records                                                                      Previous | Next                                |
+------------------------------------------------------------------------------------------------------------------------------------------------+
```

#### 3. Screen Fields Table
| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Search** | Text Input | No | Accepts alphanumeric characters. | `TXN123` | Filters by Month, Transaction ID, or Status. |
| **Month** | Data Column | N/A | Displays the billing month. | `Jul` | - |
| **Rent Amount** | Data Column | N/A | Formatted as currency. | `₹6,000` | - |
| **Due Date** | Data Column | N/A | Formatted as Date. | `05-Jul` | - |
| **Payment Date** | Data Column | N/A | Formatted as Date. Blank if unpaid. | `04-Jul` | - |
| **Payment Status** | Data Column | N/A | Pending, In Review, Paid, Rejected. | `Paid` | - |
| **Transaction ID** | Data Column | N/A | Alphanumeric. Blank if not provided. | `TXN12345` | - |
| **Payment Screenshot**| Link | N/A | Displays 'View' or '--'. | `View` | Indicates if an image is attached. |
| **Verified By** | Data Column | N/A | Admin name who approved it. | `Admin` | - |
| **Verified On** | Data Column | N/A | Formatted as Date. | `04-Jul` | - |
| **Remarks** | Data Column | N/A | Alphanumeric text. | `Late payment` | - |
| **View Screenshot Action**| Button | N/A | Opens a modal to display the image. | `View Screenshot` | Disabled/Hidden if no screenshot exists. |
| **Pagination (Prev/Next)**| Buttons | N/A | Controls page navigation. | N/A | Disabled on boundaries. |

#### 4. Validations
*   **Member Context Enforcement:** Display only payments belonging exclusively to the currently selected member.
*   **Chronological Order:** Payment history should default to descending order (latest first).
*   **Empty State Handling:** Handle empty payment history gracefully by displaying a user-friendly "No payment history found" message.
*   **Status Display Rules:**
    *   Display "Pending" for unpaid rent.
    *   Display "In Review" for submitted payments awaiting verification.
    *   Display "Rejected" for rejected payments.
    *   Display "Paid" for verified payments.
*   **Screenshot Availability:** Screenshot preview action must only be available when a screenshot file actively exists in the system.
*   **Transaction ID Display:** Transaction ID should be displayed only if available and submitted by the member; otherwise, render a placeholder (e.g., `--`).
*   **Read-Only Data:** All data rendered within this tab must be strictly read-only to prevent unauthorized inline edits.
*   **Search Interoperability:** Search functionality should instantly filter the payment history specifically by Month, Transaction ID, or Payment Status.
*   **Pagination Functionality:** Pagination should work correctly and reset appropriately when a new search query is executed.
*   **Graceful Degradation:** Handle missing, corrupted, or deleted payment records gracefully without crashing the UI.

---

## SCREEN 4 : EDIT MEMBER

### 1. Overview
The **Edit Member** screen empowers authorized administrators to modify the existing profile and financial configuration of a tenant. Designed to mirror the layout of the 'Add Member' screen, this interface pre-populates all fields with the member's current data. It is crucial for executing room transfers, adjusting rent amounts, updating contact information, or processing a member's transition to a 'Notice Period'.

### 2. Screen Preview
```text
+-------------------------------------------------------------------------------------------------+
| Edit Member : Rahul Patel (M01)                                                                 |
|-------------------------------------------------------------------------------------------------|
| Personal Information                                                                            |
| Member ID *                  Full Name *                  Mobile Number *                       |
| [ M01___________________]    [ Rahul Patel__________]     [ 9876543210___________]              |
| Alternate Mobile Number      Email                        Occupation *                          |
| [ 9123456780____________]    [ rahul@email.com______]     [ Student      v ]                    |
| Date of Birth *              Gender *                     Company / College Name *              |
| [ 15/08/2000 ]               [ Male         v ]           [ ABC University_______]              |
|                                                                                                 |
| Identity Verification                                                                           |
| Aadhaar Card *                                                                                  |
| [ 123456789012 ] [ Current: aadhaar.pdf ] [ Replace File ]                                      |
| PAN                                                                                             |
| [ ABCDE1234F ] [ Current: pan.jpg ] [ Replace File ]                                            |
| Driving Licence                                                                                 |
| [ DL142011001 ] [ Current: dl.png ] [ Replace File ]                                            |
|                                                                                                 |
| Emergency Contact                                                                               |
| Contact Person Name *        Relationship *                                                     |
| [ Ajay Patel____________]    [ Father       v ]                                                 |
|                                                                                                 |
| Address                                                                                         |
| Address Line 1 *             Address Line 2               City *                                |
| [ Plot 42, Sector 1_____]    [ Opposite Park________]     [ Bengaluru____________]              |
| State *                      Pincode *                    Country                               |
| [ Karnataka_____________]    [ 560034_______________]     [ India________________]              |
|                                                                                                 |
| Stay Details                                                                                    |
| PG Name *                    Room Number *                Bed *                                 |
| [ Sunshine PG  v ]           [ 102          v ]           [ A            v ]                    |
|                                                                                                 |
| Rent Details                                                                                    |
| Monthly Rent *               Security Deposit *           Maintenance Charge *                  |
| [ 6000__________________]    [ 12000________________]     [ 500__________________]              |
| Rent Due Date *              Notice Period *                                                    |
| [ 5            v ]           [ 30 Days      v ]                                                 |
|                                                                                                 |
| Member Status                                                                                   |
| Status *                     Reason                                                             |
| [ Active       v ]           [______________________]                                           |
|                                                                                                 |
|                            [ Cancel ] [ Update ]                                                |
+-------------------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table
| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **All Fields matching Add Member** | Standard Inputs | Same as Add | Pre-filled with database records. | `Rahul Patel` | Adheres to all 'Add Member' rules. |
| **Document Replace File** | File Input | No | Allowed MIME types, Max size. | `new_aadhaar.pdf` | Overwrites existing document if provided. |
| **Update** | Button | N/A | Submits the modified data payload. | N/A | Validates changes before API call. |
| **Cancel** | Button | N/A | Discards unsaved modifications. | N/A | Returns to View Member or List screen. |

### 4. Validations
*   **Accurate Data Pre-population:** Upon initialization, the system must accurately load and bind all existing tenant data, including dropdown selections and conditionally visible fields.
*   **Duplicate Checks (Excluding Self):** When validating unique fields like `Mobile Number` or `Aadhaar Card`, the backend must exclude the current member's ID from the uniqueness query to prevent false-positive duplicate errors.
*   **Stay Details Conflict Prevention:** If the `PG Name`, `Room Number`, or `Bed` is modified (i.e., a room transfer), the system must validate that the new destination bed is genuinely unoccupied and not double-booked.
*   **Financial Update Handling:** Altering the `Monthly Rent` or `Rent Due Date` mid-cycle requires backend logic to determine if the changes apply immediately or to the subsequent billing cycle.
*   **Retain Standard Validations:** Enforce all frontend format validations identical to the Add Member screen (e.g., 10-digit mobile, age limit, required fields).

---

## SCREEN 5 : DEACTIVATE MEMBER

### 1. Overview
The **Deactivate Member** screen functions as a critical safety checkpoint before terminating a tenant's active lifecycle. Deactivation formally changes the member's status to 'Inactive', officially releasing their allocated bed back into the available inventory. To preserve systemic data integrity, financial history, and compliance, the member's profile is never permanently deleted from the database.

### 2. Screen Preview
```text
+-----------------------------------------------------------------------+
|                       Deactivate Member                               |
|-----------------------------------------------------------------------|
| Are you sure you want to deactivate Rahul Patel (M01)?                |
|                                                                       |
| This action will:                                                     |
| - Change the member's status to 'Inactive'.                           |
| - Mark their allocated Bed (Room 102, Bed A) as Vacant.               |
| - Stop future automatic rent generation.                              |
|                                                                       |
| Note: Historical data and payment records will be preserved safely.   |
|                                                                       |
|                     [ Cancel ] [ Deactivate ]                         |
+-----------------------------------------------------------------------+
```

### 3. Screen Fields Table
| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Confirmation Message** | Label | N/A | Dynamically includes Member Name and ID. | `...deactivate Rahul Patel (M01)?`| Clearly explains systemic consequences. |
| **Cancel** | Button | N/A | Closes the modal. | N/A | Prevents accidental execution. |
| **Deactivate** | Button | N/A | Initiates the deactivation API call. | N/A | Disabled after click to prevent duplicate processing. |

### 4. Validations
*   **Explicit Action Requirement:** Deactivation must require a deliberate button click on the modal; it cannot be triggered via accidental keystrokes.
*   **Already Inactive Check:** The system must gracefully reject the request if the member has already been marked as 'Inactive' (e.g., concurrent admin action), returning a descriptive error.
*   **Outstanding Dues Warning:** (Optional Frontend Check) If the member has an active `Outstanding Amount > 0`, the system may display an inline warning: "This member still has pending dues." but still allow deactivation for operational flexibility.
*   **Success Notification:** Upon successful execution, the modal must close, the backend must update the bed inventory, and the UI should display a success toast: "Member deactivated successfully."

---

## SCREEN 6 : PAYMENT HISTORY

### 1. Overview
The **Payment History** screen provides an exhaustive, chronological ledger of every monthly rent transaction associated with a specific member. It serves as the definitive financial log, replacing the need for a separate rent module. Administrators utilize this screen to audit past payments, verify historical transaction IDs, download payment proofs, and reconcile a member's complete financial journey from onboarding to the present day.

### 2. Screen Preview
```text
+------------------------------------------------------------------------------------------------------------------------------------------+
| Payment History : Rahul Patel (M01)                                                                                                      |
|------------------------------------------------------------------------------------------------------------------------------------------|
| Search [_________________________]                                                                                                       |
|------------------------------------------------------------------------------------------------------------------------------------------|
| Month  | Rent Amount | Due Date   | Payment Date | Payment Status | Transaction ID | Payment Screenshot | Remarks | Actions            |
|--------|-------------|------------|--------------|----------------|----------------|--------------------|---------|--------------------|
| Aug-26 | ₹6,000      | 05-Aug-26  | -            | In Review      | TXN987654321   | [View Image]       | -       | View | Download  |
| Jul-26 | ₹6,000      | 05-Jul-26  | 04-Jul-26    | Paid           | TXN123456789   | [View Image]       | On time | View | Download  |
| Jun-26 | ₹6,000      | 05-Jun-26  | 06-Jun-26    | Paid           | TXN445566778   | [View Image]       | Late    | View | Download  |
|------------------------------------------------------------------------------------------------------------------------------------------|
| Showing 1-3 of 12 Records                                                                              Previous | Next                   |
+------------------------------------------------------------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table
| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Search** | Text Input | No | Accepts alphanumeric input. | `TXN123` | Searches specifically by Month or Transaction ID. |
| **Month** | Data Column | N/A | Sortable. Displays billing cycle. | `Aug-26` | Identifies the rent period. |
| **Rent Amount** | Data Column | N/A | Sortable. Currency format. | `₹6,000` | Processed transaction amount. |
| **Due Date** | Data Column | N/A | Sortable. Date format. | `05-Aug-26` | Original target date. |
| **Payment Date** | Data Column | N/A | Sortable. Date format. | `04-Jul-26` | Actual cleared date. |
| **Payment Status** | Data Column | N/A | Sortable. (Pending, In Review, Paid, Rejected). | `Paid` | Terminal state of the cycle. |
| **Transaction ID** | Data Column | N/A | Sortable. | `TXN123456789` | From bank or UPI app. |
| **Payment Screenshot**| Link/Button| N/A | Opens preview modal of uploaded image. | `[View Image]` | Proof of transaction. |
| **Remarks** | Data Column | N/A | Text content. | `Late payment` | Admin notes. |
| **View** | Action Link | N/A | Opens a detailed summary modal for the cycle. | N/A | - |
| **Download** | Action Link | N/A | Triggers download of the Payment Screenshot. | N/A | Hidden if no screenshot exists. |

### 4. Validations
*   **Data Integrity Check:** Ensure all financial data accurately mirrors the backend ledger without permitting any inline editing.
*   **Screenshot Availability:** The `[View Image]` link and `Download` action must be dynamically disabled or hidden if no file was uploaded for a specific billing cycle (e.g., manual cash payments).
*   **Chronological Default:** The table must default to sorting by `Month` or `Due Date` in descending order, displaying the most recent transactions first.
*   **Search Constraints:** The local search should quickly filter records on the client side without executing heavy database queries, given the typically limited size of a single member's history.

---

## SCREEN 7 : PAYMENT VERIFICATION

### 1. Overview
The **Payment Verification** screen empowers the administrator to manually audit and reconcile rent payments submitted by members. Integrated directly into the unified Member Management flow, this screen isolates transactions explicitly flagged as **'In Review'**. The admin can meticulously inspect the member-uploaded UPI screenshot, cross-reference the Transaction ID against bank statements, and decisively mark the payment as 'Paid' or 'Rejected', thereby maintaining strict financial accuracy.

### 2. Screen Preview
```text
+-------------------------------------------------------------------------------------------------+
| Payment Verification                                                                            |
|-------------------------------------------------------------------------------------------------|
| Member Details                                                                                  |
| Member Name  : Rahul Patel                    Monthly Rent : ₹6,000                             |
| PG Name      : Sunshine PG                    Due Date     : 05-Aug-2026                        |
| Room Number  : 102                            Payment Date : 04-Aug-2026                        |
|                                                                                                 |
|-------------------------------------------------------------------------------------------------|
| Transaction Details                                                                             |
| Transaction ID (if provided) : TXN987654321                                                     |
| Payment Status               : In Review                                                        |
|                                                                                                 |
| Uploaded Screenshot:                                                                            |
| +---------------------------------------------------------+                                     |
| |                                                         |                                     |
| |             [ Image Preview Rendering ]                 |                                     |
| |                                                         |                                     |
| +---------------------------------------------------------+                                     |
|                                                                                                 |
| Admin Remarks (Optional):                                                                       |
| [_______________________________________________________]                                       |
|                                                                                                 |
|-------------------------------------------------------------------------------------------------|
|                      [ Back ]    [ Reject Payment ]    [ Approve Payment ]                      |
+-------------------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table
| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Member Name** | Label | N/A | Read-only. | `Rahul Patel` | - |
| **PG Name** | Label | N/A | Read-only. | `Sunshine PG` | - |
| **Room Number** | Label | N/A | Read-only. | `102` | - |
| **Monthly Rent** | Label | N/A | Read-only. Currency format. | `₹6,000` | Expected value to match against screenshot. |
| **Due Date** | Label | N/A | Read-only. Date format. | `05-Aug-2026` | - |
| **Payment Date** | Label | N/A | Read-only. Date format. | `04-Aug-2026` | Date member submitted the form. |
| **Transaction ID** | Label | N/A | Read-only. | `TXN987654321` | - |
| **Payment Status** | Label | N/A | Read-only. Must be 'In Review'. | `In Review` | - |
| **Uploaded Screenshot**| Image Render| N/A | Must render the actual uploaded image. | N/A | Admin inspects this visually. |
| **Admin Remarks** | Text Input | No | Alphanumeric. Max 255 chars. | `Verified with bank.` | Saved to the transaction log. |
| **Back** | Button | N/A | Navigates back to previous screen. | N/A | Discards unsaved remarks. |
| **Reject Payment** | Button | N/A | Triggers API to mark status as 'Rejected'. | N/A | Prompt for mandatory reason if rejected. |
| **Approve Payment**| Button | N/A | Triggers API to mark status as 'Paid'. | N/A | Resolves the billing cycle. |

### 4. Validations
*   **State Constraint (In Review Only):** The screen must explicitly reject rendering and redirect the admin if the targeted payment record's status is anything other than `In Review` (e.g., if it was already verified by another admin).
*   **Screenshot Existence:** The system must validate that an image asset exists. If the link is broken, display a clear "Screenshot Unavailable" placeholder.
*   **Duplicate Action Prevention:** The "Approve" and "Reject" buttons must immediately disable upon click and display a loading state to prevent double execution of financial state changes.
*   **Rejection Requirement:** If the admin clicks "Reject Payment", the system should ideally mandate the `Admin Remarks` field to ensure the member receives an explanation (e.g., "Blurry screenshot").
*   **Post-Action Redirection:** Upon successful approval or rejection, display a success toast ("Payment Verified Successfully") and automatically redirect the admin back to the Member List or the previous queue.

---

# FILTER CONFIGURATION

Update the unified Member Management filters to support robust querying across both member demographics and financial rent parameters.

| Filter | Type | Values | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Member Name** | Text Input | User Input (Alphanumeric) | Empty | Filters records by tenant's full or partial name. |
| **Mobile Number** | Text Input | User Input (Numeric) | Empty | Filters records by contact number. |
| **PG Name** | Dropdown | All, [List of Active PGs] | All | Isolates tenants residing in a specific property. |
| **Room Number** | Text Input | User Input (Alphanumeric) | Empty | Narrows down to members in a specific room. |
| **Bed Number** | Text Input | User Input (Alphanumeric) | Empty | Locates the specific bed allocation. |
| **Occupation** | Dropdown | All, Student, Employee, Business, Freelancer, Other | All | Filters by tenant professional status. |
| **Gender** | Dropdown | All, Male, Female, Other | All | Filters tenants by demographic gender. |
| **Member Status** | Dropdown | All, Active, Notice Period, Inactive| Active | Filters based on current residency lifecycle phase. |
| **Rent Status** | Dropdown | All, Paid, Pending, Overdue| All | Filters by the operational state of rent collection. |
| **Payment Status** | Dropdown | All, Pending, In Review, Paid, Rejected | All | Filters specifically by the transaction verification state. |
| **Due Date** | Date Picker| Custom Date Range | Empty | Isolates expected payment timelines. |
| **Payment Date** | Date Picker| Custom Date Range | Empty | Filters by actual historical transaction dates. |
| **Joining Date** | Date Picker| Custom Date Range | Empty | Identifies onboarding periods. |
| **Monthly Rent Range** | Number Input| Custom Range (Min/Max) | Empty | Filters members by their base rent threshold. |
| **Outstanding Amount Range**| Number Input| Custom Range (Min/Max) | Empty | Quickly identifies major default accounts. |
| **Notice Period** | Dropdown | All, [Configured Notice Days]| All | Filters tenants bound by specific notice parameters. |
| **City** | Dropdown | All, [List of Cities] | All | Filters geographically based on member origin or property. |

**Filter Behaviour Principles:**
*   **Simultaneous Application:** Multiple filters can be applied simultaneously to narrow down highly specific records (e.g., searching for Active members with Overdue rent in PG1).
*   **Search Interoperability:** Search works together with filters seamlessly; keyword inputs refine the currently filtered dataset further.
*   **State Persistence:** Filters persist until the 'Reset' button is clicked, maintaining the operational context without losing criteria during page refreshes or navigation.
*   **Dynamic Pagination:** Pagination updates instantly after filtering, ensuring accurate page counts and boundary control.
*   **Sort Compatibility:** Sorting works gracefully with filters, correctly ordering the active subset of data generated by the filter conditions.
*   **Consistent Export:** Export operations respects the applied filters and search criteria, outputting exactly what is displayed on the screen.
