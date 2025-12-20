---
title: "Google Form - Response Emailer"
meta_title: "Google Form Response Email Automation"
description: "Automatically send email notifications when someone submits a Google Form using Apps Script"
date: 2023-07-25T00:00:00Z
image: "/images/image-placeholder.png"
categories: ["How To"]
author: "Chris"
tags: ["google-apps-script", "automation", "google-forms"]
draft: false
---

This tutorial demonstrates how to leverage Google Apps Script to automatically send email notifications whenever someone submits a Google Form.

## Setting Up the Form

Begin by creating a basic feedback form with fields for:

- Customer name
- Phone number
- Email address
- Feedback

Important: Disable domain restrictions in form settings to allow external submissions.

## Script Development

The core implementation involves:

1. Accessing the Script Editor from the form's menu
2. Creating a `Code.gs` file with an `onFormSubmit()` function
3. Adding an HTML template file for email formatting

## Code Structure

The script uses two main variable sections:

**emailHeader:** Contains recipient email, sender name, and subject line

**emailBody:** Includes image URL, message text, button details, and form source link

The `onFormSubmit()` function captures form responses, processes them, and sends formatted emails via Gmail using the HtmlService template.

## Configuration Requirements

1. Customize variables at the script's top
2. Establish a trigger in the script editor to activate the automation on form submissions
3. Authorize the script with your Google Account when prompted

## Testing

Test your automation by:

1. Submitting sample data through the live form preview
2. Verifying receipt of the formatted notification email

This beginner-friendly approach integrates seamlessly with existing Google Workspace environments.
