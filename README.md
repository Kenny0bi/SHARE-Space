# SHARE Space Website – Onboarding & Maintenance Guide

Welcome to the SHARE Space project! This guide will help you understand how to maintain, update, and build on this website. Whether you're fixing a bug or adding a feature, everything you need to know is right here.

---

## What is SHARE Space?

SHARE Space stands for **Stories, Hope, and Real Experiences** — a digital storytelling platform created for the Autism Program at Boston Medical Center.

It allows individuals in the autism community to:

- 📨 Submit heartfelt letters (anonymously or with names)  
- 📚 Read categorized and approved letters (Parents, Siblings, Autistic Individuals, Caregivers, Allies)  
- 🧠 Access accessibility features like **"Read Aloud"**  
- 🔐 Manage content through a secure **Admin Dashboard**

---

## Getting Started

### 1 Folder Structure Overview

```
SHARE-SPACE/
├── index.html
├── read.html
├── submit.html
├── login.html
├── admin.html
├── parents.html
├── siblings.html
├── autistic.html
├── caregivers.html
├── allies.html
│
├── css/
│   ├── global.css
│   ├── header.css
│   ├── footer.css
│   ├── tabs.css
│   ├── login.css
│   ├── admin.css
│   ├── read.css
│   ├── submit.css
│   ├── parents.css
│   ├── siblings.css
│   ├── autistic.css
│   ├── caregivers.css
│   └── strangers.css
│
├── images/
│   ├── autismlogo.png
│   ├── envelope-icon.png
│   ├── himg1.png
│   ├── himg3.png
│   └── (uploaded letter images)
│
├── scripts/
│   └── (optional: external JS if refactored)
│
└── firebase/
    └── firebaseConfig.js (or inline in HTML)
```

---

### 2 Firebase Setup

SHARE Space uses:

- 🔐 **Firebase Authentication**  
- 📂 **Firebase Realtime Database**  
- 🔒 **Firebase Rules**

#### 📜 Firebase Rules

Paste this into your Firebase Realtime Database "Rules" tab:

```json
{
  "rules": {
    "letters": {
      "$category": {
        ".read": "true",
        "$letterId": {
          ".read": "data.child('approved').val() === true || root.child('admins').child(auth.uid).exists()",
          ".write": "!data.exists() || root.child('admins').child(auth.uid).exists()"
        }
      }
    },
    ".read": "root.child('admins').child(auth.uid).exists()",
    ".write": "root.child('admins').child(auth.uid).exists()"
  }
}
```

---

### Giving Admin Access

Add a new admin UID to your Firebase Database like this:

```json
"admins": {
  "FIREBASE_AUTH_UID": true
}
```

>  You can find the UID under the Firebase Authentication panel after creating a new user.

---

##  Key Features & Pages

### `/read.html`

- Displays 5 identity-based categories as clickable cards  
- Leads to pages like `parents.html`, `siblings.html`, etc.

### `/submit.html`

- Users submit a letter with category, name (optional), and consent  
- Floating prompts offer encouragement  
- Submissions are hidden until approved

### `/login.html`

- Admins sign in using email/password  
- Redirects to admin dashboard upon success

### `/admin.html`

- Admin-only dashboard protected with Firebase auth  
- Toggle between `Pending` and `Approved` letters  
- Bulk approve/delete with checkboxes  
- Auto-logout after 5 minutes of inactivity

---

##  Troubleshooting Guide

| Problem                 | Solution                                                                 |
|-------------------------|--------------------------------------------------------------------------|
| Modal not opening       | Ensure modal HTML exists and `showLetterModal()` is properly defined     |
| Letters not displaying  | Check Firebase rules and confirm `approved: true` exists                 |
| Admin login not working | Check Firebase config and ensure credentials are valid in Auth panel     |
| Blank admin dashboard   | Ensure the user's UID is listed under the `admins` node in the database  |

---


##  Running Locally

To test the site on your local machine:

1. Open `index.html` or any HTML file directly in your browser.
2. If you make changes, simply refresh the page to see updates.
3. Firebase features will work as long as your Firebase config is correctly included in the HTML.

---

##  Firebase Config (Example)

Each HTML file includes a Firebase configuration block. Here’s a sample:

```js
// Inside <script type="module">
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "your-app.firebaseapp.com",
  databaseURL: "https://your-app-default-rtdb.firebaseio.com",
  projectId: "your-app",
  storageBucket: "your-app.appspot.com",
  messagingSenderId: "your-messaging-id",
  appId: "your-app-id"
};
```

---

##  Live Demo

If the site is deployed, include a link here:

[Visit SHARE Space Site](https://bmcautismfriendly.github.io/SHARE-Space/)

---

##  Admin Workflow Notes

- Admins are **notified automatically** when new letters are submitted.
- It is recommended to still **check the admin dashboard regularly** to review and approve pending letters.

---

##  Notes for Future Interns

###  Hey there,

Thank you for stepping in to continue this meaningful work. Here are some helpful tips to guide your journey with clarity and care.

####  What To Do

- Test changes on a local/dev copy before updating production  
- Keep tone and design gentle, accessible, and welcoming  
- Only approve respectful, thoughtful letters  
- Use the floating prompts to guide users  
- Take pride in small improvements — they matter  

####  What Not To Do

- Don’t change Firebase rules unless you're confident in what you're doing  
- Don’t rename or delete categories without full team discussion  
- Don’t push changes without notifying others on the team  

---

---

###  Making Edits to the Website

To make changes, you can:

1. Open the repository in VS Code or your preferred code editor.
2. Navigate to the `html` or `css` files you want to change.
3. Make changes and preview them locally if possible.
4. Only update the `firebase rules` if you're confident in what you're doing.
5. Push your changes or upload via GitHub.

>  If you're unsure, reach out (see email below)

---

###  Deploying the Site

If you're using Firebase Hosting:

1. Install Firebase CLI (only once):
   ```bash
   npm install -g firebase-tools
   ```

2. Login to Firebase:
   ```bash
   firebase login
   ```

3. Deploy the site:
   ```bash
   firebase deploy
   ```

>  Always test your changes locally before deploying!

---

###  Glossary

- **UID**: Unique ID used to identify an admin in Firebase.
- **Modal**: A popup window on the screen (e.g., letter preview).
- **Firebase**: A backend platform that powers this website's login and data.
- **.read rule**: Firebase rule that controls who can view data.
- **.write rule**: Firebase rule that controls who can submit or change data.

---

###  Admin Login Behavior

- Admins are required to log in with their email and password.
- After 5 minutes of inactivity, they will be automatically logged out for security reasons.
- If you’re logged out unexpectedly, try refreshing the page or logging in again.

---

###  Where to Check if Something Isn't Working

- **Letters not showing**: Check Firebase Database – is `approved: true` set?
- **Login not working**: Confirm the user is listed under Firebase > Authentication
- **Styling broken?**: Open the correct `.css` file (e.g., `parents.css`) and look for a typo
- **Modal not opening**: Make sure the `<div class="modal">` exists in the HTML and script is linked

---


##  Useful Link

- [Firebase Console](https://console.firebase.google.com/)    

---

##  Future Wishlist

- Allow users to upload images with their letters  
- Add an analytics dashboard (e.g., most read letters, submission volume by category)  
- Build multi-language support  
- Improve mobile UI responsiveness  
- Enhance Read Aloud (voice options, speed controls)

---

_If all fails and you need help with anything, please reach out to me - obidelek19@gmail.com. Goodluck!_

