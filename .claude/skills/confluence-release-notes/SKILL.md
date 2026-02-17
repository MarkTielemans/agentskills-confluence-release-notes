---
name: confluence-release-notes
description: Create a Confluence release notes page from a local release notes file. Use when the user wants to publish release notes to Confluence.
argument-hint: <release-notes-file> <space-key>
allowed-tools: Read, Bash, AskUserQuestion
---

# Release Notes Skill

Create a Confluence release notes page from a local file.

## Arguments

- `$0` — Path to the release notes input file (required)
- `$1` — Confluence space key (required)

## Steps

### 1. Read the input file

Read the input file at `$0`.

The input file has this format:
```
Release X.X, Mon YYYY
Feature title 1
Description of feature 1.

Feature title 2
Description of feature 2.
```

Parse: version number, release month/year, and all feature title + description pairs.

### 2. Resolve the space

Use the Atlassian tool `getConfluenceSpaces` with Space key `$1` to get the numeric space ID.

### 3. Find the parent page

The parent page ID is `88473601`. Verify it exists by fetching it with `getConfluencePage`.

### 4. Get the author

Use `atlassianUserInfo` to get the current user's account ID and display name for the author mention.

### 5. Ask clarifying questions

Ask the user:
- Confirm the author (current user or someone else)
- ERP API version (if any, or leave empty)

### 6. Determine the page title

Format: `YYYY-MM QN Release Notes`
- Use the month/year from the input file
- Q1 = Jan-Mar, Q2 = Apr-Jun, Q3 = Jul-Sep, Q4 = Oct-Dec

### 7. Generate a unique macro ID

Generate a 64-character hex string for the macroId. Use Bash:
```bash
python3 -c "import secrets; print(secrets.token_hex(32))"
```

### 8. Build the ADF body and create the page

Use `createConfluencePage` with `contentFormat: "adf"`.

CRITICAL requirements for the ADF body:
- The metadata table MUST be wrapped in a `bodiedExtension` with `extensionKey: "details"` (Page Properties macro)
- The `macroParams` MUST include `"id": {"value": "metadata"}` — this is the Content Properties ID that the parent page's Page Properties Report uses to find child pages. Without it, the page won't appear in the report.
- The `macroId` value must be the unique 64-char hex string generated in step 7
- Left column cells are `tableHeader`, right column cells are `tableCell`
- Author field uses a `mention` node: `{"type": "mention", "attrs": {"id": "ACCOUNT_ID", "text": "@Display Name"}}`
- Each feature is a `listItem` with bold title, `hardBreak`, then description text

Here is the exact ADF structure to use (fill in the placeholders):

```json
{
  "type": "doc",
  "content": [
    {
      "type": "bodiedExtension",
      "attrs": {
        "layout": "default",
        "extensionType": "com.atlassian.confluence.macro.core",
        "extensionKey": "details",
        "parameters": {
          "macroParams": {"id": {"value": "metadata"}},
          "macroMetadata": {
            "macroId": {"value": "GENERATED_HEX_STRING"},
            "schemaVersion": {"value": "1"},
            "title": "Page Properties"
          }
        }
      },
      "content": [
        {
          "type": "table",
          "attrs": {"layout": "default", "width": 760},
          "content": [
            {
              "type": "tableRow",
              "content": [
                {"type": "tableHeader", "attrs": {"colspan": 1, "rowspan": 1}, "content": [{"type": "paragraph", "content": [{"text": "Author", "type": "text", "marks": [{"type": "strong"}]}]}]},
                {"type": "tableCell", "attrs": {"colspan": 1, "rowspan": 1}, "content": [{"type": "paragraph", "content": [{"type": "mention", "attrs": {"id": "AUTHOR_ACCOUNT_ID", "text": "@AUTHOR_NAME"}}, {"text": " ", "type": "text"}]}]}
              ]
            },
            {
              "type": "tableRow",
              "content": [
                {"type": "tableHeader", "attrs": {"colspan": 1, "rowspan": 1}, "content": [{"type": "paragraph", "content": [{"text": "Version number", "type": "text", "marks": [{"type": "strong"}]}]}]},
                {"type": "tableCell", "attrs": {"colspan": 1, "rowspan": 1}, "content": [{"type": "paragraph", "content": [{"text": "VERSION_NUMBER", "type": "text"}]}]}
              ]
            },
            {
              "type": "tableRow",
              "content": [
                {"type": "tableHeader", "attrs": {"colspan": 1, "rowspan": 1}, "content": [{"type": "paragraph", "content": [{"text": "ERP API Version", "type": "text", "marks": [{"type": "strong"}]}]}]},
                {"type": "tableCell", "attrs": {"colspan": 1, "rowspan": 1}, "content": [{"type": "paragraph", "content": [{"text": "ERP_VERSION_OR_EMPTY", "type": "text"}]}]}
              ]
            }
          ]
        }
      ]
    },
    {"type": "paragraph"},
    {
      "type": "bulletList",
      "content": [
        {
          "type": "listItem",
          "content": [{"type": "paragraph", "content": [
            {"text": "FEATURE_TITLE", "type": "text", "marks": [{"type": "strong"}]},
            {"type": "hardBreak"},
            {"text": "FEATURE_DESCRIPTION", "type": "text"}
          ]}]
        }
      ]
    },
    {"type": "paragraph"}
  ],
  "version": 1
}
```

For an empty ERP API Version, use an empty paragraph: `{"type": "paragraph"}` (no content array).

Repeat the `listItem` block for each feature from the input file.

### 9. Report the result

Show the user the page URL from the `_links.webui` field in the response (prepend the tenant URL of format `https://<tenant>.atlassian.net/wiki`).