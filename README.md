# McCoy's / Community Coffee Purchase Orders

## Setup — keep separate from all existing Live and Test apps
1. Create a **new** GitHub repository, e.g. `community-coffee-orders`. Upload all files in this ZIP to the repository root. Do not overwrite your existing racking or other apps.
2. Create a **new** Firebase project for coffee orders (not the Mc Racking Live or Test projects).
3. Firebase Console > Build > Authentication > Sign-in method: enable **Email/Password**. Under Users, add approved employee accounts with strong passwords. Do not enable public self-registration.
4. Firebase Console > Build > Firestore Database: create a database. Open Rules and replace the rules with `firestore.rules`, then Publish. These rules allow all signed-in employees to see orders, but do not include per-user roles or a full audit trail. Consider restricting to an allowlist for stronger access control.
5. Firebase Console > Project settings > Your apps > Add web app. Copy the Firebase config object values into `firebase-config.js` (keep `export default { ... }`).
6. Firebase Authentication > Settings > Authorized domains: add your GitHub Pages host, typically `YOURNAME.github.io`.
7. GitHub repository > Settings > Pages > Deploy from branch > `main` / `(root)` > Save. Your site will appear at `https://YOURNAME.github.io/community-coffee-orders/`.
8. On an iPhone open the website in Safari > Share > Add to Home Screen.

## How it works
- Sign in, fill PO number and requester, enter quantities, and **Save order** to Firestore.
- **Create PDF** downloads a PDF to your device. PDF creation alone does not save the order.
- **Email PDF** downloads a PDF and opens your mail client with a draft; **attach the downloaded PDF manually** before sending. The vendor email is intentionally not prefilled because no confirmed address was supplied.
- **Order history** lists saved orders. Use View/edit, PDF, or status selector (Draft, Sent, Received, Cancelled). Mark Sent manually after sending; sending email does not auto-update status.

## Product catalog
Product IDs, descriptions and unit prices were transcribed from Community Coffee invoice **14245628736, dated 2026-10-14**. Prices are **reference estimates, not a live price feed**. The free filter line is included at $0 only as it appeared on that invoice, not as a guaranteed promotion. Change prices/catalog in the `products` array near the top of `index.html` if needed.

## Important limitations
- Hosting and Firestore require your own GitHub/Firebase setup. This ZIP is a working starter app, not an already published site.
- Firebase authentication is required. Anyone with a valid Firebase user account for this project can read all orders. Add finer-grained permissions if required.
- Email uses the phone/computer email application; the app cannot attach the PDF automatically using a mailto link. For true one-tap sending with automatic attachment, a secure backend/email service is needed.
- A PDF is a purchase order, **not** an invoice, and does not charge a payment method.
- This version does not upload PDF files to Firebase; it stores the order data and regenerates PDFs on demand.

## Pricing admin (v2)
1. Publish the **new firestore.rules** in the Firebase Console (Firestore Database > Rules > Publish) BEFORE using Admin.
2. Firebase Console > Authentication > Users: copy the **User UID** for the account that will manage prices.
3. Firebase Console > Firestore Database > Data: start collection `admins`. Set **document ID to the exact UID** copied above. Create field `enabled` of type **boolean** with value `true` (not the text string "true"). Only do this for trusted admins. Do not put the UID into the HTML.
4. Sign out and sign back in to the website. The **Admin · Pricing** tab is shown only to accounts that have an enabled admin document.
5. Change paid-item prices and select **Save pricing changes**. This writes catalog documents to Firestore and updates prices for new orders on every signed-in device. The sample defaults remain until an admin saves the catalog.
6. Bunn filters (item 90238) are placed under **Free merchandise** at $0.00, and appear in a separate PDF section. This reflects the sample invoice only, not a guarantee they will be supplied free.
7. Existing orders store unit prices with their line items, so historical order PDFs retain their order-time prices. If an older order is edited, its existing line items retain historical prices; new items use current pricing.
8. GitHub: upload updated `index.html`, `firestore.rules`, and `README.md`. Leave your already-configured `firebase-config.js` untouched. No new Firebase project needed.

