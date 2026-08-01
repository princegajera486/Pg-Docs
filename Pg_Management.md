# Authentication Module

## 1. Overview

**Purpose of this module:**  
The Authentication module is designed to provide a secure and reliable gateway for the PG Management System. It ensures that only authorized personnel can access sensitive property, tenant, and financial data.

**Business objective:**  
To restrict unauthorized access to the application, safeguard confidential business information, and maintain a seamless, centralized entry point for system administration.

**User who can access this module:**  
Only one authorized user (Owner/Admin) is permitted to access this system. Self-registration is strictly disabled to prevent unauthorized account creation.

**Expected system behaviour:**  
The system will prompt the user to provide their valid credentials (Email Address or Mobile Number, along with a Password). Upon successful validation, the system will establish a secure session and redirect the user to the central dashboard. In the event of a forgotten password, the system provides a secure recovery mechanism.

---

## 2. Screen Preview

```text
+--------------------------------------+
|              Login                   |
|--------------------------------------|
| Email / Mobile Number                |
| [________________________]           |
|                                      |
| Password                             |
| [________________________]           |
|                                      |
| ( Login )                            |
|                                      |
| Forgot Password?                     |
+--------------------------------------+
```

---

## 3. Screen Fields

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Email / Mobile Number** | Alphanumeric | Yes | Must be a valid Email format OR a valid 10-digit Mobile Number. Max length: 50. | `admin@pgsystem.com` or `9876543210` | Accepts either email or mobile to enhance usability. Leading/trailing spaces should be trimmed. |
| **Password** | Password | Yes | Minimum 8 characters. Must contain at least one uppercase letter, one number, and one special character. | `**********` | Input must be masked. Trailing/leading spaces should be trimmed. |
| **Login** | Button | N/A | Trigger authentication sequence upon click. | N/A | Button should be disabled during active API calls to prevent double submission. |
| **Forgot Password?** | Link | N/A | Redirects user to the password recovery screen. | N/A | Placed below the login button for easy discovery. |

---

## 4. Validations

**Frontend Validations:**
*   **Empty Fields:** If either the identifier or password field is left empty upon submission, display an inline error: "Please enter your Email/Mobile Number" or "Password is required".
*   **Input Trimming:** Automatically strip leading and trailing whitespace from the Email/Mobile field before submission.
*   **Format Check:** Determine if the input is an email or mobile number using Regex. If it contains an `@`, validate against standard email format. If numerical, ensure it matches the 10-digit mobile number format.
*   **Password Rules:** Ensure the password field is not empty before transmitting (complex rules are deferred to the backend to prevent leaking security policies).

**Backend Validations:**
*   **Invalid Email / Mobile Number:** If the identifier does not exist in the database, return a generic error message to prevent enumeration: "Invalid credentials."
*   **Incorrect Credentials:** Validate password against stored hash. Reject unauthorized access and log the failed attempt. Return generic "Invalid credentials."
*   **SQL Injection Prevention:** Use parameterized queries or ORM (Object-Relational Mapping) for all database interactions.
*   **XSS Prevention:** Sanitize all inputs to strip any potentially malicious scripts.
*   **Session Timeout:** Ensure active session expires after 30 minutes of inactivity.
*   **Maximum Login Attempts:** Restrict login attempts to 5 consecutive failures. 
*   **Account Lock Policy:** Lock the account for 15 minutes after 5 consecutive failed attempts and send a warning notification to the registered email.
*   **Rate Limiting:** Implement rate limiting (e.g., max 10 requests per minute per IP) on the authentication endpoint to thwart brute-force attacks.
*   **Case Sensitivity:** The Email address should be treated as case-insensitive, whereas the Password MUST be strictly case-sensitive.
*   **HTTPS Requirement:** Reject any authentication requests that do not originate over a secure HTTPS connection.

---

## 5. Business Rules

*   **Authorized Access Only:** Only predefined, authorized users (Owner/Admin) can log in.
*   **No Self-Registration:** The system shall not expose any public registration endpoints. Account provisioning is handled manually via database seeding or an internal tool.
*   **Protected Routes:** A user cannot access the dashboard or any internal module without presenting a valid, active session/token.
*   **Session Expiry:** A user session should automatically expire after a predefined period of inactivity (e.g., 30 minutes) to maintain security.
*   **Secure Storage:** Passwords should never be stored in plain text. They must be salted and hashed using strong cryptographic algorithms (e.g., bcrypt or Argon2).
*   **Secure Tokens:** Authentication tokens (such as JWTs) must be securely generated, signed, and transmitted.
*   **Active Status:** Only accounts marked with an 'Active' status in the database can successfully log in. Disabled or suspended accounts must be rejected with a "Your account has been deactivated" message.

---

## 6. Dependencies

*   **User Database:** Required to store and retrieve the admin credentials, account status, and login history.
*   **Authentication Service:** The core backend logic handling credential verification and token/session generation.
*   **Session Management:** Required to maintain user state across requests, utilizing either secure HTTP-only cookies or server-side sessions.
*   **Password Hashing Service:** Required for securely hashing user input to compare against the stored database hash (e.g., bcrypt library).
*   **Email Service:** Required to dispatch secure OTPs or password reset links for the "Forgot Password" flow.
*   **SMS Service:** Intended as a future dependency for mobile-based OTP recovery (if implemented later).
*   **Logging Service:** Required to track both successful logins and failed attempts for audit trails and anomaly detection.

---

# Dashboard Module

## 1. OVERVIEW
The **Dashboard** serves as the central landing page for the PG owner/administrator immediately following a successful login. It acts as the executive control center, seamlessly consolidating real-time data from the **PG Management**, **Member Management**, and **Rent Management** modules. Its primary purpose is to provide the owner with a complete, at-a-glance overview of the business health. By leveraging real-time KPIs, visual charts, actionable tables, and automated alerts, the dashboard empowers the administrator to monitor occupancy, track member statistics, assess rent collection performance, manage outstanding dues, and respond promptly to critical operational events.

## 2. DASHBOARD PREVIEW

```text
+-------------------------------------------------------------------------------------------------------------------------+
| Dashboard                                                                                         Welcome, Admin        |
|-------------------------------------------------------------------------------------------------------------------------|
| Total PGs | Total Rooms | Occupied Beds | Occupancy Rate | Active Members | Rent Collected | Pending Rent | Revenue     |
|    5      |     120     |      200      |      85%       |      200       |    ₹12,00,000  |   ₹50,000    | ₹12,50,000  |
|-------------------------------------------------------------------------------------------------------------------------|
| Occupancy Overview (Doughnut)       | Monthly Rent Collection Trend (Line Chart)                                     |
|-------------------------------------|--------------------------------------------------------------------------------|
| [ Chart Area ]                      | [ Chart Area ]                                                                 |
|-------------------------------------------------------------------------------------------------------------------------|
| Member Status Distribution (Pie)    | Revenue by PG (Horizontal Bar)                                                 |
|-------------------------------------|--------------------------------------------------------------------------------|
| [ Chart Area ]                      | [ Chart Area ]                                                                 |
|-------------------------------------------------------------------------------------------------------------------------|
| Upcoming Due Table                  | Recent Payments Table                                                           |
|-------------------------------------------------------------------------------------------------------------------------|
| Alerts Box (e.g., 5 Overdue Rents, 2 Low Occupancy PGs)                                                                 |
+-------------------------------------------------------------------------------------------------------------------------+
```

## 3. KPI CARDS

| KPI | Description | Formula | Data Source |
| :--- | :--- | :--- | :--- |
| **Total PGs** | Total number of properties registered. | Count of all PGs. | PG Management |
| **Active PGs** | Properties currently operational. | Count of PGs where Status = Active. | PG Management |
| **Inactive PGs** | Properties temporarily or permanently closed. | Count of PGs where Status = Inactive. | PG Management |
| **Total Rooms** | Total room capacity across all PGs. | Sum of Rooms across all Active PGs. | PG Management |
| **Total Beds** | Total bed capacity across all PGs. | Sum of all configured beds across Active PGs. | PG Management |
| **Occupied Beds** | Total beds currently allocated to Active Members. | Count of beds assigned to Members where Status = Active or Notice Period. | Member Management |
| **Vacant Beds** | Total beds available for new tenants. | Total Beds - Occupied Beds. | PG & Member Management |
| **Occupancy Rate (%)** | Percentage of beds currently occupied. | (Occupied Beds / Total Beds) * 100 | PG & Member Management |
| **Total Members** | Total number of registered members (all time). | Count of all Members. | Member Management |
| **Active Members** | Members currently residing in a PG. | Count of Members where Status = Active. | Member Management |
| **Members in Notice Period** | Members who are preparing to vacate. | Count of Members where Status = Notice Period. | Member Management |
| **Inactive Members** | Members who have permanently vacated. | Count of Members where Status = Inactive. | Member Management |
| **New Members This Month**| Members onboarded in the current month. | Count of Members created in the current calendar month. | Member Management |
| **Total Monthly Rent** | Total rent billed for the current month. | Sum of Monthly Rent for all Active/Notice members for current month. | Rent Management |
| **Rent Collected This Month** | Rent successfully received for the current month. | Sum of Paid rent amounts for the current month. | Rent Management |
| **Pending Rent** | Total rent amount due but not yet past the due date. | Sum of outstanding rent where Due Date >= Today. | Rent Management |
| **Overdue Rent** | Total rent amount past the due date. | Sum of outstanding rent where Due Date < Today. | Rent Management |
| **Collection Percentage** | Efficiency of rent collection for the month. | (Rent Collected / Total Monthly Rent) * 100 | Rent Management |
| **Expected Monthly Revenue** | Total projected revenue for the month. | Sum of (Monthly Rent + Maintenance Charge) for all active members. | Rent Management |

## 4. CHARTS

