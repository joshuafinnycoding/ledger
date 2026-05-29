# Ledger — Setup & Deployment Guide

A mobile-friendly tracker for **expenses, income, and investments**, restricted to you and your family via a private URL + access token.

**Total setup time: ~20 minutes. Total cost: ₹0 forever.**

Just **two files** to deploy:
- `ledger.html` — the entire app (HTML + CSS + JS in one file)
- `AppsScript_Code.gs` — the backend, pasted into Google Apps Script

---

## How access works

Three layers protect your data:

1. **The static site** (where `ledger.html` is hosted) is public — but it's just an empty lock screen. Knowing the URL alone gets you nothing.
2. **The Apps Script URL** is a secret. Entered once on the lock screen, stored in the browser.
3. **The access token** is a second secret. Without it the backend rejects every request.

```
   Public                Secret (kept private)              Secret
┌──────────────┐      ┌────────────────────────┐       ┌──────────────┐
│ Your hosted  │      │ Apps Script Web App    │       │ Google Sheet │
│ ledger.html  │──────│ /exec URL              │───────│ (4 tabs)     │
│              │  +   │ + ACCESS_TOKEN check   │       │              │
└──────────────┘      └────────────────────────┘       └──────────────┘
   anyone can               only you + family              only you
   visit                    have these credentials         (and Google)
```

---

## What you'll need

- A Google account (for Sheets + Apps Script)
- A GitHub account (free) for hosting `ledger.html`
- ~20 minutes
- A private channel to share credentials with family (WhatsApp DM, Signal, email)

---

# Step 1 — Create your Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and click **+ Blank**.
2. Rename it to `Ledger` (top-left).
3. Leave the sheet empty — the script populates it automatically.

---

# Step 2 — Generate a strong access token