**Security note:** Admin rights are enforced by Firestore rules, not just by hiding the tab. The `admins` collection is provisioned manually in Firebase Console; browser clients cannot write their own admin status. For production use, consider additional protections around catalog fields and historical order integrity.


## Version 3: auto-generated purchase orders
- Order dates default to today's local date and are editable.
- Requested by fills from the logged-in user's Firebase display name or email; it is read-only.
- On first **Save order**, a Firestore transaction assigns sequential PO numbers starting at **9001** using `settings/poCounter`. Subsequent edits keep the original number.
- **Create PDF** and **Email PDF** require saving the new order first. They load jsPDF with a backup CDN if necessary.
- **IMPORTANT:** Publish the NEW `firestore.rules` in Firebase console. GitHub upload alone does not deploy database rules.
- If Firestore already contains numbered orders from another numbering system, reconcile those manually before first use to avoid collisions.
- The counter is protected against non-incrementing updates, but signed-in employees can still increment it; use a server-side Cloud Function for audited/tamper-proof numbering.
- Existing `firebase-config.js` in your live GitHub project MUST be retained when uploading this ZIP. The packaged config remains a placeholder.


## Version 4 – Clear form after saving and PDF logos

- Saving a new or edited PO clears its input fields and quantities, ready for a new order.
- After saving, **Create PDF** and **Email PDF** operate on the just-saved order even though the form has cleared.
- **Order history** retains every saved order and offers PDF downloads.
- PDF header embeds both provided brand logos when available.
- The **Clear form** button also clears the last-saved PDF shortcut.
- Preserve existing `firebase-config.js` when replacing files on GitHub. No Firestore rules change required from version 3.

## Version 5: password reset

On the sign-in screen employees can enter their account email and click **Forgot password?**. Firebase Authentication sends a password-reset email (Email/Password sign-in must be enabled). The email action uses Firebase's default authorized handler; the site must be configured in Firebase Authentication. The password is never handled or emailed by this app.


## Version 6 — Add products from Admin

Admin → Product pricing → Add new product. Enter the vendor item number, product description, choose Purchase items or Free merchandise, and set a price. Free items must cost $0.00. Click Add product. The catalog is saved in Firestore under `catalog/{itemNumber}` and appears in every signed-in user's order page automatically, including PDF exports and saved order history. The item number must be unique and cannot be changed after creation. Existing items can have their paid prices edited using Save pricing changes.

Replace `index.html` on GitHub to deploy this version. Your existing Firebase config stays unchanged. The included `firestore.rules` protects the catalog with admin-only writes. Publish those rules if they differ from your current Firebase rules.

## Version 7: item sorting
Both the order entry sections and the Admin Pricing catalog display item numbers in ascending numeric order (lowest item number first), including items newly added through Firebase. No rules or configuration change is required.


## Version 9 — Admin pricing reliability
- Price edits remain in the Admin form while Firestore catalog updates arrive.
- Save pricing changes writes only prices that have changed and confirms the number saved.
- Rejects invalid prices, empty entries and more than two decimal places.
- Existing free merchandise remains at $0; previously saved order lines retain saved prices.
- Only users registered in the `admins` collection (enabled: true) can write prices, per `firestore.rules`.
- Test on your published site: change one price, save, sign out/in, verify the order form shows the change. Existing order PDFs should retain the old amount.


## Version 10 — Edit product names
On the Admin page, edit a product name and/or price and click **Save product changes**. Product item numbers remain unchanged, and free merchandise remains $0.00. New orders use the updated name and price; past orders keep their stored line descriptions and prices. No new Firestore rules are required if the existing catalog write permissions for admins are working.


## Version 11 — home-screen and desktop app icons

