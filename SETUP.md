# Reorder Desk: setup guide for IT

This folder is a complete website. It needs a free Firebase project for three things: hosting the page, member sign-in, and the shared database. Setup takes about 30 minutes. Menu names below are as in the Firebase console in October 2026 and may differ slightly.

## What is in the folder

| File | Purpose |
| --- | --- |
| `public/index.html` | The whole site in one page. |
| `public/config.js` | Where the Firebase project settings are pasted (step 4). |
| `firestore.rules` | Who may read and write. Only signed-in, email-confirmed people on the member list. |
| `firebase.json` | Tells the Firebase tool what to publish. |
| `../starter-data/` | Five upload files made from the Myra order file of 8 Oct 2026, in the layouts below. |

## Steps

1. **Create the project.** Go to https://console.firebase.google.com, add a project, and stay on the no-cost Spark plan. No payment method is needed.
2. **Turn on sign-in.** Build > Authentication > Get started > Sign-in method > enable **Email/Password**.
3. **Create the database.** Build > Firestore Database > Create database. Choose Standard edition, production mode, and a nearby region such as `asia-south1` (Mumbai). The region cannot be changed later.
4. **Connect the site.** Project settings > Your apps > add a **Web app**. Copy `apiKey`, `authDomain`, `projectId` and `appId` into `public/config.js`.
5. **Name the first admin.** Firestore Database > Data > Start collection. Collection ID `members`. Document ID is the admin's email address in lower case. Add one field: `role` (string) = `admin`.
6. **Publish.** With Node.js installed, run these in this folder:
   ```
   npm install -g firebase-tools
   firebase login
   firebase deploy --project YOUR_PROJECT_ID
   ```
   This publishes the page and the access rules together. The site is then at `https://YOUR_PROJECT_ID.web.app`.
7. **First sign-in.** The admin opens the site, chooses "First time here? Create an account", uses the email from step 5, opens the confirmation link sent by email, then presses Continue.
8. **Settings and members.** On the Settings tab, check each depot's safety, transit and order frequency days and press Save. On the Users tab, add the other members' emails. Each member then creates their own password the same way as in step 7.
9. **Load data.** On the Updates tab upload the five files in `starter-data` in order, or your own files in the same layout.

## How access works

- A person can use the site only if they have signed in, confirmed their email, and their email is a document in `members`.
- Members see everything, change filters and download every report. Only admins upload data and change Settings and the member list.
- Each person can write only their own sign-in and upload record.
- The values in `config.js` identify the project and are visible to anyone who opens the page. That is normal for Firebase. They do not give access to data; the rules do.
- Anyone can create a sign-in account, but an account whose email is not on the member list sees only a "not on the member list" message and can read nothing.

## Free plan limits (Firebase pricing page, checked 10 Oct 2026)

- Database: 1 GiB stored, 50,000 reads and 20,000 writes per day.
- Hosting: 10 GB stored, 360 MB transfer per day.
- Sign-in: 50,000 monthly active users.

Twenty members uploading daily use a small fraction of this. One full day of uploads is about 90 writes, and each page load is about 110 reads.

## Uploads

Each upload is checked first and a summary is shown. Nothing is stored until the person presses Save. A file that fails a check cannot be saved, and the reason is shown.

| Upload | Layout |
| --- | --- |
| Item master | One sheet: Item Code, Item Name, Brand, Category, Sub-Category, Location, Status, Case Lot, Case Wgt, CBM, MRP, POC, Size. |
| Forecast | One sheet: Item Code, Item Name, then one column per channel and depot, such as `AVT | Chennai`, `AVT | Coimbatore`. Monthly forecast in cases. |
| Direct sale forecast | Same layout as the forecast. The part of the forecast planned as direct sale from the plant. It is taken off the forecast, channel by channel, before orders and reports are worked out. |
| Daily stock | Three sheets named `Phy`, `Transit` and `Reserve`. Each has Item Code, Item Name, then one column per depot. An optional `Plant FG` column on the Phy sheet switches on the Plant stock tab. |
| MTD sale | One sheet, one row per item, channel and depot: Item Code, Item Name, Sale Channel, Depot, Sale Qty. |
| Open PO | Two sheets named `Ecom` and `MT`, one row per item and depot: Item Code, Item Name, Depot, PO Qty. The Ecom sheet also has PO Date. |
| Factory order | One row per item: Item Code, Item Name, Brand, Category, Sub-Category, Size, Case Wgt, CBM, Case Lot, then a column per depot with the order qty in cartons. The order date is set at upload. |
| Factory dispatch | One row per dispatch: Dispatch Date, Item Code, Item Name, Location, Qty in cartons. |
| Factory FG | One row per item: Item Code, FG Qty (Cases). |
| Ecom direct factory plan | One row per item and depot: Item Code, Item Name, Depot, Plan Qty (Pcs). |
| Factory direct plan | One row per item: Item Code, Plan Qty (Cases). |
| Ecom SKU list | One row per item: Item Code (Item Name optional). The SKUs sold on Ecom. |
| Sales history | One row per item, month, channel and depot: Item Code, Item Name, Month, Sales Channel, Depot, Qty (pcs). Any number of months; each upload replaces only the months in it. |
| Past forecast (for accuracy) | Same layout as the forecast; pick its month at upload. Compared with the MTD sale in the Forecast accuracy report. |