Use [random.org/strings](https://www.random.org/strings/) or any password generator. Recommended: **20+ characters, letters + numbers**.

Example: `f7K9xQ2pL8mN4vR3wY5z`

**Save it somewhere safe** (password manager). You'll need it twice — once in the Apps Script, and once to share with family.

---

# Step 3 — Add the Apps Script backend

1. In your sheet, click **Extensions → Apps Script** (new tab opens).
2. Delete the placeholder code in `Code.gs`.
3. Open `AppsScript_Code.gs` from this project. Copy **all** its contents and paste into `Code.gs`.
4. Near the top, find:
   ```js
   const ACCESS_TOKEN = 'REPLACE_WITH_YOUR_SECRET_TOKEN';
   ```
   Replace with your token from Step 2:
   ```js
   const ACCESS_TOKEN = 'f7K9xQ2pL8mN4vR3wY5z';
   ```
5. Click 💾 **Save**, give the project a name (e.g., "Ledger backend").

---

# Step 4 — Initialize the sheet

1. In the Apps Script editor, find the function dropdown near the top.
2. Select **`setup`** → click **▶ Run**.
3. First-time permission prompt:
   - **Review permissions** → choose your account
   - Click **Advanced → Go to [project name] (unsafe) → Allow**
   - ("Unsafe" appears because the script isn't Google-verified. It's your own code; safe.)
4. Switch to your Google Sheet — four new tabs appear:
   - **Transactions**, **Categories**, **Subcategories**, **PaymentMethods** — all pre-filled with sensible defaults.

---

# Step 5 — Customize your data (recommended)

**Do this before sharing with family**, otherwise renaming things later means existing transactions reference stale names.

### `PaymentMethods` tab — most important to verify
The 15 defaults represent a 2-person family with 4 bank accounts + 4 UPI instruments + 5 credit cards + Pluxee + Cash. Edit to match your real situation.

Valid `type` values: `Bank Account`, `UPI`, `Credit Card`, `Debit Card`, `Net Banking`, `Wallet`, `Cash`, `Other`.

### `Categories` and `Subcategories` tabs
Add, edit, or delete rows freely. Categories belong to one of three types: `expense`, `income`, `investment`. Subcategories reference their parent category by exact name.

---

# Step 6 — Deploy the script as a Web App

1. In Apps Script, click **Deploy** (top right) → **New deployment**.
2. Click ⚙️ next to "Select type" → choose **Web app**.
3. Fill in:
   - **Description:** `Ledger v1` (any label)
   - **Execute as:** `Me (your email)` ← critical
   - **Who has access:** `Anyone` ← required. Your token gates access, not Google's permission system.
4. Click **Deploy**.
5. Copy the **Web app URL**:
   ```
   https://script.google.com/macros/s/AKfycby...../exec
   ```
6. Save this URL alongside your token. This is your second secret.
7. Click **Done**.

> 🔄 If you ever change the Apps Script code: **Deploy → Manage deployments → ✏️ Edit → Version: New version → Deploy**. URL stays the same.

---

# Step 7 — Personalize the frontend (optional)

Open `ledger.html`. Near the top of the `<script>` block at the bottom, you'll see:

```js
window.LEDGER_CONFIG = {
  CURRENCY: '₹',
  PEOPLE: ['JF', 'AJ']
};
```

Edit `CURRENCY` and `PEOPLE` if needed. Nothing else needs changing — the Apps Script URL and token are entered on the lock screen, not baked into the file.

---

# Step 8 — Test locally (optional but recommended)

```bash
cd ledger
python3 -m http.server 8000
```

Open `http://localhost:8000/ledger.html` in a browser. You'll see the lock screen:
1. Paste your Apps Script URL
2. Paste your access token
3. Click **Open**

If it works, add a test transaction and verify it lands in the Transactions tab of your sheet.

Common errors:
- "Wrong token" — token in `AppsScript_Code.gs` doesn't match what you typed
- "Could not connect" — bad URL, or deployment isn't "Anyone access"
- "unauthorized" — re-run `setup` in script editor (you may have skipped permissions)

---

# Step 9 — Host on GitHub Pages

1. Go to [github.com](https://github.com) → **+ → New repository**.
2. Name it (e.g., `ledger`). Set **Public**. Don't init with README. **Create**.
3. On the empty repo page, click **uploading an existing file**.
4. Drag in **only one file**: `ledger.html`. (Optionally also `README.md` for your own reference.)
   - **Do NOT upload** `AppsScript_Code.gs` — that lives in Apps Script, not on your site.
5. Click **Commit changes**.
6. Settings → **Pages** (left sidebar).
7. Source: Branch `main`, folder `/ (root)` → **Save**.
8. Wait ~60 seconds. Refresh the Pages settings — your URL appears:
   ```
   https://yourname.github.io/ledger/ledger.html
   ```
   *(If you want a cleaner URL like `https://yourname.github.io/ledger/`, rename `ledger.html` to `index.html` before uploading.)*
9. Open on your phone → you should see the lock screen.

---

# Step 10 — First-time login

On your phone:
1. Open your Pages URL.
2. Paste the **Apps Script URL** into the first field.
3. Paste the **access token** into the second.
4. Tap **Open**.

The credentials are saved in your phone's browser. From now on, opening the URL lands you directly in the app — no typing.

**Pro tip:** add to Home Screen on iOS/Android so it feels like a native app:
- Safari (iOS): Share → Add to Home Screen
- Chrome (Android): ⋮ → Add to Home Screen

---

# Step 11 — Share with family

Send each family member three things via a private channel (WhatsApp DM, Signal, etc.):
1. Your Pages URL: `https://yourname.github.io/ledger/`
2. The Apps Script URL
3. The access token

They paste the URL + token into their lock screen once, and they're in.

> ⚡ Want one-click family setup? You can construct a "magic link" that pre-fills credentials:
> ```
> https://yourname.github.io/ledger/#setup=BASE64
> ```
> where `BASE64` is base64 of `{"api":"<your-apps-script-url>","token":"<your-token>"}`.
>
> Quickest way to generate this from the browser console:
> ```js
> btoa(JSON.stringify({api:'PASTE_URL', token:'PASTE_TOKEN'}))
> ```
> Append the output to `#setup=` and share that link. Family members tap it, credentials get auto-saved, no typing required.
>
> ⚠️ The link contains your token — share only privately, never publicly.

---

# Daily use

**Adding a transaction (~5 seconds):**
1. Tap **Expense / Income / Investment**
2. Type the amount
3. Pick category → subcategory
4. Pick how you paid
5. Tap **Add**

**Editing/deleting:**
- Long-press an entry in the Recent list → Delete appears
- For edits: edit the row directly in the Transactions tab of your sheet, then refresh the app

**Insights:**
- Tap the **Insights** tab → toggle type at the top → use ‹ › to scroll months

---

# Security best practices

**Rotate the access token** if a link leaks or anyone outside the family sees the URL+token combo:
1. Generate a new random token
2. Edit `AppsScript_Code.gs` → change `ACCESS_TOKEN`
3. Save → **Deploy → Manage deployments → Edit → New version → Deploy**
4. Send the new token to family members. They tap "change" on the lock screen and re-enter.

**Back up the sheet:** in Google Sheets, **File → Make a copy** periodically. Google also keeps version history automatically.

---

# Troubleshooting

| Problem | Fix |
|---|---|
| "Wrong token" | Token in `AppsScript_Code.gs` ≠ what's typed. Edit, **redeploy** new version. |
| "Could not connect" | Bad URL, or deployment not set to "Anyone" access. |
| "URL must be an Apps Script /exec URL" | The URL must end in `/exec`. The `/dev` URL won't work. |
| Family member can't see new transactions | Tell them to refresh the page — the app fetches data on launch. |
| Changed Apps Script code, app uses old logic | Forgot to redeploy. **Deploy → Manage deployments → Edit → New version → Deploy**. |
| Family member's app stopped working after token rotation | They need the new token. Tell them to tap "change" on the lock screen and re-enter URL + new token. |
| Added a category in the sheet, not showing in app | Refresh the page. |

---

# Reference: data model

**Transactions tab columns:**
`id | type | date | amount | category | subcategory | note | person | payment_method | payment_type | created_at`

Your Google Sheet is the source of truth. The app is just a friendlier interface over it — you can sort, filter, pivot, and chart directly in Sheets whenever you need a view the app doesn't provide.
