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