The web manifest now uses the custom Coffee Orders app icon (192 and 512 px).
The browser tab uses a favicon, and iOS uses the 180px Apple touch icon.
Upload the entire package to GitHub, but **keep your current configured firebase-config.js** instead of overwriting it with the placeholder included here. No new Firestore rules are needed.

**Windows Chrome/Edge**: Open the published GitHub Pages website, use the browser menu to install the app (the menu option varies by browser). You can also create a desktop shortcut via the browser menu, but a regular URL shortcut may use the browser icon rather than the app icon. If installing is offered, that is preferable for the branded icon.

**iPhone Safari**: Share > Add to Home Screen. If an older icon is cached, remove the previous home-screen shortcut and add it again.

Note: the manifest and icons make the site eligible for a branded install; browser/version/OS control the exact shortcut icon shown.


## Version 12: locking, receiving variances, grouped history, PO Search

**Deploy:** upload `index.html` and `README.md` to GitHub and keep your configured `firebase-config.js`. **You must also publish the new `firestore.rules`** (Firebase Console > Firestore Database > Rules > Publish). The new app will not work correctly with the old rules.

- **Anyone can change status.** Any signed-in employee can mark any PO Sent, Received or Cancelled. Only admins can move a PO back to Draft, which unlocks it.
- **Sent POs are locked.** Once a PO is Sent, Received or Cancelled, its products, quantities, prices, date, ship-to and notes cannot be changed. This is enforced by the database rules. After saving or emailing, use the **Mark PO … as Sent** button, or the status menu in Order history.
- **Receiving and variances.** On a Sent PO, choose **Log receiving**. Enter the quantity actually received per line, plus a note for each difference and general receiving notes. Saving sets the status to Received and flags the PO if anything was short or over. The received PDF adds a RCVD column, shows variances in red, and adds a Receiving section. Receiving can be edited later. The original ordered quantities never change.
- **Old orders keep their saved product names and prices** when viewed, edited or printed.
- **No skipped PO numbers.** New POs are stored with the PO number as their document ID, and the rules only let the counter advance in the same save that creates that PO. Older orders keep working as before.
- **Vendor email.** Admin > Vendor email. Email PDF drafts are addressed to it automatically.
- **PDF.** Long product names wrap instead of being cut off, columns line up, and pages are numbered.
- **Read-only fields** (PO number, Requested by) are shaded gray.
- **Auto-save.** **Save & create PDF** and **Save & email PDF** save the order first, then create the PDF. There's no need to press Save order first.
- **Order history.** This month is listed in full. Earlier months this year fold into one line each, and earlier years fold into one line per year with the months inside. Click any line to open it; every PO can still be viewed. A quick find or status filter opens all groups.
- **PO Search tab.** Filter by product, store, requester, status, PO #, receiving (variance / complete / not received), date range and keyword. Tiles show PO count, units, value and rows. Results are listed one row per product line, with received quantities in red where they differ. **Export Excel** produces an .xlsx file (or a CSV if the Excel library can't load). **Export PDF** produces a landscape report with the filters used and totals.

**Test after publishing:** create a PO, use Save & create PDF, mark it Sent, confirm View no longer allows edits, log receiving with one short line, then find it in PO Search with Receiving = Variance logged.

### v12.1 fix
- Receiving can now be logged on a PO still in **Draft**, for example when someone picks Received from the status menu without marking it Sent first. Saving receiving marks it Received and locks it. **Republish `firestore.rules`.**
- Permission errors now explain what to check instead of showing "Missing or insufficient permissions."

### v12.2: PDF viewer
- **Save & view PDF** (new orders), **View PDF** (Order history and locked POs) and **View PDF report** (PO Search) open the PDF in an on-screen viewer. Use **Download** to save it, **Open in new tab** for full screen or printing, and **Close** (or Esc) to return.
- **Save & email PDF** still downloads the PDF automatically, so it's ready to attach.
- On iPhone, the in-page preview may show only the first page. Use **Open in new tab** to see the whole PDF.
- No rules change from v12.1. Upload `index.html` only.