| Chart Name | Chart Type | Description | Data Source |
| :--- | :--- | :--- | :--- |
| **Occupancy Overview** | Doughnut / Pie | Visualizes the ratio of Occupied Beds vs Vacant Beds across the business. Identifies capacity utilization. | PG & Member Management |
| **Member Status Distribution** | Pie Chart | Visualizes the proportion of Active, Notice Period, and Inactive members. Helps forecast upcoming vacancies. | Member Management |
| **Monthly Rent Collection** | Line Chart | Tracks the trend of Collected Rent month-wise over the last 12 months. Identifies seasonal revenue variations. | Rent Management |
| **Rent Status** | Bar Chart | Compares the total volume of Paid, Pending, and Overdue rent for the current billing cycle. | Rent Management |
| **Revenue by PG** | Horizontal Bar Chart | Ranks PGs based on revenue generated. Helps identify top-performing properties. | PG & Rent Management |
| **Occupancy by PG** | Horizontal Bar Chart | Compares the number of Occupied Beds in every PG. Highlights properties needing marketing efforts. | PG & Member Management |
| **Upcoming Due Trend** | Column Chart | Forecasts upcoming rent dues grouped by time intervals (Next 7 Days, 15 Days, 30 Days) to aid cash flow projection. | Rent Management |

## 5. TABLES

| Table Name | Description | Columns |
| :--- | :--- | :--- |
| **Upcoming Rent Due** | Lists members whose rent is approaching the due date. Enables proactive follow-ups before rent becomes overdue. | Member, PG, Room, Rent, Due Date, Days Remaining |
| **Overdue Rent** | Highlights critical payment defaults requiring immediate administrative action or penalty application. | Member, PG, Room, Outstanding Amount, Due Date, Days Overdue |
| **Recent Payments** | A quick ledger of the most recently logged or reconciled payment transactions for rapid cross-verification. | Member, PG, Amount, Payment Date, Payment Mode |
| **Recently Added Members** | Displays the latest tenant onboardings to track recent sales and occupancy growth. | Member, PG, Room, Joining Date |

## 6. ALERTS

| Alert | Priority | Trigger Condition | Action Required |
| :--- | :--- | :--- | :--- |
| **Rent Due Today** | High | Member's Rent Due Date equals current system date and Status != Paid. | Send reminder to member or follow up for collection. |
| **Rent Overdue** | Critical | Member's Rent Due Date has passed and Status != Paid. | Initiate overdue follow-up or apply late penalty. |
| **Member in Notice Period** | Medium | Member Status is changed to 'Notice Period'. | Prepare for checkout process and market the upcoming vacant bed. |
| **Low Occupancy PG** | High | A specific PG's Occupancy Rate drops below a defined threshold (e.g., 50%). | Focus marketing and sales efforts on this property. |
| **Fully Occupied PG** | Low | A specific PG's Occupancy Rate reaches 100%. | None immediate; consider expansion or optimizing rent yields. |
| **Inactive PG** | Medium | A PG's Status is manually changed to 'Inactive'. | Verify that no active members are mistakenly mapped to it. |
| **Recently Added Member** | Low | A new member is registered in the system within the last 24 hours. | Ensure welcome kit or onboarding formalities are complete. |
| **Upcoming Vacant Bed** | Medium | Calculated when a member in a specific bed enters their Notice Period. | Update external listings to reflect future availability. |
| **Missing Rent Payment** | High | Rent record not generated or missing for an active member for the current billing cycle. | Systematically generate or manually investigate missing invoice. |
| **Failed Bank Statement Processing** | Critical | An uploaded Bank Statement PDF fails to parse or reconcile due to incorrect password or format. | Re-upload correct document or verify file integrity. |
| **Duplicate Member Detected** | High | System detects another active member attempting entry with identical Aadhaar or Mobile. | Merge records or reject duplicate registration. |
| **Room Capacity Exceeded** | Critical | Number of Active Members mapped to a Room exceeds its defined 'Room Sharing' limit. | Reassign tenants to correct rooms immediately. |

---

# PG Management Module

## SCREEN 2.1 : LIST PG

### 1. Overview
The **List PG** screen serves as the central directory for managing all properties (PGs and Apartments) registered within the system. Its primary purpose is to provide authorized users with a comprehensive, searchable, and sortable view of all properties. From this interface, users can quickly find specific properties and trigger administrative actions such as viewing detailed property information, editing configurations, or deactivating a property.

### 2. Screen Preview

```text
+------------------------------------------------------------------------------------+
| PG Management                                                           Add New PG |
|------------------------------------------------------------------------------------|
| Search : [____________________]                                                    |
|                                                                                    |
| Code | PG Name | Type | Gender | Contact | Mobile | Status | Actions              |
|------|---------|------|---------|---------|--------|--------|----------------------|
| P001 | Sai PG  | PG   | Male    | Ravi    | 98765..| Active | View Edit Deactivate |
| P002 | Om Apt  | Apt  | Co-Liv  | Amit    | 91234..| Inact..| View Edit Deactivate |
|                                                                                    |
| Showing 1-10 of 50 Records                                       Previous Next     |
+------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Add New PG** | Button | N/A | Click redirects user to the "Add PG" screen. | N/A | Placed prominently at the top right. |
| **Search** | Text Input | No | Accepts alphanumeric characters. Supports partial matching. | `Sai` or `P001` | Searches across Code, Name, Contact, and Mobile columns. |
| **Code** | Data Column | N/A | Sortable column. Displays alphanumeric ID. | `P001` | Unique identifier for the property. |
| **PG Name** | Data Column | N/A | Sortable column. | `Sai PG` | Name of the property. |
| **Type** | Data Column | N/A | Sortable column. Displays PG or Apartment. | `PG` | - |
| **Gender** | Data Column | N/A | Sortable column. Displays Male, Female, or Co-Living. | `Male` | - |
| **Contact** | Data Column | N/A | Sortable column. | `Ravi` | Primary contact person's name. |
| **Mobile** | Data Column | N/A | Sortable column. Must display 10 digits. | `9876543210` | Formatted for readability. |
| **Status** | Data Column | N/A | Sortable column. Displays Active or Inactive. | `Active` | Visual indicator (e.g., green for Active, red for Inactive). |
| **View** | Action Button | N/A | Click redirects to "View PG" screen for the specific record. | N/A | Available for both Active and Inactive records. |
| **Edit** | Action Button | N/A | Click redirects to "Edit PG" screen. Hidden or disabled for Inactive records based on business rules. | N/A | - |
| **Deactivate** | Action Button | N/A | Click opens "Deactivate PG" confirmation popup. | N/A | Hidden or disabled if record is already Inactive. |
| **Records Per Page** | Dropdown | No | Must be a valid integer (e.g., 10, 20, 50, 100). | `10` | Defaults to 10. |
| **Pagination (Prev/Next)** | Buttons | N/A | Disabled if on the first/last page respectively. | N/A | Allows navigating through large datasets. |
| **Total Records** | Label | N/A | Displays accurate count based on search filters. | `Showing 1-10 of 50 Records` | Updates dynamically upon searching. |

### 4. Validations

*   **Search Functionality:** 
    *   Search must support partial matching (e.g., typing "Sai" matches "Sai PG").
    *   Search input must automatically ignore/trim leading and trailing spaces.
*   **Pagination Handling:**
    *   Invalid page number handling: If a user manually alters the URL to a non-existent page, redirect to Page 1.
*   **Empty State:** 
    *   If no records exist (or search yields zero results), display a user-friendly empty state message: "No properties found."
*   **Action Availability:**
    *   Users cannot edit inactive records. The "Edit" button must be disabled or hidden.
    *   Confirmation is required before deactivation (handled by the Deactivate PG screen).

---

## SCREEN 2.2 : ADD PG

### 1. Overview
The **Add PG** screen provides a structured form to onboard and register a new PG or Apartment into the system. This allows the administrator to capture essential property details including basic identifiers, location parameters, configuration specifics, and available amenities. Successfully saving this form makes the property visible and operational within the system.

### 2. Screen Preview

```text
+------------------------------------------------------------------------------------+
| Add New PG                                                                         |
|------------------------------------------------------------------------------------|
| Basic Information                                                                  |
| Code / ID *             Name *                    Type *                           |
| [_____________]         [_____________]           [ Dropdown v ]                   |
| Gender Type *           Contact Person *          Mobile *                         |
| [ Dropdown v ]          [_____________]           [_____________]                  |
| Description                                                                        |
| [_______________________________________________________]                          |
|                                                                                    |
| Location                                                                           |
| Address Line 1 *        Address Line 2            Area *                           |
| [_____________]         [_____________]           [_____________]                  |
| Landmark                City *                    State *                          |
| [_____________]         [_____________]           [_____________]                  |
| Pincode *               Country *                                                  |
| [_____________]         [_____________]                                            |
|                                                                                    |
| Property Configuration                                                             |
| No. of Rooms / Flats *  Room Sharing / Flat Sharing *                              |
| [_____________]         [_____________]                                            |
| Rent *                  Property Status *                                          |
| [_____________]         [ Dropdown v ]                                             |
|                                                                                    |
| Amenities                                                                          |
| [ ] WiFi            [ ] RO Water         [ ] Washing Machine  [ ] Refrigerator     |
| [ ] TV              [ ] CCTV             [ ] Parking          [ ] Lift             |
| [ ] House Keeping   [ ] Food             [ ] Kitchen Staff                         |
|                                                                                    |
|                                                     [ Cancel ] [ Save ]            |
+------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Code / ID** | Text Input | Yes | Max 20 chars. Must be unique. Alphanumeric without spaces. | `P001` | System may auto-generate or allow manual entry. |
| **Name** | Text Input | Yes | Max 100 chars. Alphanumeric with spaces. | `Sai PG` | - |
| **Type** | Dropdown | Yes | Value must be 'PG' or 'Apartment'. | `PG` | - |
| **Gender Type** | Dropdown | Yes | Value must be 'Male', 'Female', or 'Co-Living'. | `Male` | Restricts tenant assignment later. |
| **Contact Person** | Text Input | Yes | Max 100 chars. Alphabetic characters and spaces only. | `Ravi Kumar` | - |
| **Mobile** | Text Input | Yes | Exactly 10 digits. Numeric only. | `9876543210` | Do not allow special characters or country codes. |
| **Description** | Text Area | No | Max 500 chars. | `Near main market` | - |
| **Address Line 1** | Text Input | Yes | Max 255 chars. | `Plot 42, Sector 1` | - |
| **Address Line 2** | Text Input | No | Max 255 chars. | `Opposite Park` | - |
| **Area** | Text Input | Yes | Max 100 chars. | `Koramangala` | - |
| **Landmark** | Text Input | No | Max 100 chars. | `Near Metro Station` | - |
| **City** | Text Input | Yes | Max 100 chars. | `Bengaluru` | Can be a dropdown based on State. |
| **State** | Text Input | Yes | Max 100 chars. | `Karnataka` | Can be a dropdown. |
| **Pincode** | Text Input | Yes | Exactly 6 digits. Numeric only. | `560034` | Valid Indian pincode format. |
| **Country** | Text Input | Yes | Max 50 chars. | `India` | Default to 'India'. |
| **No. of Rooms / Flats**| Number Input | Yes | Must be a positive integer greater than 0. Max 999. | `20` | - |
| **Room/Flat Sharing** | Number Input | Yes | Must be a positive integer (1 to 10). | `2` | Determines bed capacity per room. |
| **Rent** | Number Input | Yes | Must be a positive number. Max 2 decimal places. | `8500.00` | Default/base rent display. |
| **Property Status** | Dropdown | Yes | Value must be 'Active' or 'Inactive'. | `Active` | Default to 'Active'. |
| **Amenities** | Checkboxes | No | Array of boolean values. | Checked / Unchecked | Multiple selections allowed. Includes WiFi, RO Water, Washing Machine, Refrigerator, TV, CCTV, Parking, Lift, House Keeping, Food, Kitchen Staff. |
| **Save** | Button | N/A | Submits the form. Disabled during API call. | N/A | Validates all fields before submission. |
| **Cancel** | Button | N/A | Discards changes and returns to "List PG". | N/A | - |

