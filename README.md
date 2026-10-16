# One to Ten

A one-page site: "Pick up a random number between 1 and 10". Each valid pick is recorded, and the visitor then sees a live histogram of every pick made so far, by everyone, from the beginning.

Hosting is free end to end:

- **GitHub Pages** serves the page at `https://YOUR-USERNAME.github.io/one-to-ten/`.
- **Firebase Firestore** stores the picks and pushes live updates to every open page. No sign-in is needed to pick a number.

Files:

| File | What it is |
|---|---|
| `index.html` | The whole page (markup, styles, logic). |
| `firebase-config.js` | Your Firebase project's web config. **You must fill this in.** |
| `firestore.rules` | Database security rules. **You must paste these into Firebase.** |

## 1. Create the database (Firebase, about 5 minutes)

1. Go to <https://console.firebase.google.com> and click **Add project**. Any name works (for example `one-to-ten`). You can turn Google Analytics off. Stay on the default no-cost plan.
2. In the left menu open **Build > Firestore Database** and click **Create database**.
   - Choose a location near your visitors (this can't be changed later).
   - Choose **Start in production mode**.
3. Open the **Rules** tab, delete what is there, paste the entire contents of `firestore.rules`, and click **Publish**.
4. Click the gear icon next to **Project Overview** > **Project settings**. Under **Your apps** click the web icon (`</>`), give the app a nickname, leave Firebase Hosting unchecked, and click **Register app**.
5. Copy the `firebaseConfig` values it shows (apiKey, authDomain, projectId, storageBucket, messagingSenderId, appId) into `firebase-config.js`, replacing the placeholders. You can find them again later under Project settings > Your apps > SDK setup and configuration > Config.

## 2. Publish the page (GitHub Pages, about 3 minutes)

1. On GitHub, create a new **public** repository named `one-to-ten` (any name works; it becomes part of the URL).
2. Upload the three files: `index.html`, `firebase-config.js`, `firestore.rules` (the rules file is just for your records; the page doesn't load it). Use **Add file > Upload files** on the repository page, or `git push`.
3. In the repository, open **Settings > Pages**. Under **Build and deployment**, set Source to **Deploy from a branch**, choose the `main` branch and the `/ (root)` folder, and click **Save**.
4. After a minute or two the page is live at `https://YOUR-USERNAME.github.io/one-to-ten/`. The Settings > Pages screen shows the exact link.

Pick a number yourself to confirm it works. The pick appears in the Firebase console under Firestore Database > Data (collection `picks`), and the running totals in `tally/counts`.

## Testing locally

The page uses JavaScript modules, so it must be served over HTTP rather than opened as a file. From this folder:

```
python3 -m http.server 8080
```

then open <http://localhost:8080>.

## How it stays correct under concurrent picks

Every pick is one atomic write batch: a new document in `picks` (the number and a server timestamp) plus `+1` on `tally/counts`. The `+1` is a server-side increment, so two people picking at the same instant both count; nothing is read-then-written on the client. Visitors subscribe only to the single `tally/counts` document, so each visit costs one database read no matter how many picks exist, and new picks reach every open page within a second or so.

The rules only let a write add exactly one pick (total +1 and one of n1..n10 +1), and never let anyone edit, delete, or bulk-download the log from the web.

## Free limits

As of October 2026, Firestore's free quota is 50,000 document reads, 20,000 writes, and 20,000 deletes per day, with 1 GiB of storage, on the first database in a project; GitHub Pages is free for public repositories with soft limits of 100 GB of bandwidth per month and a 1 GB site. For this page, each pick is 2 writes, and each visitor is 1 read plus 1 read per pick that happens while they have the page open. That is thousands of picks a day before anything is throttled. If the daily write quota is ever used up, the page says so and picks resume the next day.

Sources: [Firestore pricing](https://firebase.google.com/docs/firestore/pricing), [GitHub Pages limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits).

## Optional hardening

The Firebase web config is designed to be public; the rules protect the data. If you want to make sure only your site can use the key, go to <https://console.cloud.google.com/apis/credentials>, open the **Browser key (auto created by Firebase)**, and under **Website restrictions** add `https://YOUR-USERNAME.github.io/*`.
