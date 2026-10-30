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