### 4. Validations

*   **Required Fields:** All fields marked with an asterisk (*) must contain valid data. Display inline error (e.g., "This field is required") if left blank on Save.
*   **Mobile Number:** Must be strictly numeric and exactly 10 digits long.
*   **Duplicate PG Code:** On submission (or on blur), check if the `Code / ID` already exists in the database. If yes, display "PG Code already exists".
*   **Duplicate PG Name:** Display a warning or error if a property with the exact same Name exists in the same Area/City.
*   **Numeric Validations:** `No. of Rooms / Flats` and `Room Sharing / Flat Sharing` must be strictly positive integers without decimals. 
*   **Rent Validation:** Rent must be a positive number and cannot be negative.
*   **Pincode:** Must be strictly numeric and exactly 6 digits long.
*   **Character Limits:** Input length must not exceed database column limits (e.g., Name max 100 chars). Restrict typing past the limit.
*   **Dropdown Validations:** Submitted values must match one of the predefined list options to prevent API manipulation.
*   **Checkbox Handling:** Should properly bind to a list of selected amenities and pass an empty array if none are selected.

---

## SCREEN 2.3 : VIEW PG

### 1. Overview
The **View PG** screen provides a detailed, read-only snapshot of a specific property's information. Its purpose is to allow administrators to securely inspect property details, location data, configuration, and amenities without the risk of accidental modification.

### 2. Screen Preview

```text
+------------------------------------------------------------------------------------+
| View PG Details                                                                    |
|------------------------------------------------------------------------------------|
| Basic Information                                                                  |
| Code / ID               Name                      Type                             |
| P001                    Sai PG                    PG                               |
| Gender Type             Contact Person            Mobile                           |
| Male                    Ravi Kumar                9876543210                       |
| Description                                                                        |
| Near main market                                                                   |
|                                                                                    |
| Location                                                                           |
| Address Line 1          Address Line 2            Area                             |
| Plot 42, Sector 1       Opposite Park             Koramangala                      |
| Landmark                City                      State                            |
| Near Metro Station      Bengaluru                 Karnataka                        |
| Pincode                 Country                                                    |
| 560034                  India                                                      |
|                                                                                    |
| Property Configuration                                                             |
| No. of Rooms / Flats    Room Sharing / Flat Sharing                                |
| 20                      2                                                          |
| Rent                    Property Status                                            |
| 8500.00                 Active                                                     |
|                                                                                    |
| Amenities                                                                          |
| [v] WiFi            [v] RO Water         [x] Washing Machine  [x] Refrigerator     |
| [x] TV              [v] CCTV             [v] Parking          [x] Lift             |
| [v] House Keeping   [v] Food             [x] Kitchen Staff                         |
|                                                                                    |
|                                                     [ Back ] [ Edit ]              |
+------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Code / ID** | Label | N/A | Read-only display. | `P001` | - |
| **Name** | Label | N/A | Read-only display. | `Sai PG` | - |
| **Type** | Label | N/A | Read-only display. | `PG` | - |
| **Gender Type** | Label | N/A | Read-only display. | `Male` | - |
| **Contact Person** | Label | N/A | Read-only display. | `Ravi Kumar` | - |
| **Mobile** | Label | N/A | Read-only display. | `9876543210` | - |
| **Description** | Label | N/A | Read-only display. | `Near main market` | Handles multi-line display gracefully. |
| **Address Line 1** | Label | N/A | Read-only display. | `Plot 42, Sector 1` | - |
| **Address Line 2** | Label | N/A | Read-only display. | `Opposite Park` | Display 'N/A' or hide if empty. |
| **Area** | Label | N/A | Read-only display. | `Koramangala` | - |
| **Landmark** | Label | N/A | Read-only display. | `Near Metro Station` | Display 'N/A' or hide if empty. |
| **City** | Label | N/A | Read-only display. | `Bengaluru` | - |
| **State** | Label | N/A | Read-only display. | `Karnataka` | - |
| **Pincode** | Label | N/A | Read-only display. | `560034` | - |
| **Country** | Label | N/A | Read-only display. | `India` | - |
| **No. of Rooms / Flats**| Label | N/A | Read-only display. | `20` | - |
| **Room/Flat Sharing** | Label | N/A | Read-only display. | `2` | - |
| **Rent** | Label | N/A | Read-only display. | `8500.00` | Formatted with currency if applicable. |
| **Property Status** | Label | N/A | Read-only display. | `Active` | - |
| **Amenities** | Label/Icons | N/A | Read-only display. | `WiFi, RO Water` | Display only selected amenities or check/cross icons. |
| **Back** | Button | N/A | Redirects to "List PG". | N/A | - |
| **Edit** | Button | N/A | Redirects to "Edit PG" for this record. | N/A | Hidden if PG Status is Inactive. |

### 4. Validations

*   **Read-Only Display:** All fields must strictly be non-editable text or labels. No input controls should be rendered.
*   **PG Must Exist:** If the URL contains an invalid or non-existent PG ID, display a 404/Error page or redirect to "List PG" with an error toast ("Property not found").
*   **Handle Deleted/Deactivated Records:** If the record is Inactive, display it with a clear visual warning (e.g., a red banner: "This property is currently inactive"). The "Edit" button should not be rendered for inactive records.

---

## SCREEN 2.4 : EDIT PG

### 1. Overview
The **Edit PG** screen allows administrators to modify the details of an existing, active property. The screen mirrors the Add PG layout but pre-populates all form fields with the current data from the database. It is essential for correcting mistakes, updating contact information, or modifying property configurations and amenities as the business evolves.

### 2. Screen Preview

```text
+------------------------------------------------------------------------------------+
| Edit PG Details                                                                    |
|------------------------------------------------------------------------------------|
| Basic Information                                                                  |
| Code / ID *             Name *                    Type *                           |
| [P001___________]       [Sai PG_________]         [ PG       v ]                   |
| Gender Type *           Contact Person *          Mobile *                         |
| [ Male       v ]        [Ravi Kumar_____]         [9876543210___]                  |
| Description                                                                        |
| [Near main market_______________________________________________________]          |
|                                                                                    |
| Location                                                                           |
| Address Line 1 *        Address Line 2            Area *                           |
| [Plot 42, Sector 1]     [Opposite Park__]         [Koramangala__]                  |
| Landmark                City *                    State *                          |
| [Near Metro Station]    [Bengaluru______]         [Karnataka____]                  |
| Pincode *               Country *                                                  |
| [560034_________]       [India__________]                                          |
|                                                                                    |
| Property Configuration                                                             |
| No. of Rooms / Flats *  Room Sharing / Flat Sharing *                              |
| [20_____________]       [2______________]                                          |
| Rent *                  Property Status *                                          |
| [8500.00________]       [ Active     v ]                                           |
|                                                                                    |
| Amenities                                                                          |
| [v] WiFi            [v] RO Water         [ ] Washing Machine  [ ] Refrigerator     |
| [ ] TV              [v] CCTV             [v] Parking          [ ] Lift             |
| [v] House Keeping   [v] Food             [ ] Kitchen Staff                         |
|                                                                                    |
|                                                     [ Cancel ] [ Update ]          |
+------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Code / ID** | Text Input | Yes | Pre-filled. Max 20 chars. Must be unique. | `P001` | May be configured as read-only depending on system design. |
| **Name** | Text Input | Yes | Pre-filled. Max 100 chars. | `Sai PG` | - |
| **Type** | Dropdown | Yes | Pre-filled. Value 'PG' or 'Apartment'. | `PG` | - |
| **Gender Type** | Dropdown | Yes | Pre-filled. | `Male` | - |
| **Contact Person** | Text Input | Yes | Pre-filled. | `Ravi Kumar` | - |
| **Mobile** | Text Input | Yes | Pre-filled. Exactly 10 digits. | `9876543210` | - |
| **Description** | Text Area | No | Pre-filled. Max 500 chars. | `Near main market` | - |
| **Address Line 1** | Text Input | Yes | Pre-filled. Max 255 chars. | `Plot 42, Sector 1` | - |
| **Address Line 2** | Text Input | No | Pre-filled. | `Opposite Park` | - |
| **Area** | Text Input | Yes | Pre-filled. | `Koramangala` | - |
| **Landmark** | Text Input | No | Pre-filled. | `Near Metro Station` | - |
| **City** | Text Input | Yes | Pre-filled. | `Bengaluru` | - |
| **State** | Text Input | Yes | Pre-filled. | `Karnataka` | - |
| **Pincode** | Text Input | Yes | Pre-filled. Exactly 6 digits. | `560034` | - |
| **Country** | Text Input | Yes | Pre-filled. | `India` | - |
| **No. of Rooms / Flats**| Number Input | Yes | Pre-filled. Positive integer. | `20` | Modifying this might require backend checks if rooms are occupied. |
| **Room/Flat Sharing** | Number Input | Yes | Pre-filled. Positive integer. | `2` | - |
| **Rent** | Number Input | Yes | Pre-filled. Positive number. | `8500.00` | - |
| **Property Status** | Dropdown | Yes | Pre-filled. 'Active' or 'Inactive'. | `Active` | Changing to Inactive has system-wide effects. |
| **Amenities** | Checkboxes | No | Pre-filled based on saved data. | Checked / Unchecked | - |
| **Update** | Button | N/A | Submits the form. Disabled during API call. | N/A | Validates all fields before submission. |
| **Cancel** | Button | N/A | Discards unsaved changes and returns to "List PG" or "View PG". | N/A | - |

