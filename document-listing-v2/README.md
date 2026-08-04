# document-listing-v-2

## Summary

`document-listing-v-2` is a SharePoint Framework (SPFx) web part for listing documents from a SharePoint library and optionally logging access requests in a SharePoint list. The web part supports category and sub-category grouping, search, pinned documents, and request access workflows.

## Used SharePoint Framework Version

![version](https://img.shields.io/badge/version-1.22.0-green.svg)

## Applies to

- [SharePoint Framework](https://aka.ms/spfx)
- [Microsoft 365 tenant](https://docs.microsoft.com/sharepoint/dev/spfx/set-up-your-developer-tenant)

> Get your own free development tenant by subscribing to [Microsoft 365 developer program](http://aka.ms/o365devprogram)

## Prerequisites

- Node.js `22.x` (as required by this repo)
- `npm` installed
- A SharePoint site with a document library and optional request access list
- A SharePoint workbench or page where the SPFx web part can be deployed

## Solution

| Solution | Author(s) |
| --- | --- |
| `document-listing-v-2` | `vishpowerlabs` |

## Overview

This SPFx solution renders a document listing web part that:

- Loads documents from a selected SharePoint document library
- Groups items by category and sub-category fields
- Supports custom title, description, page size, and header opacity
- Allows users to request access to documents
- Logs access requests to a configurable SharePoint list
- Sends reminders by updating a reminder field in the request list

## Build and Run

From the repository root:

```bash
npm install
npm run start
```

For a production package:

```bash
npm run build
```

## Web Part Configuration

The web part uses the SPFx property pane to configure display settings, source library mappings, and request access behavior.

### Display Settings

- `Web Part Title`: visible title text for the web part
- `Title Font Size`: `16px`, `20px`, `24px`, `32px`
- `Web Part Description`: optional description shown below the title
- `Description Font Size`: `12px`, `14px`, `16px`, `18px`
- `Header Opacity`: opacity value between `0` and `1`

### Document Library Settings

- `Select Site`: choose the site containing your document library
- `Source Document Library`: choose the library that contains documents
- `Category Field`: field used to group documents into categories
- `Sub-Category Field`: field used to group documents into sub-categories
- `Description Field`: field used for item descriptions
- `Items per page`: page size for listing results
- `Pinned Column`: optional field used to pin documents to the top
- `Title Column`: optional field used as the document title instead of the default Title field

### Request Access Settings

- `Show Request Access Column`: toggle request access feature on/off
- `Select Request Site`: choose the site containing your requests list
- `Requests List`: choose the SharePoint list used to store access requests
- `Email Field (in Request List)`: field storing the requester's email
- `File ID Field`: field storing the file identifier
- `Request ID Field`: field storing the generated request ID
- `Request Date Field`: field storing the request date
- `Reminder Field`: field used to mark reminders when users ask for a follow-up request
- `Already Requested Message`: text shown when the user has already requested access
- `Reminder Sent Message`: toast text shown after sending a reminder

## Required SharePoint Fields

### Source Document Library Fields

The web part requires a document library and the following field mappings:

- `Category Field`: defaults to `Category`
- `SubCategory Field`: defaults to `SubCategory`
- `Description Field`: defaults to `Description`
- `Title Column`: optional; if not provided the web part uses the library item `Title` or document file name
- `Pinned Column`: optional; supports boolean values or `Yes` / `1` / `true`

> If no field is selected, the web part defaults to `Category`, `SubCategory`, and `Description` internal names.

### Request Access List Fields

The request access workflow logs items in a separate SharePoint list and requires these field mappings:

- `Email Field`: defaults to `Email`
- `File ID Field`: defaults to `FileID`
- `Request ID Field`: defaults to `RequestID`
- `Request Date Field`: defaults to `RequestDate`
- `Reminder Field`: defaults to `Reminder`

The web part writes a new item to the requests list when a document access request is created, and updates the reminder field for follow-up reminders.

## Notes

- List and field dropdowns are populated dynamically from the selected site.
- The request access feature only works when `Requests List` and required request fields are configured.
- The request list item title is generated automatically as `Request: <email> - <fileId>`.

## References

- [Getting started with SharePoint Framework](https://docs.microsoft.com/sharepoint/dev/spfx/set-up-your-developer-tenant)
- [Heft Documentation](https://heft.rushstack.io/)
- [SPFx Property Pane Controls](https://pnp.github.io/spfx-controls-react/)
