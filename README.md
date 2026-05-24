# Mariki Family Portal — SCEM System

Vanilla HTML + CSS + JavaScript + Firebase. No frameworks.

## Setup
1. Create a Firebase project at https://console.firebase.google.com
2. Enable **Authentication** → Email/Password + Google.
3. Enable **Firestore Database** (start in production mode, then apply rules below).
4. Enable **Storage**.
5. Open `js/firebase.js` and replace the `firebaseConfig` block with your project's keys (Project settings → SDK setup → Web).
6. Deploy as a static site: drag the folder into Netlify, push to GitHub Pages, or `firebase deploy --only hosting`.

## Bootstrapping the first admin
First user registers normally → status = `pending`. In the Firebase Console, open Firestore → `users/{uid}` and manually set:
```
role: "superadmin"
status: "approved"
```
That user can now approve everyone else from `admin.html`.

## Suggested Firestore rules
```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    function isSignedIn() { return request.auth != null; }
    function isApproved() {
      return isSignedIn() &&
        get(/databases/$(db)/documents/users/$(request.auth.uid)).data.status == "approved";
    }
    function role() { return get(/databases/$(db)/documents/users/$(request.auth.uid)).data.role; }
    function isAdmin() { return isSignedIn() && (role() == "admin" || role() == "superadmin"); }
    function isTreasurer() { return isSignedIn() && role() == "treasurer"; }

    match /users/{uid} {
      allow read: if isApproved() || request.auth.uid == uid;
      allow create: if isSignedIn() && request.auth.uid == uid;
      allow update: if request.auth.uid == uid || isAdmin();
      allow delete: if isAdmin();
    }
    match /contributions/{id} {
      allow read: if isApproved();
      allow create, update, delete: if isAdmin() || isTreasurer();
    }
    match /events/{id} {
      allow read: if isApproved();
      allow write: if isAdmin();
    }
    match /announcements/{id} {
      allow read: if isApproved();
      allow write: if isAdmin();
    }
    match /media/{id} {
      allow read: if isApproved();
      allow create: if isApproved();
      allow delete: if isAdmin();
    }
  }
}
```

## Suggested Storage rules
```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /profiles/{uid}/{file=**} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == uid;
    }
    match /media/{file=**} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }
  }
}
```

## Files
- `index.html` landing
- `login.html` / `register.html`
- `dashboard.html` member dashboard (gated by approval)
- `admin.html` admin panel (admin/superadmin only)
- `members.html` directory
- `profile.html` self-edit profile
- `contributions.html` TZS tracking
- `events.html` event management
- `gallery.html` media uploads
- `js/firebase.js` Firebase init (EDIT YOUR KEYS HERE)
- `js/common.js` shared helpers
- `css/styles.css` glassmorphism gold/dark theme