### 4. Validations

*   **Existing Data Loading:** On initial load, all fields must accurately reflect the current data stored in the database.
*   **Duplicate Checks (Excluding Current Record):** When validating `Code / ID` or `Name` for duplicates, the backend must exclude the current PG being edited to avoid false-positive duplicate errors.
*   **Required Fields & Formats:** Same standard validations as "Add PG" (e.g., Mobile must be 10 digits, Rent must be positive, Required fields cannot be empty).
*   **Update Confirmation:** If critical configurations (like reducing `No. of Rooms / Flats` or changing `Gender Type`) are modified, prompt a warning: "Changing these settings may conflict with existing tenant allocations. Proceed?"
*   **Property Status Change:** If changing status to 'Inactive', prompt a warning: "Making this property inactive will hide it from active listings. Proceed?"

---

## SCREEN 2.5 : DEACTIVATE PG

### 1. Overview
The **Deactivate PG** screen (typically implemented as a modal or dialog box) acts as a safety checkpoint before changing an active property's status to Inactive. This ensures properties are never permanently deleted (preserving historical and financial data) but are removed from active operations, listings, and tenant onboarding flows.

### 2. Screen Preview

```text
+--------------------------------------------------+
|            Deactivate PG                         |
|--------------------------------------------------|
| Are you sure you want to deactivate this PG?     |
|                                                  |
| This action can be reversed later.               |
|                                                  |
|      [ Cancel ]       [ Deactivate ]             |
+--------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Confirmation Message** | Label | N/A | Displays the warning message. | `Are you sure you want to deactivate...` | Informs user of the consequence. |
| **Cancel** | Button | N/A | Closes the popup without making changes. | N/A | Returns user to the previous state. |
| **Deactivate** | Button | N/A | Triggers the API call to update status to Inactive. | N/A | Disabled after clicking to prevent double submission. |

### 4. Validations

*   **Confirmation Required:** The action requires an explicit click on the "Deactivate" button. It cannot be triggered accidentally.
*   **Prevent Duplicate Requests:** The "Deactivate" button must enter a loading state and disable immediately upon click to prevent multiple simultaneous API requests.
*   **Handle Already Inactive PG:** If the PG is already inactive (e.g., modified in another tab), the backend must gracefully reject the request and the frontend should refresh the list.
*   **Business Rule Restrictions:** Prevent deactivation if business rules dictate otherwise (e.g., if there are active tenants). Display an appropriate error: "Cannot deactivate PG. There are active tenants assigned to this property."
*   **Proper Messages:** On success, close modal and show a green toast: "Property deactivated successfully." On failure, show a red toast with the specific error message.

---

# Member Management Module

## SCREEN 3.1 : LIST MEMBER

### 1. Overview
The **List Member** screen acts as the central hub for managing all PG members (tenants) across properties. It provides administrators with a consolidated, searchable, and sortable table displaying vital member information. From this screen, users can monitor tenant status (Active, Notice Period, Inactive) and easily navigate to view detailed profiles, edit records, or initiate deactivation.

### 2. Screen Preview

```text
+-------------------------------------------------------------------------------------------------------------------------------+
| Member Management                                                            Add New Member                                  |
|-------------------------------------------------------------------------------------------------------------------------------|
| Search : [__________________________]                                                                               Filter     |
|                                                                                                                               |
| ID | Name | Mobile | PG | Room | Bed | Monthly Rent | Status | Actions                                                      |
|----|------|--------|----|------|-----|--------------|--------|--------------------------------------------------------------|
| M01| John | 9876.. | PG1| 101  | A   | 8500.00      | Active | View | Edit | Deactivate                                    |
| M02| Amit | 9123.. | PG2| 202  | B   | 7500.00      | Notice | View | Edit | Deactivate                                    |
|                                                                                                                               |
| Showing 1-10 of 150 Records                                                           Previous | Next                         |
+-------------------------------------------------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Add New Member** | Button | N/A | Click redirects user to the "Add Member" screen. | N/A | Placed at top right. |
| **Search** | Text Input | No | Accepts alphanumeric characters. Supports partial matching. | `John` or `M01` | Searches across ID, Name, Mobile, and PG columns. |
| **Filter** | Button | N/A | Opens advanced filtering options (e.g., filter by Status or PG). | N/A | - |
| **ID** | Data Column | N/A | Sortable. | `M01` | Unique member identifier. |
| **Name** | Data Column | N/A | Sortable. | `John Doe` | Full Name of the member. |
| **Mobile** | Data Column | N/A | Sortable. Must display 10 digits. | `9876543210` | Formatted for readability. |
| **PG** | Data Column | N/A | Sortable. | `Sai PG` | Property where member resides. |
| **Room** | Data Column | N/A | Sortable. | `101` | - |
| **Bed** | Data Column | N/A | Sortable. | `A` | - |
| **Monthly Rent** | Data Column | N/A | Sortable. Formatted as currency. | `8500.00` | - |
| **Status** | Data Column | N/A | Sortable. Displays Active, Notice Period, or Inactive. | `Active` | - |
| **View** | Action Button | N/A | Click redirects to "View Member" screen. | N/A | Available for all records. |
| **Edit** | Action Button | N/A | Click redirects to "Edit Member" screen. | N/A | - |
| **Deactivate** | Action Button | N/A | Click opens "Deactivate Member" modal. | N/A | Hidden/disabled if Status is Inactive. |
| **Records Per Page** | Dropdown | No | Valid integer (e.g., 10, 20, 50). | `10` | Controls table pagination size. |
| **Pagination (Prev/Next)**| Buttons | N/A | Disabled on first/last page respectively. | N/A | - |
| **Total Records** | Label | N/A | Dynamically reflects current search/filter state. | `Showing 1-10 of 150 Records` | - |

### 4. Validations

*   **Search Functionality:** 
    *   Search must support partial matching across fields.
    *   Search input must automatically ignore and trim leading/trailing spaces.
*   **Empty State:** 
    *   If no records exist or search yields zero results, display "No members found."
*   **Invalid Page Handling:**
    *   If the user alters the URL to a non-existent page number, redirect to page 1.
*   **Action Handling:**
    *   Confirmation is required before deactivating a member.
    *   Handle inactive members appropriately (e.g., dim the row or disable the Edit/Deactivate actions based on system policy).

---

## SCREEN 3.2 : ADD MEMBER

### 1. Overview
The **Add Member** screen provides a comprehensive form to register a new tenant into the PG Management System. It captures essential personal details, identity verification documents, emergency contacts, property allocation (PG, Room, Bed), and financial terms (Rent, Security Deposit). Successfully completing this form formally onboards the member into the system.

### 2. Screen Preview

