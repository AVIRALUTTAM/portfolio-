# Portfolio Setup Guide

Two files to deploy:

- `portfolio.html` — your portfolio (rename to `index.html` before deploying)
- `hire.html` — services + project brief form

Everything else (Firebase, Razorpay, deployment) is wired in already and just needs your keys.

---

## 1. Firebase setup (the "Google database" — stores project briefs)

You need this so the brief form on `hire.html` actually saves submissions somewhere.

### a) Create the project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and sign in with Google.
2. Click **Add project** → name it something like `aviral-portfolio` → disable Google Analytics (you don't need it).
3. Once created, click the **`</>`** (web) icon on the project home to register a web app.
4. Nickname it `portfolio` → **Register app**.
5. You'll see a `firebaseConfig` snippet. Copy it.

### b) Paste config into `hire.html`

Open `hire.html`, search for `REPLACE_ME`, and replace the entire `firebaseConfig` object near the bottom of the file with the snippet Firebase gave you. It looks like this:

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "aviral-portfolio.firebaseapp.com",
  projectId: "aviral-portfolio",
  storageBucket: "aviral-portfolio.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123:web:abc"
};
```

> ⚠️ The `apiKey` here is safe to expose publicly — it identifies your project, it's not a secret. Security comes from the rules below.

### c) Enable Firestore

1. In Firebase console sidebar: **Build → Firestore Database → Create database**.
2. Pick **Start in production mode** → choose a region close to you (e.g. `asia-south1` for Mumbai).
3. Once created, go to the **Rules** tab and paste this:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /project_briefs/{doc} {
      // Allow anyone to submit a brief, nobody to read them from the client.
      allow create: if request.resource.data.keys().hasAll(['name','email','tier','details'])
                    && request.resource.data.email is string
                    && request.resource.data.email.size() < 200
                    && request.resource.data.details.size() < 5000;
      allow read, update, delete: if false;
    }
  }
}
```

4. Click **Publish**.

That's it — briefs now save to a `project_briefs` collection. To read them, go to **Firestore Database → Data** in the Firebase console anytime.

### d) (Optional) Get notified by email when a brief arrives

The easiest path: in Firebase console, install the **"Trigger Email"** extension (Build → Extensions). Configure it to send an email to `aviraluttam@gmail.com` every time a new document is added to `project_briefs`. Takes ~5 minutes.

Alternative: use [Zapier](https://zapier.com/apps/firebase/integrations/email) Firebase → Gmail.

---

## 2. Razorpay setup (payments)

1. Sign up at [razorpay.com](https://razorpay.com) with your PAN + bank details.
2. Once verified, go to **Payment Pages** (or **Payment Links**) in the dashboard.
3. Create one link per service tier — set fixed amounts of ₹2,999 / ₹7,999 / ₹19,999.
4. After a client approves your quoted proposal, send them the appropriate Razorpay link by email.

If you want a **"Pay 50%"** button on the form-confirmation step later, tell me and I'll add the Razorpay Checkout JS button inline. For now, sending a link per quote is simpler and avoids exposing keys.

---

## 3. Deploy to the web (free)

### Easiest: Netlify drop

1. Rename `portfolio.html` → `index.html` so it becomes the homepage.
2. Go to [app.netlify.com/drop](https://app.netlify.com/drop).
3. Drag the entire folder (containing `index.html` and `hire.html`) onto the page.
4. You get a live URL instantly like `https://aviral-uttam.netlify.app`.
5. (Optional) Buy a domain like `aviraluttam.dev` from Namecheap (~$10/year) and point it to Netlify under Site settings → Domain management.

### Alternative: Vercel

Same idea — push the folder to a GitHub repo, then "Import Project" at [vercel.com](https://vercel.com).

### Alternative: GitHub Pages

1. Create a repo called `aviraluttam.github.io`.
2. Push both HTML files (rename portfolio → index).
3. In repo settings → Pages → enable. URL becomes `https://aviraluttam.github.io`.

---

## 4. Things to update before going live

Open `portfolio.html` and `hire.html` and edit:

- LinkedIn URL — currently `linkedin.com/in/aviral-uttam`. Confirm it matches your real handle.
- GitHub URL — currently `github.com/aviraluttam`. Confirm.
- Email — currently `aviraluttam@gmail.com`. Confirm.
- Replace the `firebaseConfig` `REPLACE_ME` block in `hire.html` (Section 1b above).
- Update Razorpay payment links once you have them (Section 2).

---

## 5. Testing checklist

Before sharing the URL:

- [ ] Open the live URL on phone + desktop, click every nav link.
- [ ] Submit a test brief through the hire form using a fake name.
- [ ] Confirm the brief appears in Firebase console under Firestore → `project_briefs`.
- [ ] Click "Email me" — confirms it opens your mail client.
- [ ] Click LinkedIn and GitHub buttons — confirm they go to your real profiles.

Once those all pass, you're ready to share the link in applications, on LinkedIn, in your email signature.