### v12.3: PO Search shows POs
- Results show **one row per PO**: date, PO, requester, lines, units ordered, units received, value, status and receiving result (Complete, "2 short", "1 over", or Not received).
- **Click a PO row** to expand it and see every item with ordered, received, variance (OK / Short / Over), price and variance note, plus order and receiving notes. Click again to collapse. Lines that match a product or keyword filter are highlighted.
- **Open** goes to the full PO; **PDF** opens it in the viewer.
- Filtering by product shows the POs containing that product, and the tiles show units and value for that product.
- **Export Excel** has two sheets: **POs** (one row per PO) and **PO items** (every line with its variance). **View PDF report** lists POs, with variance lines in red under each affected PO.
- Upload `index.html` only. No rules change.

### v12.4: Email PDF attaches the PDF automatically
- **On a computer:** **Save & email PDF** downloads `PO-####-email.eml`. Open it from your browser's downloads, and Outlook (or Windows Mail) opens a **draft addressed to mblackburn@communitycoffee.com** with the subject, order details and the **PDF already attached**. Press Send.
  - Tip (Chrome/Edge): click the arrow or "..." on the downloaded .eml and choose **Always open files of this type**, so future drafts open by themselves.
  - If .eml files open in the wrong program: right-click any .eml > Open with > Outlook > Always.
- **On iPhone/Android:** it opens the share sheet with the PDF attached. Pick Mail or Outlook. The vendor address is copied, so paste it into To. If the share sheet doesn't appear, tap **Email PDF** in the PDF viewer that opens.
- The PDF viewer has an **Email PDF** button too, so any PO can be emailed from Order history or PO Search.
- The default recipient is **mblackburn@communitycoffee.com**. An admin can change it under Admin > Vendor email.
- PDFs are now about 100 KB instead of about 9 MB (logos are compressed), so they're easy to email.
- Upload `index.html` only. No rules change.

### v12.5
- The email draft's To line is written as `<mblackburn@communitycoffee.com>` so Outlook shows the address once. If Outlook still shows it twice, press **Check names** (Ctrl+K). Either way, the email goes to the correct address.

### v12.6
- Email drafts are addressed to **"Mark Blackburn" <mblackburn@communitycoffee.com>**, so Outlook shows his name.
- Admin > Vendor email accepts `Name <email>` (for example `Mark Blackburn <mblackburn@communitycoffee.com>`) or a plain email. Separate multiple recipients with commas. If left blank, Mark Blackburn is the default.

### v12.7
- Email drafts **Cc Facilities Management <facilities_management@mccoys.com>** automatically (computer/Outlook drafts). On phones, the share sheet can't fill in Cc, so the app message reminds you to add it.

### v12.8: export the product list
- **Admin · Pricing > Export Excel** (next to Save product changes) downloads `Community-Coffee-Catalog-YYYY-MM-DD.xlsx` with every product on the list: Item #, Description, Section (Purchase items / Free merchandise) and Unit price. Paid items come first, then free.
- It exports the **saved** catalog. Save any edits first if you want them included.
- Upload `index.html` only. No rules change.

### v12.9: import a spreadsheet to update the product list
- **Admin · Pricing > Import Excel** lets you choose an .xlsx, .xls or .csv file. Easiest: click **Export Excel**, edit the file, then import it.
- Columns: **Item #**, **Description**, **Section** (Purchase items / Free merchandise), **Unit price**. Only Item # is required. A blank Description or Price keeps the current value. New items need a description, and a price if they're purchase items.
- A **preview** shows what will change: new products (green), updates such as "Price: $7.00 → $7.50" (yellow), and rows with problems (red), such as a duplicate Item # or a free item with a price. **Nothing is saved until you click Apply.** Rows with errors are skipped.
- Products missing from the sheet are left unchanged; nothing is deleted. Past POs keep their original prices and names.
- Upload `index.html` only. No rules change.