```text
+------------------------------------------------------------------------------------+
| Add New Member                                                                     |
|------------------------------------------------------------------------------------|
| 2.1 Personal Information                                                           |
| ID *                    Full Name *               Mobile Number *                  |
| [_____________]         [_____________]           [_____________]                  |
| Alternate Number        Email                     Occupation *                     |
| [_____________]         [_____________]           [ Dropdown v ]                   |
| Date of Birth *         Gender *                  Company / College Name *         |
| [_____________]         [ Dropdown v ]            [_____________]                  |
|                                                                                    |
| 2.2 Identity Verification                                                          |
| Aadhaar Card *          PAN                       Driving Licence                  |
| [ Text / Upload ]       [_____________]           [_____________]                  |
|                                                                                    |
| 2.3 Emergency Contact                                                              |
| Contact Person Name *   Relationship *                                             |
| [_____________]         [ Dropdown v ]                                             |
|                                                                                    |
| 2.4 Address                                                                        |
| Address Line 1 *        Address Line 2            City *                           |
| [_____________]         [_____________]           [_____________]                  |
| State *                 Pincode *                 Country                          |
| [_____________]         [_____________]           [_____________]                  |
|                                                                                    |
| 2.5 Stay Details                                                                   |
| PG Name *               Room No *                 Bed *                            |
| [ Dropdown v ]          [ Dropdown v ]            [ Dropdown v ]                   |
|                                                                                    |
| 2.6 Rent Details                                                                   |
| Monthly Rent *          Security Deposit *        Maintenance Charge *             |
| [_____________]         [_____________]           [_____________]                  |
| Rent Due Date *         Notice Period *                                            |
| [_____________]         [_____________]                                            |
|                                                                                    |
| 2.7 Member Status                                                                  |
| Status *                Reason                                                     |
| [ Dropdown v ]          [_____________]                                            |
|                                                                                    |
|                                                     [ Cancel ] [ Save ]            |
+------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **ID** | Text Input | Yes | Max 20 chars. Must be unique. | `M001` | System may auto-generate. |
| **Full Name** | Text Input | Yes | Max 100 chars. Alphabetic with spaces. | `John Doe` | - |
| **Mobile Number** | Text Input | Yes | Exactly 10 digits. | `9876543210` | Must be unique. |
| **Alternate Number** | Text Input | No | Exactly 10 digits if provided. | `9123456780` | - |
| **Email** | Text Input | No | Valid email format if provided. Max 100 chars. | `john@example.com` | - |
| **Occupation** | Dropdown | Yes | Value in [Student, Employee, Business, Freelancer, Other] | `Student` | - |
| **Date of Birth** | Date Picker | Yes | Must be a past date. Age >= 16 (or per policy). | `1998-05-15` | - |
| **Gender** | Dropdown | Yes | Value in [Male, Female, Other]. | `Male` | Used for validation against PG Gender rules. |
| **Company / College Name**| Text Input | Yes | Max 150 chars. | `XYZ Tech` | - |
| **Aadhaar Card (Number)** | Text Input | Yes | Exactly 12 digits. | `123456789012` | Must be unique. |
| **Aadhaar Card (Upload)** | File Upload| Yes | Image/PDF size <= 5MB. | `aadhaar.jpg` | Required for verification. |
| **PAN** | Text Input | No | Standard PAN regex (e.g., 5 chars, 4 digits, 1 char). | `ABCDE1234F` | - |
| **Driving Licence** | Text Input | No | Standard Driving Licence format. Max 20 chars. | `DL1420110012345` | - |
| **Contact Person Name** | Text Input | Yes | Max 100 chars. Alphabetic with spaces. | `Jane Doe` | Emergency contact. |
| **Relationship** | Dropdown | Yes | Value in [Father, Mother, Brother, Sister, Friend, Guardian, Other] | `Father` | - |
| **Address Line 1** | Text Input | Yes | Max 255 chars. | `Plot 10, MG Road` | - |
| **Address Line 2** | Text Input | No | Max 255 chars. | `Apt 4B` | - |
| **City** | Text Input | Yes | Max 100 chars. | `Bengaluru` | - |
| **State** | Text Input | Yes | Max 100 chars. | `Karnataka` | - |
| **Pincode** | Text Input | Yes | Exactly 6 digits. | `560001` | - |
| **Country** | Text Input | No | Max 50 chars. | `India` | - |
| **PG Name** | Dropdown | Yes | Value must be a valid, active PG ID. | `Sai PG` | Selecting PG populates Room dropdown. |
| **Room No** | Dropdown | Yes | Value must be a valid Room ID in selected PG. | `101` | Selecting Room populates Bed dropdown. |
| **Bed** | Dropdown | Yes | Value must be an available Bed in selected Room. | `A` | Bed must be currently vacant. |
| **Monthly Rent** | Number Input | Yes | Positive number. Max 2 decimal places. | `8500.00` | - |
| **Security Deposit** | Number Input | Yes | Positive number or 0. | `10000.00` | - |
| **Maintenance Charge** | Number Input | Yes | Positive number or 0. | `500.00` | - |
| **Rent Due Date** | Number Input | Yes | Integer between 1 and 31. | `5` | - |
| **Notice Period** | Number Input | Yes | Positive integer (days). | `30` | - |
| **Status** | Dropdown | Yes | Value in [Active, Notice Period, Inactive]. | `Active` | - |
| **Reason** | Text Area | Cond.* | Required only if Status = Notice Period. | `Relocating` | Displayed conditionally. |
| **Save** | Button | N/A | Submits form. Disabled during API call. | N/A | - |
| **Cancel** | Button | N/A | Discards changes, redirects to "List Member". | N/A | - |

### 4. Validations

*   **Required Fields:** All fields marked with (*) must be filled. Form cannot submit if empty.
*   **Duplicate Mobile Number:** Validate against database; throw error if number already exists for another member.
*   **Aadhaar Validation:**
    *   **Duplicate Aadhaar Number:** Must be unique across all members.
    *   **Format:** Exactly 12 numeric digits.
    *   **Upload:** Image must be PNG/JPG/PDF and under specified size limit (e.g., 5MB).
*   **PAN / DL Format:** Apply specific regex matching for standard Indian PAN and Driving Licence formats if provided.
*   **Date of Birth:** Must be a valid date in the past. Member must meet minimum age requirements.
*   **Mobile / Alternate Number:** Strictly 10 numeric digits.
*   **Email Format:** Must match standard email regex `^[^\s@]+@[^\s@]+\.[^\s@]+$`.
*   **Pincode:** Exactly 6 numeric digits.
*   **Stay Details Selection (PG/Room/Bed):**
    *   Room dropdown must only show rooms belonging to the selected PG.
    *   Bed dropdown must only show currently vacant beds in the selected room.
*   **Financial Validations (Rent, Deposit, Maintenance):** Must be valid positive numbers.
*   **Rent Due Date:** Must be a valid day of the month (1-31).
*   **Status & Conditional Reason:** The `Reason` field must be dynamically displayed and marked as Required *only* if `Status` is set to 'Notice Period'.
*   **Character Limits & Numeric Checks:** Text inputs must not exceed defined database column sizes. Numeric fields must not accept alphabets.

---

## SCREEN 3.3 : VIEW MEMBER

### 1. Overview
The **View Member** screen provides a detailed, read-only display of a member's complete profile. It allows administrators to securely review a tenant's personal data, emergency contacts, submitted documents, stay allocation, and current financial terms without the risk of accidental modifications.

### 2. Screen Preview

```text
+------------------------------------------------------------------------------------+
| View Member Details                                                                |
|------------------------------------------------------------------------------------|
| 2.1 Personal Information                                                           |
| ID                      Full Name                 Mobile Number                    |
| M001                    John Doe                  9876543210                       |
| Alternate Number        Email                     Occupation                       |
| 9123456780              john@example.com          Student                          |
| Date of Birth           Gender                    Company / College Name           |
| 1998-05-15              Male                      XYZ Tech                         |
|                                                                                    |
| 2.2 Identity Verification                                                          |
| Aadhaar Card            PAN                       Driving Licence                  |
| 123456789012 [View]     ABCDE1234F                N/A                              |
|                                                                                    |
| 2.3 Emergency Contact                                                              |
| Contact Person Name     Relationship                                               |
| Jane Doe                Father                                                     |
|                                                                                    |
| 2.4 Address                                                                        |
| Address Line 1          Address Line 2            City                             |
| Plot 10, MG Road        Apt 4B                    Bengaluru                        |
| State                   Pincode                   Country                          |
| Karnataka               560001                    India                            |
|                                                                                    |
| 2.5 Stay Details                                                                   |
| PG Name                 Room No                   Bed                              |
| Sai PG                  101                       A                                |
|                                                                                    |
| 2.6 Rent Details                                                                   |
| Monthly Rent            Security Deposit          Maintenance Charge               |
| 8500.00                 10000.00                  500.00                           |
| Rent Due Date           Notice Period                                              |
| 5                       30 days                                                    |
|                                                                                    |
| 2.7 Member Status                                                                  |
| Status                  Reason                                                     |
| Notice Period           Relocating to another city                                 |
|                                                                                    |
|                                                     [ Back ] [ Edit ]              |
+------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **All Form Fields** | Label | N/A | Read-only display of all Add Member fields. | `John Doe` | No editable controls. Displays 'N/A' if empty. |
| **Aadhaar Card [View]** | Link/Button | N/A | Opens uploaded document in a new tab or modal. | N/A | - |
| **Back** | Button | N/A | Redirects to "List Member". | N/A | - |
| **Edit** | Button | N/A | Redirects to "Edit Member" screen. | N/A | Hidden for Inactive members (if policy dictates). |

### 4. Validations

*   **Read-Only Display:** All data must be rendered as text labels. No input fields should be accessible.
*   **Member Must Exist:** If a user accesses an invalid Member ID via URL, display a 404/Error page or redirect to "List Member".
*   **Missing Documents Handling:** If an optional document (e.g., Driving Licence) was not uploaded, gracefully display "N/A" or "Not Provided".
*   **Handle Inactive Members:** If the member status is Inactive, display a clear warning banner (e.g., "This member is currently inactive").

---

## SCREEN 3.4 : EDIT MEMBER

### 1. Overview
The **Edit Member** screen allows authorized users to modify the details of an existing member. This screen mirrors the layout of the Add Member form but pre-fills all inputs with the member's current data. It is used to update contact information, change room allocations, or update member status.

### 2. Screen Preview

```text
+------------------------------------------------------------------------------------+
| Edit Member Details                                                                |
|------------------------------------------------------------------------------------|
| 2.1 Personal Information                                                           |
| ID *                    Full Name *               Mobile Number *                  |
| [M001___________]       [John Doe_______]         [9876543210___]                  |
| Alternate Number        Email                     Occupation *                     |
| [9123456780_____]       [john@example.com]        [ Student    v ]                 |
| Date of Birth *         Gender *                  Company / College Name *         |
| [1998-05-15_____]       [ Male       v ]          [XYZ Tech______]                 |
|                                                                                    |
| 2.2 Identity Verification                                                          |
| Aadhaar Card *          PAN                       Driving Licence                  |
| [123456789012 ] [File]  [ABCDE1234F_____]         [_____________]                  |
|                                                                                    |
| 2.3 Emergency Contact                                                              |
| Contact Person Name *   Relationship *                                             |
| [Jane Doe_______]       [ Father     v ]                                           |
|                                                                                    |
| 2.4 Address                                                                        |
| Address Line 1 *        Address Line 2            City *                           |
| [Plot 10, MG Road]      [Apt 4B_________]         [Bengaluru____]                  |
| State *                 Pincode *                 Country                          |
| [Karnataka______]       [560001_________]         [India________]                  |
|                                                                                    |
| 2.5 Stay Details                                                                   |
| PG Name *               Room No *                 Bed *                            |
| [ Sai PG     v ]        [ 101        v ]          [ A          v ]                 |
|                                                                                    |
| 2.6 Rent Details                                                                   |
| Monthly Rent *          Security Deposit *        Maintenance Charge *             |
| [8500.00________]       [10000.00_______]         [500.00_______]                  |
| Rent Due Date *         Notice Period *                                            |
| [5______________]       [30_____________]                                          |
|                                                                                    |
| 2.7 Member Status                                                                  |
| Status *                Reason                                                     |
| [ Notice Period v]      [Relocating_____]                                          |
|                                                                                    |
|                                                     [ Cancel ] [ Update ]          |
+------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **All Add Member Fields**| Input/Dropdown | Yes/No | Pre-filled with existing data. | `John Doe` | Values loaded automatically from database. |
| **Update** | Button | N/A | Submits changes. Disabled during API call. | N/A | - |
| **Cancel** | Button | N/A | Discards changes, redirects back to List/View. | N/A | - |

### 4. Validations

*   **Existing Data Loading:** On initial load, all fields must accurately reflect the member's current data.
*   **Duplicate Validation (Excluding Current Record):** When validating `Mobile Number` or `Aadhaar Number` for uniqueness, the backend must exclude the current member ID to prevent false positive errors.
*   **Standard Form Validations:** Retain all validations present in the Add Member screen (Required fields, Number formats, Document sizes, etc.).
*   **Conditional Reason Validation:** If `Status` is changed to 'Notice Period', the `Reason` field must become visible and required.
*   **Update Confirmation:** If the PG, Room, or Bed is modified, prompt a warning indicating accommodation change.

---

## SCREEN 3.5 : DEACTIVATE MEMBER

### 1. Overview
The **Deactivate Member** screen functions as a confirmation checkpoint before setting a member's status to Inactive. This ensures the tenant is removed from active rosters (vacating their allocated bed) without permanently deleting their historical, personal, or financial records from the system.

### 2. Screen Preview

```text
+------------------------------------------------------+
|               Deactivate Member                      |
|------------------------------------------------------|
| Are you sure you want to deactivate this member?     |
|                                                      |
| This action can be reversed later.                   |
|                                                      |
|      [ Cancel ]            [ Deactivate ]            |
+------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Confirmation Message**| Label | N/A | Displays the warning message. | `Are you sure you want to deactivate...` | Informs user of the consequence. |
| **Cancel** | Button | N/A | Closes the modal. | N/A | - |
| **Deactivate** | Button | N/A | Triggers API to mark member as Inactive. | N/A | Disabled upon click. |

