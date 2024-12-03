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
## Step 1 - Create Your Google Sheet

Start by creating a new Google Sheet. Label the columns to store subscriber details such as:

- Email Address	
- First Name Last Name	
- Website	
- Accept Terms and Privacy	
- Timestamp

### Example Google Sheet - https://docs.google.com/spreadsheets/d/1JN1osfZ4pMBZK0Jm_5tCS4rM8-IS2_4Mib8DwrCjLXw/edit?usp=sharing

## 2. Set Up a Google Apps Script

Google Apps Script allows you to automate data entry into your Google Sheet. Here’s how you can set it up:

1. Open the Google Sheet.[^1]
2. Navigate to Extensions > Apps Script.[^2]
3. Paste the following code:[^3]

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
This script listens for POST requests, extracts the subscriber’s email address, first and last name, website url and aceptance for the Terms and Conditions and Pricacy
When the Website Form is Submitted the Form Input Values add the user form values as data to your Google Sheet.

## 3. Deploy the Script as a Web App

### Steps to make the AppScript accessible 

Deploy your Google Sheets AppScript as WebApp to use custom HTML Form at your External Website as Newsletter Subscribe Form:

1. Click Deploy > New Deployment in the Apps Script editor
   - Choose Web App as the deployment type
2. Set the following configurations:
   - Description: Newsletter Subscription API
   - Execute As: Me
   - Who Has Access: Anyone
   - Click Deploy and copy the generated URL.
  


## Benefits of This Setup

### 1. Real-Time Data Management
As subscribers sign up, their data is instantly added to your Google Sheet, which you can review or export at any time.

### 2. Customizable Automation

Using Google Apps Script, you can:

- Send automated confirmation emails.
- Trigger notifications when a new subscriber joins.
- Sync data with other tools using APIs like Zapier.

### 3. No Third-Party Dependencies
Unlike subscription services like Mailchimp or ConvertKit, this setup gives you complete control over your subscriber data.

## Things to Keep in Mind

### 1. API Rate Limits
Google Apps Script has execution limits, so this solution is ideal for small to medium-scale applications.

### 2. Email Verification
You can enhance the setup by integrating email verification tools to ensure data accuracy.

### 3. Data Security
Ensure proper access control to your Google Sheet. Only authorized users should have edit access.

## Conclusion
Using Google Sheets as a backend for your newsletter subscription system is a simple, cost-effective, and scalable solution. With just a few steps and some basic coding, you can have a fully functional application ready to capture subscriber details and grow your audience.

This approach not only saves money but also gives you the flexibility to adapt the system to your unique requirements. Whether you're a developer, a business owner, or a blogger, Google Sheets and Apps Script can help you manage your subscribers efficiently.
---
Ready to implement your subscription system? Try out the code sample below and start building a seamless experience for your audience today!

# Build a Subscription Form with Real-Time Validation Using HTML and JavaScript

Creating an interactive subscription form for newsletters can seem like a daunting task, but with HTML and JavaScript, you can create a functional, user-friendly, and dynamic form. This guide provides a complete implementation using the provided code samples.

## The Subscription Form
Here’s the HTML structure for the subscription form. This form captures essential details like the subscriber's name, email, and website, along with terms and privacy acceptance.

### HTML Code Sample

```
          <button id="chat-bubble" class="chat-bubble"><span class="material-symbols-outlined">chat</span></button>
          <!-- Chat Window -->
          <div id="chat-window" class="chat-window" style="display:none;">

  <div class="chat-header">
      <img src="https://seobookpro.com/wp-content/uploads/2023/11/seo-book-pro-logo.png" class="chat-brand-logo" alt="SEO Book Pro">
      <button id="close-chat" class="close-chat">
<span class="material-symbols-outlined">close</span>
</button>
    </div>

            <div class="chat-header-form">



<div id="chat-body">
        <form class="row needs-validation" id="custom-form" novalidate>
          <div class="col-user-first-last-name-website">
                <!-- Email Address -->
                <div class="col">
                  <label for="firstNameLastName" class="form-label">First and Last Name</label>
                  <input type="text" class="form-control" id="firstNameLastName" required>
                  <div class="invalid-feedback">Valid First and Last Name is Required</div>
                  <div class="invalid-feedback-exist">First and Last Name already Exist</div>
                </div>
                <!-- Email Address -->
                <div class="col">
                  <label for="yourWebsite" class="form-label">Your Website</label>
                  <input type="text" class="form-control" id="yourWebsite" required>
                  <div class="invalid-feedback">Valid Website is Required</div>
                  <div class="invalid-feedback-exist">Website already Exist</div>
                </div>
          </div>

          <div class="col-user-email-acceptance">
                <!-- Email Address -->
                <div class="col">
                  <label for="emailAddress" class="form-label">Email Address</label>
                  <input type="email" class="form-control" id="emailAddress" required>
                  <div class="invalid-feedback">Valid Email Address is Required</div>
                  <div class="invalid-feedback-exist">Email Address already Exist</div>
                </div>
                <!-- Acceptance Terms and Privacy -->
                <div class="col">
                  <label for="acceptTermsPrivacy" class="form-label">I agree to the <a href="/terms/" target="_blank" title="Terms and Conditions | Brand Name">Terms and Conditions</a> and <a href="/privacy/" target="_blank" title="Privacy Policy | Brand Name">Privacy Policy</a> of the Brand Name website.</label>
                  <input type="checkbox" class="form-control" id="acceptTermsPrivacy" required>
                  <div class="invalid-feedback">Please Accept the Terms and Conditions and Privacy Policy of the Website!</div>
                </div>
          </div>
                <!-- Buttons: Submit and Clear -->
                <div class="col-md-12 pt-2 mt-2 pb-2 mb-2">
                  <button class="btn-main" role="button" type="submit">Subscribe</button>
                  <button id="clearForm" class="btn-dark" role="button" type="button">Clear</button>
                </div>
              </form>
              <div id="response-message" class="response-message-subscribe" style="display: none;"></div>
              <div id="loader" class="loader-subscribe" style="display: none;">
                <div class="spinner-subscribe"></div>
              </div>
            </div>
            </div>

          </div>

```

