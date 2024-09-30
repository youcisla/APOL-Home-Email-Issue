# README: Handling Multiple Email Types for APOL and PeopleSoft

## Table of Contents
1. [Overview](#overview)
2. [The Issue](#the-issue)
3. [Solution Approach](#solution-approach)
    1. [Extracting Email Data](#extracting-email-data)
    2. [Identifying the Preferred Email](#identifying-the-preferred-email)
    3. [Working with Multiple Email Types](#working-with-multiple-email-types)
4. [Implementation Guide](#implementation-guide)
    1. [JavaScript Code for Extracting Emails](#javascript-code-for-extracting-emails)
    2. [Handling Preferred Email](#handling-preferred-email)
    3. [Usage Example](#usage-example)
5. [Future Considerations](#future-considerations)
6. [Conclusion](#conclusion)

---

## Overview

In systems like APOL and PeopleSoft, email addresses for users are often stored in multiple formats (HOME, BUSINESS, WORK, etc.). However, the login process or data sync may only check against one type of email (e.g., HOME). This can cause issues when users change their email addresses or use different email types for different contexts.

The goal is to collect and handle **multiple types of email addresses** without removing any, allowing the system to reference them all, particularly during login, sync, or data validation processes.

---

## The Issue

### Scenario:
- You encountered an issue where the system checks only against the **HOME** email type in PeopleSoft, but users might log in or use email addresses marked as **BUSINESS** or **WORK**.
- The system was filtering out non-HOME emails, which caused errors in login and data sync when users expected other email types to be accepted.
  
### Problem:
- The current implementation used in PeopleSoft and APOL removes email nodes that aren’t of type HOME, which limits flexibility in how users interact with the system.
- This results in misrepresentation of data, where updates to emails (e.g., when a user updates their BUSINESS email) are not properly handled.

---

## Solution Approach

### Extracting Email Data

Rather than removing non-HOME email addresses from the XML structure or form data, all email addresses (including HOME, WORK, BUSINESS, etc.) should be **retained** and processed as part of the user’s profile. This ensures:
1. **Flexibility**: Multiple email addresses are recognized and usable in the system.
2. **Integrity**: No loss of data as all emails are kept in the system for reference or login.

### Identifying the Preferred Email

In addition to keeping all email addresses, the system also marks one email as "preferred" using a flag (`PREF_EMAIL_FLAG`). This allows you to:
- Decide which email should be used by default when multiple email types are present.
- Maintain consistency when dealing with login verification or external data sync.

### Working with Multiple Email Types

1. **Extract all emails**: Instead of filtering for just the HOME email, the system should retain all email addresses in the XML structure or data table.
2. **Check for preferred email**: Use the `PREF_EMAIL_FLAG` to determine which email should be prioritized when multiple options are available.

---

## Implementation Guide

### JavaScript Code for Extracting Emails

The following JavaScript code can be used to extract all email addresses from the system's HTML table (as seen in PeopleSoft's UI). It captures:
- **Email type** (e.g., HOME, BUSINESS)
- **Email address**
- **Preferred email** flag

```javascript
const rows = document.querySelectorAll('#table_edp-2500-oep-avira_edp-2500-oep-avira-personal-data-emails-email-preferred tbody tr');

let emailAddresses = [];

rows.forEach((row) => {
    const emailType = row.querySelector('input[data-map-key="E_ADDR_TYPE"]').value;
    const emailAddress = row.querySelector('input[type="email"]').value;
    const preferredRadio = row.querySelector('input[type="radio"][value="Y"]');
    const isPreferred = preferredRadio ? preferredRadio.checked : false;

    emailAddresses.push({
        emailType: emailType,
        emailAddress: emailAddress,
        isPreferred: isPreferred
    });
});

console.log(emailAddresses);
```

### Handling Preferred Email

Once you have extracted the email data, you can process the preferred email as follows:

```javascript
const preferredEmail = emailAddresses.find(email => email.isPreferred)?.emailAddress;
console.log("Preferred Email: ", preferredEmail);
```

This code will find and return the email marked as preferred.

### Usage Example

If you need to verify a login email against multiple email types:

```javascript
const loginEmail = 'user@example.com';
const isEmailValid = emailAddresses.some(email => email.emailAddress === loginEmail);

if (isEmailValid) {
    console.log("Login successful");
} else {
    console.log("Email not found");
}
```

This will check the login email against all stored email addresses and determine if the login should succeed.

---

## Future Considerations

1. **Syncing Data Between APOL and PeopleSoft**:
   - Ensure that the changes made in PeopleSoft (such as an email update) are reflected in APOL. This can be done via batch jobs or an API sync mechanism that handles updates in real-time.

2. **Handling Multiple Preferred Emails**:
   - It’s possible for multiple email types to be marked as "preferred" in complex cases. The system should include validation rules to handle this gracefully (e.g., prioritizing BUSINESS over HOME or showing an error if two emails are marked preferred).

3. **Customization Based on User Roles**:
   - Depending on the user role (e.g., student, alumni, staff), the system may prefer different types of emails. A rule engine can be added to dynamically assign preferences based on roles or context.

---

## Conclusion

By adjusting the way email addresses are handled (i.e., keeping all types and respecting the "preferred" flag), the system becomes more flexible and robust. This allows APOL and PeopleSoft to provide a seamless user experience where users can interact with the system using different email types, without losing data integrity or causing login errors.

This README serves as a comprehensive guide for developers or administrators looking to understand and implement a multi-email approach in systems like APOL and PeopleSoft.