### 4. Validations

*   **Confirmation Required:** The action requires an explicit click on the "Deactivate" button.
*   **Prevent Duplicate Requests:** The "Deactivate" button must disable immediately upon click to prevent multiple API calls.
*   **Handle Already Inactive Member:** If the member is already deactivated, the backend should gracefully reject the request, and the frontend should refresh the list.
*   **Prevent Deactivation if Validation Fails:** If business rules dictate that a member cannot be deactivated (e.g., pending rent dues), display a blocking error message: "Cannot deactivate member with pending dues."
*   **Proper Messages:** Show a success toast ("Member deactivated successfully") upon successful deactivation, or an error toast describing why the deactivation failed.

---

# Rent Management Module

## SCREEN 4.1 : RENT LIST

### 1. Overview
The **Rent List** screen acts as the financial command center for the PG Management System, focusing specifically on member rent collections and tracking. It displays a consolidated, real-time list of all active members whose rent is categorized as Due Today, Upcoming Due, or Overdue. This interface allows administrators to track pending payments, initiate manual payment entries, edit discrepancies, and upload bank statements for automated payment reconciliation.

### 2. Screen Preview

```text
+----------------------------------------------------------------------------------------------------------------------------------------+
| Rent Management                                                       Upload Bank Statement                                           |
|----------------------------------------------------------------------------------------------------------------------------------------|
| Search : [__________________________]                                                                              Filter              |
|                                                                                                                                        |
| Member | PG | Room | Rent | Due Date | Status | Payment Date | Days Remaining | Actions                                             |
|--------|----|------|------|----------|--------|--------------|----------------|-----------------------------------------------------|
| Rahul  | PG1| 101  | 6000 | 05-Aug   | Due    | -            | 2 Days         | View | Edit | Mark as Paid                           |
| Amit   | PG1| 102  | 5500 | 01-Aug   | Overdue| -            | -2 Days        | View | Edit | Mark as Paid                           |
|                                                                                                                                        |
| Showing 1-10 of 50 Records                                                       Previous | Next                                     |
+----------------------------------------------------------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Upload Bank Statement** | Button | N/A | Opens the "Upload Bank Statement" modal. | N/A | Prominently placed for quick access. |
| **Search** | Text Input | No | Accepts alphanumeric input. Partial matching supported. | `Rahul` | Searches across Member Name, PG, and Room. |
| **Filter** | Button | N/A | Opens advanced filters (e.g., Status: Overdue). | N/A | - |
| **Member** | Data Column | N/A | Sortable. Displays Member Name. | `Rahul Patel` | - |
| **PG** | Data Column | N/A | Sortable. | `Sai PG` | - |
| **Room** | Data Column | N/A | Sortable. | `101` | - |
| **Rent** | Data Column | N/A | Sortable. Numeric format. | `6000.00` | - |
| **Due Date** | Data Column | N/A | Sortable. Date format. | `05-Aug-2026` | - |
| **Status** | Data Column | N/A | Sortable. (Upcoming, Due Today, Overdue). | `Overdue` | Color-coded (e.g., Red for Overdue). |
| **Payment Date** | Data Column | N/A | Sortable. Date format. | `-` | Usually empty here unless filtered for 'Paid'. |
| **Days Remaining** | Data Column | N/A | Sortable. Calculated field. | `2 Days` | Can display negative for Overdue. |
| **View** | Action Button | N/A | Redirects to View Rent Details screen. | N/A | Available on all rows. |
| **Edit** | Action Button | N/A | Redirects to Edit Rent screen. | N/A | - |
| **Mark as Paid** | Action Button | N/A | Opens the "Mark as Paid" modal. | N/A | - |
| **Records Per Page** | Dropdown | No | Valid integer selection. | `10` | Controls pagination size. |
| **Pagination (Prev/Next)**| Buttons | N/A | Disabled if on the first/last page. | N/A | - |
| **Total Records** | Label | N/A | Dynamically reflects search/filter results. | `Showing 1-10 of 50 Records` | - |

### 4. Validations

*   **Search Functionality:** 
    *   Search must support partial matching (e.g., typing "Rah" matches "Rahul").
    *   Search must automatically ignore and trim leading/trailing spaces.
*   **Empty State:** 
    *   If no records exist, display a friendly empty state message (e.g., "No rent records found for the current criteria").
*   **Invalid Page Handling:**
    *   If the user navigates to an invalid/non-existent page via URL, seamlessly redirect to Page 1.
*   **Active Members Only:**
    *   Only rent records for active members should appear in this default list. Inactive/vacated members should be excluded unless explicitly filtered.
*   **Completed Rent Handling:**
    *   Members whose rent is completely paid for the current cycle should not appear in this default list (unless the 'Paid' filter is applied).
*   **Action Handling:**
    *   Confirmation is required before marking rent as paid via the Mark as Paid modal.

---

## SCREEN 4.2 : VIEW RENT DETAILS

### 1. Overview
The **View Rent Details** screen provides a comprehensive, read-only summary of a specific member's rent configuration and payment status for the current billing cycle. It allows administrators to securely review rent breakdowns, due dates, outstanding amounts, and payment histories without the risk of accidental data modification.

### 2. Screen Preview

```text
+------------------------------------------------------------------------------------+
| View Rent Details                                                                  |
|------------------------------------------------------------------------------------|
| Member Information                                                                 |
| Member Name                  PG Name                   Room Number                 |
| Rahul Patel                  Sai PG                    101                         |
| Bed Number                                                                         |
| A                                                                                  |
|                                                                                    |
| Rent Breakdown                                                                     |
| Monthly Rent                 Security Deposit          Maintenance Charge          |
| 6000.00                      10000.00                  500.00                      |
|                                                                                    |
| Payment Status                                                                     |
| Due Date                     Payment Status            Outstanding Amount          |
| 05-Aug-2026                  Overdue                   6500.00                     |
| Payment Date                 Remarks                                               |
| N/A                          Late payment fee may apply.                           |
|                                                                                    |
|                                                     [ Back ] [ Edit ]              |
+------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Member Name** | Label | N/A | Read-only display. | `Rahul Patel` | - |
| **PG Name** | Label | N/A | Read-only display. | `Sai PG` | - |
| **Room Number** | Label | N/A | Read-only display. | `101` | - |
| **Bed Number** | Label | N/A | Read-only display. | `A` | - |
| **Monthly Rent** | Label | N/A | Read-only display. Formatted as currency. | `6000.00` | - |
| **Security Deposit** | Label | N/A | Read-only display. Formatted as currency. | `10000.00` | - |
| **Maintenance Charge** | Label | N/A | Read-only display. Formatted as currency. | `500.00` | - |
| **Due Date** | Label | N/A | Read-only display. Date format. | `05-Aug-2026` | - |
| **Payment Status** | Label | N/A | Read-only display. | `Overdue` | - |
| **Outstanding Amount** | Label | N/A | Read-only display. Formatted as currency. | `6500.00` | Monthly Rent + Maintenance |
| **Payment Date** | Label | N/A | Read-only display. | `N/A` | Displays date if paid. |
| **Remarks** | Label | N/A | Read-only display. | `Late payment fee may apply.` | Displays 'N/A' if empty. |
| **Back** | Button | N/A | Redirects to "Rent List". | N/A | - |
| **Edit** | Button | N/A | Redirects to "Edit Rent" screen. | N/A | Optional based on status. |

### 4. Validations

*   **Rent Record Must Exist:** If a user accesses an invalid Rent ID via URL, display an error page or redirect to "Rent List" with a toast message.
*   **Read-Only Display:** All data must be rendered as non-editable text labels. No input fields should be accessible.
*   **Handle Inactive Members:** If the associated member is inactive, display a clear visual warning banner (e.g., "Note: This tenant has vacated.").

---

## SCREEN 4.3 : EDIT RENT

### 1. Overview
The **Edit Rent** screen enables administrators to modify specific financial details for a member's current billing cycle before the payment is formally marked as completed. It is useful for making one-off adjustments, waiving maintenance fees, altering due dates for a specific month, or adding administrative remarks.

### 2. Screen Preview

```text
+------------------------------------------------------------------------------------+
| Edit Rent Details                                                                  |
|------------------------------------------------------------------------------------|
| (Member: Rahul Patel - Sai PG - Room 101)                                          |
|                                                                                    |
| Monthly Rent *          Maintenance Charge *      Due Date *                       |
| [6000.00________]       [500.00_________]         [05-Aug-2026____]                |
|                                                                                    |
| Remarks                                                                            |
| [Late payment fee may apply.____________________________________________]          |
|                                                                                    |
|                                                     [ Cancel ] [ Update ]          |
+------------------------------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Monthly Rent** | Number Input | Yes | Pre-filled. Positive number. Max 2 decimal places. | `6000.00` | Edits apply to current cycle. |
| **Maintenance Charge** | Number Input | Yes | Pre-filled. Positive number or 0. | `500.00` | - |
| **Due Date** | Date Picker | Yes | Pre-filled. Valid date format. | `05-Aug-2026` | - |
| **Remarks** | Text Area | No | Pre-filled. Max 500 chars. | `Late payment...` | - |
| **Update** | Button | N/A | Submits changes. Disabled during API call. | N/A | Validates fields before submission. |
| **Cancel** | Button | N/A | Discards changes, redirects back. | N/A | - |

### 4. Validations

*   **Existing Data Loading:** On initial load, all fields must accurately reflect the rent details currently stored in the database.
*   **Required Fields:** All mandatory fields (*) must contain valid data before allowing an update.
*   **Numeric Validation for Rent:** `Monthly Rent` and `Maintenance Charge` must strictly accept positive numeric values and handle up to 2 decimal places.
*   **Due Date Validation:** Must be a valid, real date.
*   **Character Limits:** `Remarks` must not exceed the defined database column size (e.g., 500 chars). Restrict typing beyond this limit.
*   **Update Confirmation:** Prompt a confirmation dialog before applying financial changes to ensure accuracy.

---

## SCREEN 4.4 : MARK AS PAID

### 1. Overview
The **Mark as Paid** screen is a dedicated modal used to manually confirm and record the receipt of a member's rent payment. This action formally updates the outstanding balance, logs the transaction date, specifies the payment method, and updates the rent status to 'Paid' without deleting or archiving the core rent record.

### 2. Screen Preview

```text
+------------------------------------------------------+
|                  Mark Rent as Paid                   |
|------------------------------------------------------|
| Member : Rahul Patel                                 |
| Rent Amount : ₹6,500                                 |
|                                                      |
| Payment Date * : [ 05-Aug-2026 (Date Picker) ]       |
| Payment Mode * : [ Dropdown v ]                      |
| Remarks        : [_________________________]         |
|                                                      |
|      [ Cancel ]          [ Confirm Payment ]         |
+------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Payment Date** | Date Picker | Yes | Defaults to current date. Cannot be a future date. | `05-Aug-2026` | - |
| **Payment Mode** | Dropdown | Yes | Value in [Cash, UPI, Bank Transfer, Cheque, Card]. | `UPI` | - |
| **Remarks** | Text Input | No | Max 255 chars. | `Paid via Google Pay`| Transaction ID can be stored here. |
| **Confirm Payment** | Button | N/A | Triggers API to mark rent as paid. | N/A | Disabled upon click to prevent double submission. |
| **Cancel** | Button | N/A | Closes the popup. | N/A | - |