- Depot names must match the depots in Settings. Rows or columns with other names are skipped and listed after the upload.
- Forecast and stock files may hold only some depots; the other depots keep their earlier figures.
- MTD sale and open PO files replace all earlier figures. Depots and items not in the file count as zero.
- The Template button on each upload downloads the right layout for the current depots and channels. The sale and PO templates include a Lists sheet with the depot names.

## Other tabs

- **Reports:** forecast vs MTD sale % by depot and channel, and by item. It is compared with how much of the month had gone by the day before the MTD sale was uploaded. It downloads as Excel.
- **History:** each day's orders are saved automatically after an upload, and kept for 60 days. Each saved day can be downloaded.
- **Safety stock shortfall (Reports):** DRR = MTD sale of all channels except Ecom / days up to the day before the MTD sale upload (uploaded on the 15th = 14 days), per item and depot. Safety stock need = DRR x the item's brand safety days at the depot; total required = safety stock need + Ecom open PO older than 6 days by PO Date (set in Settings) + MT open PO (Ecom counts only through its open PO). Brand safety days are set under Settings > Safety stock days by brand and depot (typed in or uploaded as a sheet: Brand, then one column per depot). Factory balance pending = factory order (cartons x case lot) - dispatches dated on or after the order date. Shortfall = total required - (physical + transit + factory balance pending), shown as item rows with one column per depot. Factory FG of up to 5 cases for an item counts as zero (set in Settings). Ecom open PO is net of the Ecom direct factory plan for its depot. Balance FG = factory FG - factory balance pending of all depots - Ecom direct factory plan - factory direct plan; the shortfall is sent from balance FG, in full when it is enough, otherwise shared across depots in proportion to their shortfall.
- **Ecom PO not billed (Reports):** items where the Ecom open PO is older than the PO age in Settings (6 days), less the Ecom direct factory plan, and the depot already holds stock (physical + transit). One row per item with Item Code, Item Name, Category, Sub-Category and POC, then four columns per depot: Ecom ageing PO, Stock, CBM and MT (metric tonnes) for the quantity that can be billed now (the smaller of PO and stock). The Excel download also has a detail sheet by depot and a depot summary.
- **Ecom stock over PO (Reports):** Ecom SKUs from the Ecom SKU list whose depot stock (physical + transit) is more than a chosen % (dropdown, default 30%) above the Ecom open PO; a depot with stock and no PO is always listed. Per depot: Ecom PO, Stock, Excess stock (stock - PO), CBM and MT of the excess.
- **MT low pick-up (Reports):** depots holding stock where MT MTD sale + MT open PO is below a chosen % (dropdown, default from Settings, 30%) of the MT forecast (net of direct sale). Per depot: MT forecast, MT sale, MT PO, Pick-up %, Stock, and CBM and MT of the stock up to the forecast (the smaller of stock and forecast).
- **Dashboard:** built from the sales history. Filters for period (last 3/6/12/24 months or all), channel and depot, plus the item filters and Pcs/Cases. Shows sales KPIs, monthly trend, channel mix by month, depots, categories and top SKUs, then decision views: growing and falling SKUs (last 3 months vs the 3 before), ABC classes, forecast vs actual run-rate, depot stock cover in days, and stock with no sale in the last 3 months. Every chart has a table view and the whole dashboard downloads to Excel. The sales history loads only when the Dashboard is opened.
- **Statistical forecast (Dashboard):** for every SKU, channel and depot: weighted average of the last 3 months of sales history (50% latest month, 30%, 20%), times each channel's seasonal change in the same months last year (limited to 0.7-1.4x) once 15+ months are loaded. Choose the month (1-3 months ahead), compare with the uploaded (planned) forecast, see the method's back-test accuracy, and download it in the forecast upload layout to edit and upload. Also on the Dashboard: planned vs statistical gaps per SKU, ABC x XYZ demand stability with stock policy per cell, and new and lost SKUs.
- **Over consumption (Reports):** MTD sale + open PO above 100% of the current month's forecast (net of direct sale), per item and location, for all channels or one channel. Per location: Forecast, MTD sale, Open PO, Sale + PO, Consumed %, Over qty. The download has an All channels sheet, a detail sheet (item x location x channel), a location summary and one sheet per channel.
- **Forecast accuracy (Reports):** the past forecast (uploaded with Past forecast, for a chosen month) against the MTD sale as uploaded. Accuracy = sale / forecast. Remark: OK = 80% to 120%, Below forecast = under 80%, Above forecast = over 120% (or a sale with no forecast). One row per item (Item Code, Item Name, Brand, Category, Sub-Category, POC); across the top each location, under it each channel and the location Total, each with Forecast, Sale, Accuracy and Remark; then a Grand total. Summaries by channel and by location. The download has an All channels sheet, the two summaries, and one sheet per channel.
- **Send:** pick a report and the people to get it. The page downloads the report and copies the addresses, subject and message, to paste into a new email with the file attached. The team email list (Name, Email) is uploaded once on the same tab; only admins can change it.
- **Users:** each member's last sign-in, number of sign-ins and uploads, and the 40 most recent uploads with who made them. Admins add and remove members here.

