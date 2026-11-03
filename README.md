# Kamagere Mart

Kamagere Mart is a Google Apps Script storefront backed by Google Sheets. The frontend can also be built as a static site and configured to call a deployed Apps Script web app.

## Configure Google Apps Script

In the Apps Script project's **Project Settings → Script Properties**, configure:

- `SPREADSHEET_ID` — the ID of the Google Sheet used by this deployment
- `ADMIN_EMAIL` — the administrator's sign-in email
- `ADMIN_PASSWORD` — a unique password that meets the validation rules in `Config.js`
- `GOOGLE_CLIENT_ID` — optional; required only for Google sign-in

Run `setup()` from the Apps Script editor after setting the required properties and granting access to the spreadsheet. The setup creates `ADMIN_JWT_SECRET` if it is missing. Never put these values in source code or commit them.

Deploy the project as a web app and configure its access settings to suit your use case. To send customer verification mail from `kamageremart04@gmail.com`, deploy the web app while signed in to that account, or add it as a verified **Send mail as** alias on the account that owns the deployment. The manifest includes Gmail send and Gmail settings scopes for this sender check. After a scope change, open the Apps Script editor while signed in to the intended sending account, run `authorizeRegistrationEmail()`, and approve the requested permissions. This sends a test message to `kamageremart04@gmail.com`; registration sends codes only after the configured sender is authorized.
The deployment serves two separate HTML pages: `UserPortal.html` for the customer storefront and `AdminPanel.html` for administrators. Open the deployment URL as `/exec` for the customer portal or `/exec?qadmin=1` for the admin panel. Customer login uses `/api/login`; the admin page uses `/api/admin/login` and opens the protected dashboard only after an administrator signs in. Admin and customer sessions remain independent, so both pages can be open in separate tabs in the same browser without signing each other out or changing portals. Both pages use the same Apps Script backend and store data.

Customer registration sends a six-digit code to the submitted email. Codes expire after 10 minutes, can be resent once per minute, and allow up to five verification attempts. The account is created only after the code is verified; the customer is then returned to the login form.

## Payments and WhatsApp invoice

- UPI + Cash on Delivery: see `PAYMENT_SETUP_NOTES.md` (script properties `UPI_ID`, `UPI_NAME`).
- WhatsApp invoice after an order: see `WHATSAPP_SETUP.md` (script properties `WA_*`).
- Placing an order saves it and clears the cart before opening a prefilled WhatsApp message in a new tab; no extra WhatsApp confirmation popup or repeat-send button is shown. The customer must still tap **Send** in WhatsApp.

## Build a local or static frontend

Set `KM_API_URL` to the HTTPS Apps Script web app URL ending in `/exec`, then run:

```powershell
$env:KM_API_URL = "https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec"
node build-netlify.mjs
```

The generated `dist` directory is intentionally ignored by Git because it embeds the API URL. For a local preview, serve `dist` with a static HTTP server.
While the storefront is open, product stock refreshes about every 20 seconds and signed-in customer order tracking refreshes about every 15 seconds. Admin status changes and cancellations are saved to the same backend used by the customer pages.

## Important

- Do not commit `.clasp.json`, `.clasprc.json`, `.env` files, or generated deployment output.
- Keep spreadsheet IDs, deployment URLs, OAuth client IDs, and all admin credentials in local or Apps Script configuration, not in public source.
- Before making a deployment public, verify its Script Properties and rotate any credentials that may have used old defaults.
