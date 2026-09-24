# GR_Exp: Sadeq GR Exp

A static page that tracks services billed to a borrowed IBAN and the money owed between two people.
The page holds no data. Everything lives in a Google Sheet, and Google sign-in decides who can read it.

Page address (GitHub Pages): https://sadeq-alrumaih.github.io/GR_Exp/

## Rules
- Never put an IBAN, password or exported data in this repo.
- The sheet's sharing must be **Restricted**: only the two people, by email. Never "Anyone with the link".

## 1. Create the sheet
1. New Google Sheet, name it `Sadeq GR Exp`.
2. Rename the first tab to `Services`, add a second tab named `Payments` (exact names).
3. Paste the header rows from `sheet-templates/`:
   - Services: `id, service, start, end, url, amount, comment, moved, account, billing, moved_on, auto_renew`
   - Payments: `id, kind, date, amount, note`
4. Share > add your friend's email as Editor. General access: Restricted.
5. Copy the sheet ID from its URL: `docs.google.com/spreadsheets/d/<SHEET_ID>/edit`.

## 2. Google Cloud OAuth client
1. console.cloud.google.com > new project (e.g. `gr-exp`).
2. APIs & Services > Library > enable **Google Sheets API**.
3. OAuth consent screen: External, app name `Sadeq GR Exp`. Add both emails as **Test users**. Leave it in Testing.
4. Credentials > Create credentials > OAuth client ID > **Web application**.
   - Authorized JavaScript origins: `https://sadeq-alrumaih.github.io` (no path, no trailing slash).
5. Copy the client ID.

Testing mode shows an "unverified app" screen. Click Advanced > Continue; it is your own app.
In Testing mode Google expires sign-ins after 7 days, so you sign in again weekly.

## 3. Configure and publish
1. Edit `config.js`: paste `CLIENT_ID` and `SHEET_ID`.
2. Push to GitHub, then Settings > Pages > Deploy from branch `main`, folder `/ (root)`.
3. Open the page address and sign in.

## 4. Reminder before an end date (n8n)
Workflow: Schedule (daily 08:00) > Google Sheets "Get rows" (sheet `Services`) > Code > your notifier (email or Telegram).

Code node:

```js
const today = new Date(); today.setHours(0,0,0,0);
return $input.all().filter(i => {
  const r = i.json;
  if (!r.id || String(r.moved).toLowerCase() === 'yes') return false;
  const days = Math.round((new Date(r.end) - today) / 864e5);
  return days === 14 || days === 7 || days === 1;
}).map(i => ({ json: {
  text: `${i.json.service} ends ${i.json.end}. Move it to your own IBAN. ${i.json.url || ''}`
}}));
```