### v12.10: product photos
- Each product can have a photo, shown next to it on the **order form** and the **Admin page**. Click a photo to see it larger; click again (or Esc) to close.
- **Add one at a time:** Admin · Pricing > **Add photo** under the product (JPG or PNG; phone camera photos work too). **Change photo** / **Remove** are there as well.
- **Add from Excel:** put the picture in the **Item image** column on the product's row, either placed in the cell (Insert > Pictures > Place in Cell) or floating over the row with its top-left corner in that row. Then **Import Excel** (.xlsx). The preview shows each photo found, marked "Photo added" or "Photo replaced".
- Photos are shrunk automatically (about 5-100 KB each) and saved with the product, so everyone sees them. Rows without a picture keep their current photo.
- Export Excel does not include photos (Excel export can't hold pictures), so re-importing an exported sheet won't remove them.
- Upload `index.html` only. No rules change.

### v12.11: bigger, sharper photos
- Click any product photo to open it **almost full screen**, with the item number, name and price underneath. Close it with the **✕**, by clicking anywhere, or with Esc.
- Each photo is now saved at two sizes: a small one for the lists, so the order form stays fast, and a full-size one (up to 1400 px) that loads only when someone opens the photo.
- **Publish the new `firestore.rules`.** They add a `catalogPhotos` collection for the full-size copies (admins can change them, everyone signed in can view). Without the new rules, photos still save, but only the small copy, and the app tells you.
- Photos added before v12.11 only have the small copy. **Re-add them** (Add photo, or Import Excel again) to get the sharp full-size view.
- Upload `index.html` to GitHub **and** publish `firestore.rules` in Firebase.

## Version 13: scan delivery tickets and keep them with the PO

**Deploy:** upload `index.html` to GitHub **and publish the new `firestore.rules`** in Firebase (Firestore Database > Rules > Publish). The new rules add storage for ticket files under each PO.

### Scanning a delivery ticket
- On the **receiving screen** (Log receiving / Edit receiving) there's a **Delivery ticket** box:
  - **Scan with camera** opens the phone camera. Photograph the ticket flat, straight-on, filling the screen, in good light.
  - **Upload PDF or photo** takes a PDF (an emailed invoice, for example) or saved photos. You can pick several pages.
- The app reads the text **in your browser**. The first scan downloads the text reader (about 10 MB), so it takes longer. A PDF with real text is read instantly. A scanned PDF is read like a photo.
- The text appears in an editable **Ticket text** box (the "text document"). Fix anything misread, then click **Match to PO**. **Download as .txt** saves the text as a file.
- The comparison table shows each PO item: Ordered vs On ticket, with **Match / Short / Over**, **Check** (couldn't read it, so enter it by hand), and items **On ticket, not on PO**.
  - Quantities are double-checked with qty × price = total where possible. When the quantity itself is smudged, the app works it out from total ÷ price.
- **Fill received quantities from ticket** copies the readings into the Received boxes, adds "Per delivery ticket" notes on differences, and lists extra items in the receiving notes. Nothing is final until you click **Save receiving**.

### Ticket files are saved with the PO (history)
- When you save receiving, the ticket is stored **as a PDF with that PO**. An uploaded PDF is kept exactly as it was. Phone photos are combined into one PDF (one page per photo).
- Open it any time from:
  - **Order history** (the 📎 **Delivery ticket** button)
  - the receiving screen (**Ticket files**)
  - **PO Search** (open the PO row, then **Ticket files**)
- Scanning again later **adds** another file. Earlier files are kept. Only admins can delete files (in the Firebase console).
- The ticket text is saved with the PO too, and the receiving PDF notes that a ticket file is on record.
- Limits: 8 MB per file. Files are stored inside the free Firestore database in pieces, because Firebase file Storage needs a paid plan. A typical ticket is 0.1-1 MB, so the free 1 GB holds hundreds to thousands.
