# Workflow Context

**Name:** Demo - User Details Form - Google Doc (Mock)
**ID:** 5IApa6fRHNRgwAOu
**Instance:** vishalmishra.app.n8n.cloud
**Project ID:** f256nwX37BEaIkA2
**Status:** Draft
**Created/Updated:** 2026-05-20

## Description
A demo workflow that presents an n8n-hosted web form collecting four required fields: Full Name, Email, Department, and Role. On submission, the data flows into a Code node that simulates appending the row to a Google Doc, returning a structured mock response with documentId, documentTitle, appendedContent, status, and message fields. No external credentials are required.

## Trigger
n8n Form Trigger — a hosted web form is displayed to the user. The workflow executes when the user submits the form.

## Nodes & Services
- User Details Form (n8n-nodes-base.formTrigger) — presents a web form with four required fields: Full Name (text), Email (email), Department (text), Role (text)
- Save to Google Doc (Mock) (n8n-nodes-base.code) — mutates the incoming item to append mock Google Doc response fields: mock, documentId, documentTitle, appendedContent, status, message

## Mock / Production Notes
The Code node mocks a Google Docs append operation. To go live, replace it with a real Google Docs node (n8n-nodes-base.googleDocs) configured with Google OAuth2 credentials and mapped to the same four form fields.
