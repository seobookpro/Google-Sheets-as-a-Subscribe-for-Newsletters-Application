# Google Sheets as a Subscribe Newsletters Application
In today’s digital world, having a simple and effective newsletter subscription system is essential for businesses and individuals looking to engage their audience.

## Use Google Sheets as a Subscribe for Newsletters Application (Code Sample)

In today’s digital world, having a simple and effective newsletter subscription system is essential for businesses and individuals looking to engage their audience. Many popular platforms offer subscription tools, but if you're looking for a cost-effective, customizable solution, Google Sheets can serve as a powerful backend for your newsletter subscription needs. This article walks you through setting up a subscription application using Google Sheets, complete with a code sample to get started.

## Why Use Google Sheets?
Google Sheets is a versatile tool that provides several advantages:

- Free and Accessible: No additional costs, accessible from anywhere.
- Integration-Friendly: Works well with other Google Workspace tools and APIs.
- Customizable: Tailor the solution to meet your specific needs.
- Secure and Scalable: Offers robust security and real-time collaboration.
- Whether you're running a personal blog, a small business, or a side project, Google Sheets can be a practical alternative for managing your subscribers.

# Step-by-Step Guide to Setting Up a Newsletter Subscription System
## 1. Create Your Google Sheet

Start by creating a new Google Sheet. Label the columns to store subscriber details such as:

- Email Address	
- First Name Last Name	
- Website	
- Accept Terms and Privacy	
- Timestamp

### Example Google Sheet - https://docs.google.com/spreadsheets/d/1JN1osfZ4pMBZK0Jm_5tCS4rM8-IS2_4Mib8DwrCjLXw/edit?usp=sharing

## 2. Set Up a Google Apps Script

Google Apps Script allows you to automate data entry into your Google Sheet. Here’s how you can set it up:

1. Open the Google Sheet.
2. Navigate to Extensions > Apps Script.
3. Paste the following code:

