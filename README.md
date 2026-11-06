# FoodLoop — Full-stack campus food rescue application

This repository contains a responsive web frontend, Firebase Authentication, Cloud Firestore, callable Cloud Functions and Firestore security rules.

## Features and enforced constraints

- Student sign-up/sign-in using email and password; email verification is required.
- Backend checks the email domain against `COLLEGE_EMAIL_DOMAINS`.
- Roles: `student`, `cafeteria`, `safety_officer`, `admin`. Users cannot choose or self-assign a privileged role.
- Cafeteria operators submit listings in `pending_review` state.
- A separate safety reviewer (or admin) must approve/reject; the creator cannot approve their own listing.
- Student listing query only shows approved listings. Backend rechecks approval, expiry and remaining quantity.
- Reservation uses a Firestore transaction to prevent oversubscription.
- Reservations are not counted as impact. Only operator/admin handover confirmation increments handed-over portions.
- Pickup deadline is required, future-dated and limited to 7 days at listing creation.
- Quantity limited to 1–500 portions. Input fields are length-limited and validated server-side.
- Role changes, approvals and handovers produce audit log records.
- Firestore client writes are denied; writes go through validated Cloud Functions.
- Admin dashboard provides an illustrative demand estimate and stores forecasts. No live POS/weather/attendance/sensor connection is claimed.

## What you need

1. A Google account and a Firebase project.
2. Node.js 20 and Firebase CLI on a computer (or GitHub Codespaces).
3. Your institution's approved email domain(s).
4. A billing-enabled Firebase project for deploying Cloud Functions (Firebase plan requirements can change; check the Firebase console).

## A. Create Firebase project

1. Open https://console.firebase.google.com/ and create a project.
2. Add a **Web app** to the project. Copy its Firebase web config.
3. Enable **Authentication → Sign-in method → Email/Password**.
4. Enable **Authentication → Settings → Authorized domains** and add your GitHub Pages domain if you will host there.
5. Create **Firestore Database** in production mode.
6. In Project settings → Your apps → Web app, copy the config values.

## B. Configure frontend

Open `public/app.js`. Replace the `firebaseConfig` placeholders with your real web app config:

```js
const firebaseConfig = {
  apiKey: "…",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  appId: "…"
};
```

Firebase web config is not a server secret. Security is enforced by Authentication, Cloud Functions and Firestore rules. Never put service-account JSON or private keys in `public/`.

## C. Configure backend and deploy

Run these commands from the repository root in a terminal:

```bash
npm install -g firebase-tools
firebase login
firebase use --add
cd functions
npm install
cd ..
```

Copy `functions/.env.example` to `functions/.env` and replace the domain with your real college domain(s), comma-separated:

```text
COLLEGE_EMAIL_DOMAINS=youractualcollege.edu
```

Deploy:

```bash
firebase deploy --only firestore:rules,functions,hosting
```

The Firebase Hosting URL will be printed. Firebase Hosting is the simplest full-stack hosting option because it serves the site and connects to the Firebase backend.

## D. Bootstrap the first administrator (required)

No user can make themselves an admin. After creating your first account and verifying its college email:

1. Firebase Console → Authentication → Users → copy that account's UID.
2. Firestore Database → create collection `users`.
3. Create a document whose document ID is exactly that UID.
4. Add fields:
   - `uid` (string): same UID
   - `email` (string): verified admin email
   - `displayName` (string): administrator name
   - `role` (string): `admin`
   - `createdAt` (timestamp): current time

Then sign out and sign back in. The admin can assign roles using the admin dashboard. Create at least one `cafeteria` account and one `safety_officer` account. A cafeteria operator cannot approve their own listing; the separate safety reviewer must do that.

## E. GitHub Pages option

A GitHub Actions workflow is included at `.github/workflows/deploy-pages.yml`. It publishes the `public/` folder automatically whenever you push to `main`.

1. Push the complete repository to GitHub.
2. Open repository **Settings → Pages**.
3. Under **Build and deployment → Source**, choose **GitHub Actions**.
4. The workflow will run under the **Actions** tab and publish the website.
5. Add your published `YOUR-USERNAME.github.io` host under Firebase Authentication → Authorized domains.

The Firebase backend must still be deployed separately using the commands in section C. **Recommended:** use Firebase Hosting for a single-provider deployment, or use this GitHub Pages workflow for the public frontend plus Firebase for backend services.

## F. Suggested demo sequence

1. Sign up with a permitted college email and verify it.
2. Admin assigns one account `cafeteria`, another `safety_officer`.
3. Cafeteria user submits a food listing with quantity, deadline, allergens and handling notes.
4. Safety reviewer approves it. It then appears in student discovery.
5. Student reserves a portion. Backend transaction checks available quantity.
6. Cafeteria operator confirms physical handover. Only then does the admin impact count increase.

## Important limitations / production readiness

- This is a hackathon-ready starter, not a certified food-safety or commercial food-service system.
- Configure and verify the real college email domains before allowing reservations.
- A trusted institution must define food handling, allergen, temperature, time-window, traceability and disposal procedures. Software approval does not make unsafe food safe.
- Current impact unit is confirmed portions/meals, not kilograms or carbon emissions. Do not claim environmental savings without measured weights and a defensible methodology.
- Admin summary currently reads aggregate Firestore collections; for a large deployment, use aggregation counters/BigQuery and pagination.
- Add App Check, rate limits/abuse monitoring, privacy notice, data-retention policy, backups, monitoring and institution-approved incident response before production use.
- The forecast is a simple illustrative model. Calibrate with historical comparable-day data and validate against actual campus operations before operational use.
- For live use, have the institution's IT/security team review rules, domain policy, data protection and operational procedures.

## Repository structure

```text
public/
  index.html
  styles.css
  app.js
functions/
  index.js
  package.json
  .env.example
firestore.rules
firebase.json
README.md
```
