# Google Sheets Template Setup

## Content Calendar Sheet Structure

Create a new Google Sheet with the following structure for the "Content Calendar" tab:

### Column Headers (Row 1)
| A | B | C | D | E |
|---|---|---|---|---|
| **Date & Time** | **Post Text** | **Media URL** | **Status** | **Facebook Post ID** |

### Sample Data (Rows 2+)
| Date & Time | Post Text | Media URL | Status | Facebook Post ID |
|-------------|-----------|-----------|---------|-------------------|
| 2024-01-15 09:00 | "Exciting news! Our new product is launching today. Check it out!" | https://example.com/product-image.jpg | Scheduled | |
| 2024-01-16 14:30 | "Happy Monday everyone! Hope you're having a productive start to the week." | | Scheduled | |
| 2024-01-17 12:00 | "Behind the scenes: Our team working hard to bring you the best experience!" | https://example.com/team-photo.jpg | Scheduled | |
| 2024-01-18 16:00 | "Customer spotlight: Amazing feedback from @username about our service!" | | Scheduled | |

## Weekly Report Sheet Structure

The "Weekly Report" sheet will be automatically populated, but you can create it with this header structure:

### Column Headers (Row 1)
| A | B | C | D |
|---|---|---|---|
| **Report Section** | **Metric** | **Value** | **Details** |

## Google Sheets Setup Steps

### 1. Create New Google Sheet
1. Go to [Google Sheets](https://sheets.google.com)
2. Click "Blank" to create a new spreadsheet
3. Rename it to "Social Media Content Calendar"

### 2. Set Up Content Calendar Tab
1. Rename the first tab to "Content Calendar"
2. Add the column headers in row 1
3. Format headers with bold text and background color
4. Freeze the first row for easy navigation

### 3. Set Up Weekly Report Tab
1. Click the "+" button to add a new tab
2. Rename it to "Weekly Report"
3. Add the column headers in row 1
4. Format headers appropriately

### 4. Configure Data Validation (Optional)
For the Status column, you can set up data validation:
1. Select column D (Status)
2. Go to Data → Data validation
3. Set criteria to "List of items"
4. Enter: `Scheduled,Posted,Draft`
5. Check "Show validation help text"

### 5. Format Date Column
1. Select column A (Date & Time)
2. Go to Format → Number → Date time
3. Choose format: "12/31/2023 23:59"

### 6. Set Up Conditional Formatting (Optional)
For the Status column:
1. Select column D
2. Go to Format → Conditional formatting
3. Add rule: "Text is exactly 'Scheduled'" → Blue background
4. Add rule: "Text is exactly 'Posted'" → Green background

## Important Notes

### Date Format
- Use format: `YYYY-MM-DD HH:MM`
- Example: `2024-01-15 09:00`
- This ensures proper parsing by the n8n workflow

### Status Values
- **Scheduled**: Post is queued for publishing
- **Posted**: Post has been published to Facebook
- **Draft**: Post is being prepared (optional)

### Media URL
- Leave empty if no media
- Use direct links to images/videos
- Ensure URLs are publicly accessible

### Facebook Post ID
- This column will be automatically populated
- Don't manually edit this column
- Used for tracking and analytics

## Sharing and Permissions

### 1. Share with n8n Service Account
1. Click "Share" button in top right
2. Add your service account email (from Google Cloud Console)
3. Give "Editor" permissions
4. Uncheck "Notify people" to avoid spam

### 2. Share with Team Members
1. Add team member emails
2. Give appropriate permissions:
   - **Managers**: Editor access
   - **Content Creators**: Editor access
   - **Viewers**: Viewer access

### 3. Set Up Notifications (Optional)
1. Go to Tools → Notification rules
2. Set up notifications for:
   - When someone edits the sheet
   - When posts are scheduled
   - When status changes occur

## Mobile Access

### Google Sheets App
1. Download Google Sheets app on mobile
2. Sign in with your Google account
3. Access your content calendar on the go
4. Make quick edits and updates

### Mobile-Friendly Formatting
1. Ensure columns are wide enough for mobile viewing
2. Use clear, concise text in post content
3. Test mobile view before finalizing

## Workflow Integration

### Document ID
1. Copy the document ID from the URL
2. Format: `https://docs.google.com/spreadsheets/d/[DOCUMENT_ID]/edit`
3. Use this ID in your n8n environment variables

### Sheet Names
- Must match exactly: "Content Calendar" and "Weekly Report"
- Case-sensitive
- No extra spaces or special characters

### Column References
- Workflow references columns by letter (A, B, C, D, E)
- Don't change column order after setting up the workflow
- If you need to add columns, update the workflow accordingly

## Pro Tips

1. **Use Templates**: Create a template sheet for different content types
2. **Color Coding**: Use different colors for different post categories
3. **Tags**: Add a column for content tags or categories
4. **Approval Workflow**: Add approval status for content review
5. **Backup**: Regularly export your sheet as backup
6. **Version History**: Use Google Sheets version history to track changes

## Sample Advanced Structure

For more advanced users, you can expand the structure:

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| **Date & Time** | **Post Text** | **Media URL** | **Status** | **Facebook Post ID** | **Category** | **Approved By** | **Notes** |
| 2024-01-15 09:00 | "New product launch!" | image.jpg | Scheduled | | Product | John | Include CTA |
| 2024-01-16 14:30 | "Team photo" | team.jpg | Scheduled | | Culture | Sarah | Tag team members |

This advanced structure allows for better content organization and approval workflows.
