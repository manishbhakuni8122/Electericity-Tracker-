# ⚡ Pan India Electricity Dashboard 2026

A single-file, browser-based dashboard for tracking electricity bills across stores and locations pan-India. It supports month-wise tracking, due-date alerts, payment status, bill/receipt links, bulk import/export, and a full edit history with undo, with data stored in [Supabase](https://supabase.com/) and synced between users in near real time.

No build step, no framework, no backend server. Just one `index.html` file.

---

## ✨ Features

### Bill tracking
- **Month-wise tabs**: create, switch, and delete months (e.g. `Apr 2026`).
- **Per-store records**: location, brand, electricity company, CA number, bill date, amount, due date, status, remarks.
- **Second bill support**: a separate set of columns (Amount 2, Due Date 2, Paid By 2, Status 2) for stores with a second bill in the same month.
- **Inline editing**: click any cell to edit; press `Enter` to save, `Esc` to cancel.
- **Auto payment date**: typing a status containing "paid" auto-appends today's date (`DD.MM.YYYY`).
- **Add / delete stores** with confirmation prompts.

### Alerts & KPIs
- **Due-date alerts**: Overdue, Due Today, Due in N days, colour coded.
- **KPI cards**: total bills, overdue count and amount, due-today count and amount.
- **Date-range KPI**: pick a From/To range to see unpaid bills due in that window, with an Excel download of the result.
- **Row highlighting**: overdue rows in red, fully paid rows in green, duplicates in amber.

### Data management
- **Import** from CSV / XLSX / XLS with automatic column matching and a preview before confirming.
- **Export** the current month as CSV or Excel, or all months as a multi-sheet Excel workbook.
- **Duplicate detection** with a review-and-remove flow (exact duplicates are removed, keeping the first occurrence; same-CA conflicts are flagged for manual review).
- **Search** with suggestions across location, brand, CA number, and company, plus company and brand filters and sortable columns.
- **Bill & receipt links**: attach Google Drive links, preview them in-app, or open in Drive.
- **Company portals**: each electricity company links to its payment portal (BESCOM, MSEDCL, BSES, Tata Power, TNEB, KSEB, WBSEDCL, DHBVN, UHBVN and more). Add your own from the **Companies** menu.

### Collaboration & safety
- **Live sync**: polls Supabase every 5 seconds and refreshes when another user changes data.
- **Edit history panel**: every edit, add, delete, import, and reset is logged with user and timestamp.
- **Undo**: revert a single field change, restore a deleted row, or undo an import.
- **Login gate** before the dashboard loads (see the security notes below).

---

## 🧱 Tech Stack

| Layer | Tool |
|-------|------|
| UI | Vanilla HTML, CSS, JavaScript (no framework) |
| Database | Supabase (PostgreSQL via REST API) |
| Excel / CSV | [SheetJS (xlsx)](https://sheetjs.com/) loaded from CDN |
| File storage | Google Drive (links only) |

---

## 🚀 Getting Started

### 1. Clone the repo
```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

### 2. Create the Supabase tables

In your Supabase project, open the SQL editor and run:

```sql
create table electricity_bills (
  id           bigint generated always as identity primary key,
  month        text not null,
  location     text,
  brand        text,
  company      text,
  ca           text,
  bill_date    text,
  amount       text,
  due_date     text,
  status       text,
  remarks      text,
  bill_link    text,
  receipt_link text,
  amount_2     text,
  due_date_2   text,
  paid_by_2    text,
  status_2     text
);

create table change_history (
  id         bigint generated always as identity primary key,
  timestamp  timestamptz default now(),
  user_id    text,
  action     text,
  month      text,
  location   text,
  ca         text,
  field      text,
  old_value  text,
  new_value  text,
  extra      text
);
```

> The app works without `change_history`, but history will then be kept in memory only for the session.

### 3. Configure the app

Open `index.html` and update the config block near the top of the script:

```js
var SUPA_URL = "https://<your-project>.supabase.co";
var SUPA_KEY = "<your-supabase-anon-key>";
var AUTH_ID  = "<your-login-id>";
var AUTH_PASS = "<your-password>";
```

Also update the Google Drive folder links:

```js
var BILL_FOLDER = "https://drive.google.com/drive/folders/<bills-folder-id>";
var RCPT_FOLDER = "https://drive.google.com/drive/folders/<receipts-folder-id>";
```

### 4. Run it

Open `index.html` in a browser, or host it on any static host (GitHub Pages, Netlify, Vercel, Cloudflare Pages).

**GitHub Pages:** go to *Settings → Pages*, choose your branch and the root folder, and save.

---

## 📥 Import Format

Upload a CSV or Excel file where the first row is the header. Columns are matched automatically (case-insensitive):

| Field | Accepted headers (examples) |
|-------|------------------------------|
| Location | Location, Store, Site |
| Brand | Brand |
| Company | Electricity Company, Company, Provider, Discom |
| CA No. | CA No., CA Number, Consumer No, Account |
| Bill Date | Bill Date, Billing Date |
| Amount | Amount, Bill Amount, ₹ Amount |
| Due Date | Due Date, Payment Due, Last Date |
| Status | Status, Payment Status |
| Remarks | Remarks, Notes, Comments |
| Bill / Receipt Link | Bill Link, Receipt Link |
| Second bill | Amount 2, Due Date 2, Paid By 2, Status 2 |

`S.No` and `Bill No` columns are ignored. Dates in `YYYY-MM-DD`, `DD.MM.YYYY`, `DD/MM/YYYY`, and Excel serial formats are supported.

> ⚠️ Importing **replaces** all data in the currently selected month. Use the history panel to undo.

---

## 🔒 Security Notes (Please Read)

This app is a client-side page, so **anything in the HTML is visible to anyone who can load the page**. Before making this repository or site public:

1. **Do not commit real credentials.** The login ID and password in `index.html` are not real security, since anyone can read them via *View Source*. Treat the built-in login as a convenience screen only.
2. **Use Supabase Row Level Security (RLS).** The anon key is designed to be public, but only if RLS policies restrict what it can do. Without RLS, anyone with the key can read, edit, or delete your data.
3. **Prefer real authentication.** Replace the hardcoded login with [Supabase Auth](https://supabase.com/docs/guides/auth) and write RLS policies that allow access only to authenticated users.
4. **Keep the repo private** if it contains live keys, internal Drive folder links, or real consumer (CA) data.
5. **Rotate keys** if they were ever pushed to a public repository.

---

## 📁 Project Structure

```
.
├── index.html   # The entire app (UI, logic, styles)
└── README.md
```

---

## 🛠 Roadmap Ideas

- Replace the hardcoded login with Supabase Auth and per-user roles
- Use Supabase Realtime instead of 5-second polling
- Upsert-based saves instead of delete-and-reinsert per month
- Direct file upload to Drive or Supabase Storage
- Payment reminders via email or WhatsApp

---

## 🤝 Contributing

Issues and pull requests are welcome. For major changes, please open an issue first to discuss what you'd like to change.

## 📄 License

Add a license of your choice (e.g. [MIT](https://choosealicense.com/licenses/mit/)) in a `LICENSE` file.