## Who can change data

Only the owner changes data: uploads, settings, deleting reports and the mail list. On the website that is every member with the admin role; other members see everything, change filters and reports, and download, but cannot upload or edit (enforced by the database rules). The Send and Users tabs are shown to the owner/admins only.

## Deleting report types

The owner (on the website: an admin) sees a Delete report button next to the Report dropdown. A deleted report type disappears from the Reports list and the Send tab for everyone; the owner restores it from the Restore deleted report list beside it. Nothing in the data is deleted. At least one report always stays. The list of deleted reports can only be changed by the owner (admins on the website, enforced by the database rules).

## Download last upload

Each card on the Updates tab has a Download last upload button. It rebuilds the data that is saved now in that upload's template layout and units (pcs, cartons or cases, as the template asks), so it can be edited and uploaded again. For uploads that keep earlier depots (forecast, stock), it holds the combined figures in use.

## Filters

The filter bar under the summary has multi-select filters for POC, Brand, Category, Sub-Category and Item code (from the item master). Each one has a search box, Select shown and Clear; Item code also takes a pasted list of codes. Press Apply. Every tab, report, download and sent report then holds only the matching items, and the file names say which filter was used. Clear all filters shows everything. Each person's filters are remembered in their browser. The daily History copy always holds all items.

## Units

All uploads are in pcs. The "Show in" dropdown at the top of the page switches every tab, report and download between pcs and cases, where cases = pcs / Case Lot from the item master. Items with no Case Lot count 1 pc as 1 case. CBM and Case Wgt in the item master are per case.

## Order rule

`Order = (forecast - direct sale forecast) x (safety + transit + order frequency days) / working days + open PO - physical stock - transit stock`

Calculated per item and depot from the pcs uploaded, and rounded up to full cases (a multiple of the Case Lot). Forecast and open PO are summed across channels. Reserved stock and MTD sale are carried in the master file but do not change the order. Working days default to 26.

## What has and has not been tested

- Tested in a browser: all five uploads with the starter files, wrong files being refused, the order maths for every item and depot, the downloads, reports, history, per-user records, settings changes and the sign-in screens.
- The sign-in and database calls were tested against a local stand-in, not a live Firebase project. Expect to check steps 7 to 9 carefully on first use and report anything that does not behave as described.

## Good to know

- The confirmation and password-reset emails come from Firebase and can land in spam. The sender name and text can be changed under Authentication > Templates.
- A custom web address can be added under Hosting > Add custom domain.
- The database keeps only the latest upload of each kind, plus 60 days of order history. Keep the uploaded files if you need older inputs.
- The page loads the Excel reader from cdnjs.cloudflare.com and Firebase from gstatic.com, so office firewalls must allow both.
