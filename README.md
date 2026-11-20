# Trimble Connect Element Status Sharing - v0.1

This is a **deployable 3D viewer extension**, not an IFC authoring tool.

## Capabilities
- Read actual selection from Trimble Connect 3D viewer.
- Resolve selected runtime IDs to external model object IDs (IFC GUIDs where available).
- Apply Not Started / Started / Completed / Issue and coatings completed 0..7.
- Timestamped per-element comments with CSV export/import (CommentsJSON column).
- CSV import/export; records keyed by project ID + model version ID + object GUID.
- Select tracked table rows back in 3D (requires matching model version loaded).
- Optional shared-status server with basic API-key protection.

## Deploy
1. Host `index.html`, `app.js`, `style.css`, `icon.png`, and `manifest.json` at an HTTPS static host (e.g. GitHub Pages). No npm build is required.
2. Edit the two `https://YOUR-HOST.example.com/...` URLs in `manifest.json` to your actual HTTPS host URLs.
3. Ensure the manifest is accessible cross-origin (CORS). In Trimble Connect for Browser, go to Project Settings > Apps & Capabilities > Add Custom and enter the HTTPS URL of `manifest.json` (admin permission needed).
4. Open the project 3D viewer, load models, and open Element Status Sharing. Select objects in 3D, press Read viewer selection, choose status and Set status.
5. CSV export works without a server. Browser-local statuses are NOT shared between users.

## Optional team sharing server
1. Install Node.js 18+ on a trusted server.
2. Set `STATUS_API_KEY` to a long random secret and `ALLOWED_ORIGIN` to the **exact** HTTPS origin serving the extension (e.g. `https://example.github.io`). Set `PORT` and `DATA_FILE` optionally.
3. Run `node server.js` and publish it behind a trusted HTTPS reverse proxy. Do not expose it without HTTPS. Restrict access via corporate identity proxy/VPN where possible.
4. Enter the service HTTPS URL and access key in the extension. Press Save settings. All users must point to the same service to share statuses.
5. Use Load shared statuses to refresh other users' changes. Last writer wins. The service uses a JSON file for pilot use only, not enterprise concurrency/audit/identity controls.

## Important boundaries
- Status is stored in local browser storage or in the optional separate service. **It does not write Trimble Connect Property Set Service properties or modify IFC files.** True editable Trimble property sets need additional Trimble API access, schema/library setup, and authorization.
- The 3D selection is real, but live selection event names may differ by viewer version; the manual **Read viewer selection** button is reliable fallback.
- The property filter is a lightweight substring check against the returned property payload for the **currently selected objects**, not a complete property index. For large models, narrow selection first.
- A model version change can change model IDs; GUID reconciliation is needed for cross-version tracking.
- Backend security is suitable for a controlled pilot only; a production service needs per-user authentication, authorization, logging, database transactions and backups.
- The extension requires actual Trimble Connect browser context and cannot be end-to-end tested outside a project.

## Reference
https://developer.trimble.com/docs/connect/tools/api/workspace
https://components.connect.trimble.com/trimble-connect-workspace-api/interfaces/ViewerAPI.html
https://developer.trimble.com/docs/connect/tools/api/property-set

## Comments
Select one or more objects in the viewer, type a comment, and click **Add comment**. Each comment is appended with a UTC timestamp to each selected element. Existing comments remain when statuses change. Shared-service mode saves the comment history on the server. Comments are also exported in the `CommentsJSON` CSV column. The pilot server does not enforce immutable audit history or identify commenters.