### 4. Validations

*   **Required Fields:** `Payment Date` and `Payment Mode` must be provided.
*   **Payment Date Validation:** The selected date cannot be in the future.
*   **Prevent Duplicate Payment Confirmation:** The "Confirm Payment" button must enter a loading state and disable immediately upon click to prevent multiple overlapping API requests.
*   **Handle Already Paid Rent:** If the rent was marked paid concurrently (e.g., in another session or via automated bank upload), reject the request gracefully and refresh the UI.
*   **Proper Messages:** On success, close the modal and display a green toast (e.g., "Payment recorded successfully."). On failure, display a specific error toast.

---

## SCREEN 4.5 : UPLOAD BANK STATEMENT

### 1. Overview
The **Upload Bank Statement** screen provides a facility to upload official bank statements in PDF format. The system processes these statements to automatically detect matching rent payments (e.g., via transaction references or exact amounts) and assists the administrator in bulk payment reconciliation. It is designed to securely handle and parse password-protected bank PDFs.

### 2. Screen Preview

```text
+--------------------------------------------------------------+
|                 Upload Bank Statement                        |
|--------------------------------------------------------------|
| Bank Name *          [ Dropdown v ]                          |
| Statement Month *    [ Month Picker v ]                      |
| Upload PDF *         [ Choose File ] (No file chosen)        |
| PDF Password         [____________________]                  |
|                                                              |
|             [ Cancel ]             [ Upload ]                |
+--------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Bank Name** | Dropdown | Yes | Predefined list of supported banks. | `HDFC Bank` | Required for selecting correct parsing logic. |
| **Statement Month**| Month Picker | Yes | Valid past or current month. | `August 2026` | Helps narrow down reconciliation scope. |
| **Upload PDF** | File Upload | Yes | Must be a `.pdf` file. Max 10MB. | `statement.pdf` | - |
| **PDF Password** | Password Input| No | Text input. | `Rahu123` | Required only if the PDF is encrypted. |
| **Upload** | Button | N/A | Submits the file. Disabled during processing. | N/A | Shows progress indicator when active. |
| **Cancel** | Button | N/A | Closes the modal. | N/A | - |

### 4. Validations

*   **Required Fields:** `Bank Name`, `Statement Month`, and `Upload PDF` are strictly required.
*   **File Validation:** 
    *   Only files with a `.pdf` extension/MIME type are permitted.
    *   Maximum file size validation (e.g., reject files > 10MB).
*   **Invalid or Corrupted PDF:** If the file cannot be parsed or read, display an error: "Invalid or corrupted PDF file uploaded."
*   **Incorrect PDF Password:** If the PDF is encrypted and the provided password fails to decrypt it, display: "Incorrect PDF Password."
*   **Duplicate Statement Detection:** Warn or reject if a statement for the exact same Bank and Month has already been processed recently, prompting the user for confirmation.
*   **Upload Progress Handling:** Display a loading spinner or progress bar to indicate that parsing is underway.
*   **Concurrency:** Prevent multiple file uploads while one is actively processing by disabling the interface.
*   **Proper Messages:** Display a success toast summarizing reconciliation (e.g., "Statement uploaded. 15 payments matched.") or a detailed error message upon failure.

---

# Payment Verification Module

## SCREEN 5.1 : PAYMENT VERIFICATION PAGE

### 1. Overview
The **Payment Verification** page provides a secure, member-facing interface to facilitate seamless rent collections. This page is accessed exclusively via a unique, secure payment link dispatched to the tenant through WhatsApp on their scheduled rent due date. The primary purpose of this screen is to display the member's current rent obligations, offer a selection of preferred UPI payment applications (such as Google Pay, PhonePe, or Paytm), and provide a facility to upload a payment screenshot as proof of transaction. Upon successful submission, the system automatically transitions the payment status from 'Pending' to 'In Review', displaying a confirmation popup to assure the member that their transaction is undergoing manual verification within a 48-hour SLA.

### 2. Screen Preview

```text
+------------------------------------------------------------------------------------+
|                        Monthly Rent Payment                                        |
|------------------------------------------------------------------------------------|
| Member Name  : Rahul Patel                                                         |
| PG Name      : Sunshine PG                                                         |
| Room         : 102                                                                 |
| Amount       : ₹6,000                                                              |
| Due Date     : 05-Aug-2026                                                         |
|------------------------------------------------------------------------------------|
| Select Payment Method                                                              |
|                                                                                    |
| ( ) Google Pay                                                                     |
| ( ) PhonePe                                                                        |
| ( ) Paytm                                                                          |
|                                                                                    |
|                         [ Pay ₹6,000 ]                                             |
|                                                                                    |
|------------------------------------------------------------------------------------|
| Note                                                                               |
| Please upload a payment screenshot in which the Transaction ID is clearly visible. |
| Screenshots without a visible Transaction ID may be rejected during verification.  |
|------------------------------------------------------------------------------------|
| Upload Payment Screenshot                                                          |
| [ Choose File ] (No file chosen)                                                   |
|                                                                                    |
|                         [ Submit Payment ]                                         |
+------------------------------------------------------------------------------------+
```

After submission display popup:

```text
+---------------------------------------------------------------+
|                 Payment Submitted Successfully                |
|---------------------------------------------------------------|
| Thank you for your payment.                                   |
|                                                               |
| Your payment has been submitted successfully.                 |
|                                                               |
| Payment Status : In Review                                    |
|                                                               |
| Our team will verify your payment within 48 hours.            |
|                                                               |
| You will receive confirmation once verification is completed. |
|                                                               |
|                     [ Close ]                                 |
+---------------------------------------------------------------+
```

### 3. Screen Fields Table

| Field Name | Type | Required | Validation | Example | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Member Name** | Label | N/A | Read-only. Must match database record. | `Rahul Patel` | Informs member of the account being paid. |
| **PG Name** | Label | N/A | Read-only. | `Sunshine PG` | Property identifier. |
| **Room Number** | Label | N/A | Read-only. | `102` | Member's allocated room. |
| **Monthly Rent** | Label | N/A | Read-only. Formatted as currency. | `₹6,000` | The exact payable amount. |
| **Due Date** | Label | N/A | Read-only. Formatted as DD-MMM-YYYY. | `05-Aug-2026` | - |
| **Payment Method** | Radio Group | Yes | Exactly one selection is required. | `Google Pay` | Controls the UPI deep-link routing. |
| **Google Pay Option** | Radio Button | No | Belongs to Payment Method group. | `Selected` | Triggers Google Pay Intent. |
| **PhonePe Option** | Radio Button | No | Belongs to Payment Method group. | `Unselected` | Triggers PhonePe Intent. |
| **Paytm Option** | Radio Button | No | Belongs to Payment Method group. | `Unselected` | Triggers Paytm Intent. |
| **Pay Button** | Button | N/A | Disabled until a payment method is selected. | `Pay ₹6,000` | Initiates the selected UPI payment flow. |
| **Instruction Note**| Text Block | N/A | Read-only informative text. | `Please upload a payment...` | Guides member on acceptable proofs. |
| **Payment Screenshot Upload** | File Input | Yes | Accepts `.jpg`, `.jpeg`, `.png`. Max size validated. | `screenshot.png` | Required for manual verification. |
| **Submit Button** | Button | N/A | Disabled if no file is selected or during upload. | `Submit Payment` | Finalizes the transaction submission. |
| **Success Popup** | Modal | N/A | Displayed only on successful backend submission. | `Payment Submitted...` | Confirms state change to 'In Review'. |

### 4. Validations

**General Validations:**
*   **Payment Link Validity:** The accessed payment link must be structurally valid, authentic, and securely tied to a specific rent invoice.
*   **Link Expiration:** The payment link must strictly expire upon successful payment submission or after passing its system-configured validity period (e.g., 72 hours).
*   **Record Existence:** The system must verify that the corresponding Member record and Rent record still exist and are active in the database.
*   **Already Paid Check:** The page must reject access and display an "Already Paid" message if the associated rent record's status is already marked as 'Paid'.

**Payment Method Validations:**
*   **Single Selection:** Exactly one payment method (Google Pay, PhonePe, or Paytm) must be selected before payment initiation.
*   **Button State:** The "Pay" button must remain visually disabled and functionally inactive until a valid payment method is selected.

**Screenshot Upload Validations:**
*   **Mandatory Field:** Uploading a payment screenshot is strictly mandatory. Submission cannot proceed without a selected file.
*   **Allowed Formats:** The file input must restrict and validate formats to only allow `.jpg`, `.jpeg`, and `.png` image files.
*   **Size Limitation:** Enforce a maximum file size limit (e.g., 5MB) on the client and server side to prevent resource exhaustion.
*   **File Integrity:** Perform a backend validation to ensure the uploaded file is not a corrupted image or a malicious disguised file.
*   **Upload Progress:** Implement upload progress handling (e.g., progress bar or percentage) to inform the user during large file uploads on slower networks.

**Transaction Screenshot Specifics:**
*   **Visibility Warning:** Clearly display a persistent note informing the member that the Transaction ID must be explicitly visible in the uploaded image.
*   **Empty Rejection:** Reject the final submission and display an inline error if the user attempts to submit without a successfully attached screenshot.

**Submission Validations:**
*   **Duplicate Prevention:** Disable the Submit button immediately upon the first click to prevent duplicate submissions via rapid clicking.
*   **Processing State:** Show a loading spinner or "Processing..." text on the Submit button while the file is uploading and the API request is in flight.
*   **Success Feedback:** Display the predefined success popup modal immediately after receiving a 200 OK response from the submission endpoint.

**Status Update Behavior:**
*   After a successful submission, the backend must autonomously update the rent record's payment status to `In Review`. This ensures the administration dashboard reflects the pending verification state for manual admin review.

**Success Popup Content:**
*   The system must explicitly display the following confirmation message in the popup:
    *"Your payment has been submitted successfully.*
    *Payment Status: In Review*
    *Our team will verify your payment within 48 hours.*
    *You will receive confirmation once the verification is completed."*

**Error Handling & Edge Cases:**
*   **Invalid payment link:** Display "The payment link you clicked is invalid or malformed."
*   **Expired payment link:** Display "This payment link has expired. Please request a new link from your PG administrator."
*   **Missing payment screenshot:** Display "Please attach a payment screenshot before submitting."
*   **Unsupported file format:** Display "Invalid file format. Only JPG, JPEG, and PNG images are allowed."
*   **File upload failed:** Display "Failed to upload the screenshot. Please check your connection and try again."
*   **Server unavailable:** Display "Our servers are currently unreachable. Please try submitting again in a few minutes."
*   **Duplicate payment submission:** Display "A payment proof for this rent cycle is already under review."
*   **Already paid rent:** Display "This rent cycle has already been marked as Paid. No further action is required."

---

# Filter Configuration

# MODULE 1

Dashboard Filters

## Overview

Dashboard filters are critical for providing dynamic, real-time insights tailored to specific operational contexts. Filtering allows the administrator to slice and dice high-level business data across various dimensions such as properties, timeframes, or tenant demographics. Dashboard filters should dynamically update every KPI, Chart, Table, and Alert without requiring a full page reload, ensuring a highly interactive and responsive executive overview.

---

## Filter List

| Filter | Type | Values | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| **PG** | Dropdown | All, [List of Active PGs] | All | Filters data for a specific property or all properties. |
| **Property Type** | Dropdown | All, PG, Apartment | All | Filters data based on the business model type. |
| **Gender Type** | Dropdown | All, Male, Female, Co-Living | All | Segregates occupancy and revenue by gender allocation. |
| **Member Status** | Dropdown | All, Active, Notice, Inactive | All | Narrows down metrics based on current tenant state. |
| **Rent Status** | Dropdown | All, Paid, Pending, Overdue | All | Filters financial figures and lists by collection status. |
| **Date Range** | Date Picker | Custom From & To Dates | Current Month | Narrows data to a specific operational window. |
| **Month** | Dropdown | Jan, Feb, Mar, etc. | Current Month | Filters monthly recurring metrics. |
| **Year** | Dropdown | 2024, 2025, 2026, etc. | Current Year | Filters annual aggregations. |

---

## Filter Preview

```text
+------------------------------------------------------------------------------------------------------+
| PG ▼ | Property Type ▼ | Member Status ▼ | Rent Status ▼ | Month ▼ | Year ▼ | Reset |
+------------------------------------------------------------------------------------------------------+
```

---

## Filter Behaviour

*   **KPI Cards:** Automatically recalculate counts (e.g., Total Occupied Beds) and financial sums (e.g., Rent Collected) based strictly on the applied filters.
*   **Charts:** Redraw axes, datasets, and legends instantly to reflect the filtered demographic or financial window.
*   **Tables:** Re-query and render only the rows that match the filter criteria (e.g., filtering for "Overdue" will only populate the Overdue Rent table).
*   **Alerts:** Dynamically hide or show operational alerts depending on the context (e.g., filtering for a specific PG will only show alerts relevant to that property).

---

# MODULE 2

PG Management Filters

## Overview

Filtering PG records is essential for administrators to efficiently manage and navigate a large portfolio of properties. It allows quick isolation of specific properties based on operational status, geographical location, or configuration, reducing time spent searching and streamlining property-specific administrative actions.

---

## Filter List

| Filter | Type | Values | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| **PG Name** | Text Input | User Input (Alphanumeric) | Empty | Filters properties by partial or full name match. |
| **PG Code** | Text Input | User Input (Alphanumeric) | Empty | Filters properties by unique identifier. |
| **Property Type** | Dropdown | All, PG, Apartment | All | Segregates by structural business model. |
| **Gender Type** | Dropdown | All, Male, Female, Co-Living | All | Filters properties catering to specific demographics. |
| **Property Status** | Dropdown | All, Active, Inactive | Active | Hides or shows non-operational properties. |
| **City** | Dropdown | All, [List of Cities] | All | Filters properties by geographical city. |
| **State** | Dropdown | All, [List of States] | All | Filters properties by geographical state. |

---

## Filter Preview

```text
+----------------------------------------------------------------------------------------------------+
| PG ▼ | Type ▼ | Gender ▼ | Status ▼ | City ▼ | State ▼ | Reset |
+----------------------------------------------------------------------------------------------------+
```

---

## Filter Behaviour

Applying any filter instantly triggers a backend query to refine the PG list. Filtering allows identifying active vs inactive properties quickly, or drilling down into geographical spread. The table must recalculate pagination dynamically based on the filtered dataset. Sorting mechanisms must respect the currently active filters.

---

# MODULE 3

Member Management Filters

## Overview

Filtering member records is a critical function to handle tenant lifecycles efficiently. As member volumes grow, administrators require robust filtering to locate specific individuals, monitor occupancy trends, track upcoming departures, or identify tenants sharing specific attributes like location or occupation.

---

## Filter List

| Filter | Type | Values | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Member Name** | Text Input | User Input (Alphanumeric) | Empty | Filters members by their full or partial name. |
| **Mobile Number** | Text Input | User Input (Numeric) | Empty | Filters members by their contact number. |
| **PG Name** | Dropdown | All, [List of PGs] | All | Isolates tenants residing in a specific property. |
| **Room Number** | Text Input | User Input (Alphanumeric) | Empty | Narrows down to members in a specific room. |
| **Bed Number** | Text Input | User Input (Alphanumeric) | Empty | Locates the specific bed allocation. |
| **Occupation** | Dropdown | All, Student, Working Pro. | All | Filters by tenant professional status. |
| **Gender** | Dropdown | All, Male, Female | All | Filters tenants by gender. |
| **Member Status** | Dropdown | All, Active, Notice, Inactive | Active | Filters tenants based on their current residency phase. |
| **Joining Date** | Date Picker | Custom Date Range | Empty | Filters members onboarded within a specific period. |
| **City** | Dropdown | All, [List of Cities] | All | Filters tenants originating from a specific city. |

---

## Filter Preview

```text
+----------------------------------------------------------------------------------------------------------------+
| Member ▼ | PG ▼ | Room ▼ | Occupation ▼ | Gender ▼ | Status ▼ | Joining Date ▼ | Reset |
+----------------------------------------------------------------------------------------------------------------+
```

---

## Filter Behaviour

Applying filters immediately subsets the member directory. Searching by Name/Mobile works in tandem with dropdown filters (e.g., searching "Rahul" while filtered by "Active" status). The paginated table dynamically adjusts to display only matching results. The 'Reset' action clears all applied parameters, restoring the default 'Active' member view.

---

# MODULE 4

Rent Management Filters

## Overview

Filtering rent records is the cornerstone of revenue tracking. It empowers administrators to rapidly identify payment bottlenecks, isolate overdue accounts, project upcoming cash flows, and reconcile financial data month-over-month. Effective filtering ensures no outstanding dues slip through the cracks.

---

## Filter List

| Filter | Type | Values | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Member Name** | Text Input | User Input (Alphanumeric) | Empty | Filters financial records for a specific tenant. |
| **PG Name** | Dropdown | All, [List of PGs] | All | Filters rent collections for a specific property. |
| **Room Number** | Text Input | User Input (Alphanumeric) | Empty | Isolates rent data for a particular room. |
| **Payment Status** | Dropdown | Paid, Pending, Overdue | All | Filters records based on rent collection status. |
| **Due Date** | Date Picker | Custom Date Range | Current Month | Filters records based on expected payment dates. |
| **Payment Date** | Date Picker | Custom Date Range | Empty | Filters records based on actual transaction dates. |
| **Rent Amount** | Number Input | Custom Range (Min/Max) | Empty | Filters records by base rental value. |
| **Outstanding Amount**| Number Input | Custom Range (Min/Max) | Empty | Filters records by remaining balance due. |
| **Month** | Dropdown | Jan, Feb, Mar, etc. | Current Month | Isolates the billing cycle month. |
| **Year** | Dropdown | 2024, 2025, 2026, etc. | Current Year | Isolates the billing cycle year. |

---

## Filter Preview

```text
+----------------------------------------------------------------------------------------------------------------------+
| PG ▼ | Member ▼ | Status ▼ | Due Date ▼ | Payment Date ▼ | Month ▼ | Year ▼ | Reset |
+----------------------------------------------------------------------------------------------------------------------+
```

---

## Filter Behaviour

Every filter dynamically narrows down the dataset presented in the rent management view to address highly specific administrative queries.

*   Multiple filters can be applied simultaneously (e.g., 'Overdue' AND 'PG1' AND 'Month: August') to drill down into complex datasets.
*   Filters should persist across views and stay active until the 'Reset' button is explicitly clicked, preventing the loss of context during operations.
*   Search should work together with filters seamlessly; keyword inputs refine the currently filtered dataset.
*   Pagination should update immediately after filtering, reflecting the correct number of pages for the filtered record set.
*   Sorting should work after filtering, accurately ordering the subset of data generated by the filter conditions.
*   Dashboard data should refresh instantly when filters change, ensuring that metrics like 'Pending Rent' align with the records displayed.
