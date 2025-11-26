# Business Card Processor

Automate networking follow-ups. Upload a business card photo, extract contact info with AI, save to Google Sheets, and send personalized emails.

## How It Works

1. Upload business card image via Wix form
2. GPT-4o mini extracts contact information
3. Contact saved to Google Sheets
4. Personalized follow-up email sent via Microsoft 365
5. Sheet updated with email status

## Files

| File | Description |
|------|-------------|
| `business_card_processor_wix.json` | n8n workflow (import this) |
| `business_card_database.xlsx` | Google Sheets template |
| `business_card_processor_specs.png` | Full specifications |

## Setup

### 1. Google Sheets

1. Upload `business_card_database.xlsx` to Google Sheets
2. File → Import → Upload → Replace spreadsheet
3. Copy the Sheet ID from the URL (between `/d/` and `/edit`)
4. Update Sheet ID in workflow nodes: Get Email Template, Add Contact to Sheet, Update Email Status

### 2. n8n Workflow

1. Import `business_card_processor_wix.json` into n8n
2. Connect credentials:
   - OpenAI API → Extract Card Info node
   - Google Sheets OAuth2 → 3 sheet nodes
   - Microsoft 365 OAuth2 → Send Follow-up Email node
3. Update webhook URL in your Wix form

### 3. Wix Form

Create a form with these fields:

| Field | Type |
|-------|------|
| Event Name | Dropdown (Event 1, Event 2, Event 3) |
| Event Date | Date picker |
| Business Card Upload | File upload |
| Your Notes | Text area |

Set form action to POST to your n8n webhook URL.

### 4. Email Templates

Edit the Templates tab in your Google Sheet:

| Column | Description |
|--------|-------------|
| Event Name | Must match dropdown options |
| Event Date | Date of the event |
| Subject Line | Email subject (supports variables) |
| Email Body | Email content (supports variables) |

**Available variables:**
- `{{first_name}}` - Contact's first name
- `{{last_name}}` - Contact's last name
- `{{company}}` - Contact's company
- `{{title}}` - Contact's job title
- `{{event_name}}` - Event name from form
- `{{event_date}}` - Event date from form
- `{{custom_paragraph}}` - Generated from Your Notes
- `{{your_signature}}` - Your signature (edit in workflow)

## Workflow Nodes

```
Wix Webhook
    ├── Respond to Wix (immediate confirmation)
    └── Download Image from Wix
            └── Convert to Base64
                    └── Extract Card Info (GPT-4o mini)
                            └── Parse Card Data
                                    └── Get Email Template
                                            └── Build Email Content
                                                    └── Add Contact to Sheet
                                                            └── Has Email Address?
                                                                    ├── Yes → Send Email → Update Status
                                                                    └── No → (end)
```

## Extracted Fields

From business card:
- First Name, Last Name
- Email, Phone
- Company, Title
- Address, Website
- LinkedIn, Other Social
- Card Notes (handwritten text)

From form:
- Event Name, Event Date
- Your Notes

Auto-generated:
- Date Added
- Email Sent (Yes/No)
- Date Contacted

## Credentials Required

| Service | Purpose |
|---------|---------|
| OpenAI API | GPT-4o mini vision extraction |
| Google Sheets OAuth2 | Database read/write |
| Microsoft 365 OAuth2 | Send emails via Outlook |

## Troubleshooting

**Webhook not receiving data:**
- Verify Wix form action is set to POST
- Check webhook URL matches n8n (Production URL when active)

**Image not downloading:**
- Check field name in "Download Image from Wix" node matches Wix payload
- Test webhook and inspect actual field names Wix sends

**Email not sending:**
- Verify Microsoft 365 credentials are connected
- Check contact has valid email address

**Template not found:**
- Ensure Event Name in form dropdown matches exactly with Templates tab

## Version

v2.0.0 - Wix webhook integration

## License

MIT