```
function doGet(e) {
  var spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  var usersDatabaseSheet = spreadsheet.getSheetByName('Users Database');
  var emailContentSheet = spreadsheet.getSheetByName('Email Content');
  var action = e.parameter.action;

  if (!usersDatabaseSheet) {
    return ContentService.createTextOutput(JSON.stringify({ success: false, message: 'Users Database sheet not found.' }))
      .setMimeType(ContentService.MimeType.JSON);
  }

if (action === 'register_user') {
  var email = e.parameter.email ? e.parameter.email.trim().toLowerCase() : '';
  var userFirstLastName = e.parameter.userFirstLastName ? e.parameter.userFirstLastName.trim() : '';
  var userWebsite = e.parameter.userWebsite ? e.parameter.userWebsite.trim().toLowerCase() : '';
  var termsPrivacy = e.parameter.termsPrivacy ? e.parameter.termsPrivacy.trim() : '';

  if (!email) {
    return ContentService.createTextOutput(JSON.stringify({ success: false, message: 'Invalid email.' }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  var lastRow = usersDatabaseSheet.getLastRow();
  var numRows = lastRow > 1 ? lastRow - 1 : 0;
  var exists = false;

  if (numRows > 0) {
    var data = usersDatabaseSheet.getRange(2, 1, numRows, 4).getValues(); // Get all 4 relevant columns
    for (var i = 0; i < data.length; i++) {
      if (
        data[i][0].toLowerCase() === email || // Check email
        data[i][1].toLowerCase() === userFirstLastName || // Check first & last name
        data[i][2].toLowerCase() === userWebsite || // Check website
        data[i][3].toLowerCase() === termsPrivacy // Check website
      ) {
        exists = true;
        break;
      }
    }
  }

  if (exists) {
    return ContentService.createTextOutput(JSON.stringify({ success: false, message: 'User already exists.' }))
      .setMimeType(ContentService.MimeType.JSON);
  } else {
    usersDatabaseSheet.appendRow([email, userFirstLastName, userWebsite, termsPrivacy, new Date()]);
    return ContentService.createTextOutput(JSON.stringify({ success: true, message: 'User registered successfully.' }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

if (action === 'check_user') {
  const field = e.parameter.field; // Field to check (email, userFirstLastName, or userWebsite)
  const value = e.parameter.value ? e.parameter.value.trim().toLowerCase() : '';

  if (!field || !value) {
    return ContentService.createTextOutput(JSON.stringify({ success: false, exists: false }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  const lastRow = usersDatabaseSheet.getLastRow();
  const numRows = lastRow > 1 ? lastRow - 1 : 0;
  let exists = false;

  if (numRows > 0) {
    const columnIndex = {
      email: 0,
      userFirstLastName: 1,
      userWebsite: 2,
    }[field]; // Determine the column index to check

    if (columnIndex !== undefined) {
      const data = usersDatabaseSheet.getRange(2, columnIndex + 1, numRows, 1).getValues(); // Get values from the specific column
      for (let i = 0; i < data.length; i++) {
        if (data[i][0].toLowerCase() === value) {
          exists = true;
          break;
        }
      }
    }
  }

  return ContentService.createTextOutput(JSON.stringify({ success: true, exists }))
    .setMimeType(ContentService.MimeType.JSON);
}


if (action === 'send_email') {
  if (!emailContentSheet) {
    return ContentService.createTextOutput(JSON.stringify({ success: false, message: 'Email Content sheet not found.' }))
      .setMimeType(ContentService.MimeType.JSON);
  }
  
    const emailBodyTemplates = emailContentSheet.getRange('C2:C').getValues().flat().filter(String);

    if (emailBodyTemplates.length === 0) {
    SpreadsheetApp.getUi().alert('No email templates found in C2:C!');
    return;
    }

  var emailSubject = emailContentSheet.getRange('A2').getValue(); // Email subject in A2
  var emailSubjectBrand = emailContentSheet.getRange('B2').getValue(); // Email subject Brand in B2
  // var emailBodyTemplate = emailContentSheet.getRange('C2').getValue(); // Email body template in C2
  const emailBody = emailBodyTemplates.join('\n\n'); // Concatenate templates with line breaks

  if (!emailSubject || !emailBody) {
    return ContentService.createTextOutput(JSON.stringify({ success: false, message: 'Email subject or body is empty.' }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  var emails = usersDatabaseSheet.getRange(2, 1, usersDatabaseSheet.getLastRow() - 1, 3).getValues(); // Get user details (Email, First Name Last Name, Website)

  if (emails.length > 0) {
    try {
      for (var i = 0; i < emails.length; i++) {
        var email = emails[i][0]; // Email address
        var userFirstLastName = emails[i][1]; // First Name Last Name
        var userWebsite = emails[i][2]; // Website Address

        // Replace placeholders in the email body
        var personalizedBody = emailBody
          .replace(`[Subscriber Email Address]`, email)
          .replace(`[First Name Last Name]`, userFirstLastName || 'Subscriber') // Default to 'Subscriber' if name is empty
          .replace(`[User Website Address]`, userWebsite || 'N/A'); // Default to 'N/A' if website is empty

        // Send the email
        GmailApp.sendEmail(email, emailSubject + emailSubjectBrand, '', {
          htmlBody: personalizedBody
        });
      }

      return ContentService.createTextOutput(JSON.stringify({ success: true, message: 'Emails sent successfully.' }))
        .setMimeType(ContentService.MimeType.JSON);
    } catch (error) {
      return ContentService.createTextOutput(JSON.stringify({ success: false, message: 'Error sending emails: ' + error.message }))
        .setMimeType(ContentService.MimeType.JSON);
    }
  } else {
    return ContentService.createTextOutput(JSON.stringify({ success: false, message: 'No users to email.' }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

  return ContentService.createTextOutput(JSON.stringify({ success: false, message: 'Invalid action.' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

   
