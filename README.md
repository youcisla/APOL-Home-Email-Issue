# README: Email Alignment Issue Between APOL and PeopleSoft

## Overview

There’s a misalignment between the email addresses stored in **APOL** and **PeopleSoft**. The core issue is that **PeopleSoft’s HOME email** keeps changing because it's shared across multiple platforms (e.g., MyInsead), while **APOL** no longer syncs these changes after some lines of code were commented out to avoid constant updates.

This leads to problems, especially during **login** and when syncing email changes. The goal is to find a way to handle multiple email types across both systems without causing login issues or data inconsistencies.

---

## The Issue

1. **Login Issues**:
   APOL relies on the `SCC_UR_AUTHENTICATE_REQ` web service to authenticate users by verifying their information, including the email address, in PeopleSoft. Since the HOME email in PeopleSoft changes frequently, and APOL no longer syncs these updates, the systems fall out of alignment, leading to errors like HTTP 500 during login attempts.

2. **Email Sync Problems**:
   Initially, APOL synced the first email in the array from PeopleSoft. However, because we didn’t have a process to notify applicants when their email was changed, syncing was disabled. Now, APOL no longer updates its records when PeopleSoft’s HOME email changes, creating further inconsistencies.

---

## Objective

The goal is to allow APOL to handle **multiple email types** (not just the HOME email) and reduce dependency on the changing HOME email in PeopleSoft. This way, login and communication can be more flexible and accurate.

---

## Solution Approach

### 1. Sync and Store All Email Types

Instead of syncing only the HOME email, we need to **capture and store all email types** provided by PeopleSoft. This includes:
- **HOME**
- **BUSINESS**
- **CAMPUS**
- **INSEAD LOGIN**
- **LINKEDIN**, etc.

By storing all these types, we allow APOL to verify the login using any valid email address and avoid issues caused by PeopleSoft changing the HOME email.

### 2. Use Preferred Email for Login

Each user has a **preferred email** stored in APOL (identified by `PREF_EMAIL_FLAG`). During login, APOL should prioritize checking the preferred email first and fallback on other email types (if necessary). This ensures that even if PeopleSoft’s HOME email changes, the preferred email in APOL can still be used for authentication.

### 3. Re-enable Sync with a Notification

Once we capture and store multiple email types, the sync between PeopleSoft and APOL can be re-enabled. However, when the HOME email changes, users should be notified of the update and given the option to adjust their preferred email if needed.

---

## Implementation Guide

### PHP Code to Extract and Store Emails

In APOL, we can modify the PHP logic to extract all email types from the SOAP response provided by PeopleSoft. Here’s how to do it:

```php
$emailAddressDatas = $this->find($datas, 'emailAddress');
$emailAddressDatas = !isset($emailAddressDatas[0]) ? array($emailAddressDatas) : $emailAddressDatas;

if ($emailAddressDatas) {
    $emailAddresses = new EmailAddresses();

    foreach ($emailAddressDatas as $emailAddressData) {
        $emailAddress = new EmailAddress();
        $emailAddress = $this->populate($emailAddress, $emailAddressData);

        // Check if the email has a preferred flag and set it accordingly
        \is_null($emailAddress->prefEmailFlag) ? $emailAddress->prefEmailFlag = 'N' : $emailAddress->prefEmailFlag;

        // Validate and store the email address
        if ($emailAddress->validate()) {
            $emailAddresses->emailAddress[] = $emailAddress;
        }
    }

    // If only one email is available, mark it as preferred
    if (is_array($emailAddresses->emailAddress) && 1 === \count($emailAddresses->emailAddress)) {
        $emailAddresses->emailAddress[0]->prefEmailFlag = 'Y';
    }

    if ($emailAddresses->validate()) {
        $constituent->emailAddresses[] = $emailAddresses;
    }
}
```

### How It Works:
- **Extract all emails** from the SOAP response. This includes HOME, BUSINESS, and other types.
- **Check the preferred flag** (`PREF_EMAIL_FLAG`) to identify which email is marked as preferred.
- **Validate and store** the email addresses in APOL, ensuring all types are kept.

### PHP Logic for Login Verification

Instead of checking just the HOME email, APOL should validate the login against all stored email addresses:

```php
$loginEmail = 'user@example.com';
$found = false;

foreach ($emailAddresses->emailAddress as $emailAddress) {
    if ($loginEmail === $emailAddress->email) {
        $found = true;
        break;
    }
}

if ($found) {
    // Proceed with login
} else {
    // Handle login failure (email not found)
}
```

---

## Future Considerations

### 1. Notify Users of Email Changes

When PeopleSoft’s HOME email changes, users should receive a notification in APOL informing them of the update. This can be done via email or a notification within the APOL portal. They should also have the option to update their preferred email if they wish.

### 2. Multiple Preferred Emails

In some cases, users might have multiple preferred emails (e.g., one for business and one for personal use). APOL should provide a mechanism to allow users to manage multiple preferred emails and prioritize one based on the context (e.g., business vs. personal communications).

---

## Conclusion

By implementing these changes, APOL will be able to:
- **Handle multiple email types** without depending solely on the HOME email.
- **Avoid login issues** caused by mismatches between PeopleSoft and APOL.
- **Improve user experience** by giving more flexibility with email management and syncing updates across platforms.

This solution ensures that email data remains consistent and up-to-date, avoiding common pitfalls caused by frequent email changes in PeopleSoft.
