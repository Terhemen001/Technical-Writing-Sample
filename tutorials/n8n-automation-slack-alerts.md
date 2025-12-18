
# Automating Slack Notifications with n8n

## Overview
This tutorial walks through building an automated workflow in **n8n**
that sends a Slack notification whenever a new row is added to a Google Sheet.

## Prerequisites
- n8n account (cloud or self-hosted)
- Google account with access to Google Sheets
- Slack workspace with permission to install apps

## Use Case
Teams often rely on Google Sheets for tracking data.
This automation ensures real-time Slack alerts whenever new data is added.

## Step 1: Create a Google Sheets Trigger
1. Add the **Google Sheets Trigger** node.
2. Authenticate using OAuth credentials.
3. Configure the trigger to watch for new rows.

## Step 2: Configure the Slack Node
1. Add the **Slack** node.
2. Select **Send Message** as the operation.
3. Choose a channel and customize the message using expressions.

Example message:
```text
New row added: {{$json["Name"]}} – {{$json["Status"]}}

## Step 3: Test and Activate

1. Connect all nodes in the workflow.
2. Save and activate the workflow.
3. Add a new row to the connected Google Sheet to verify the automation behavior.

If configured correctly, a Slack notification will be sent automatically.

## Conclusion

This automation demonstrates how n8n enables low-code integrations
between popular tools while remaining flexible and extensible.

By combining event triggers with third-party services, teams can automate
repetitive tasks without writing extensive custom code.
