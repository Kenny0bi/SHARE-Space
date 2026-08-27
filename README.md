# SHARE Space

Stories, Hope, and Real Experiences. A digital storytelling platform I built for
the Autism Program at Boston Medical Center, where parents, siblings, autistic
individuals, caregivers, and allies can submit letters and read letters from
others who understand what they're living through. Every letter gets reviewed
before it goes live, categories keep things organized by who's reading, and a
text to speech feature reads any letter aloud, since not everyone processes a
wall of text the same way.

This is a real, deployed site for a real hospital program, not a demo. The
Firebase project backing it is live.

## What I found and fixed in this pass

**A dead script reference.** `submit.html` loaded `js/submit.js`, a file that
doesn't exist anywhere in this repository and never did in any commit I could
find. The browser would 404 on it silently, no visible error, just a request
that goes nowhere. Removed the reference.

**A half built feature.** The "Expand Writing Area" function already existed
in `submit.html`'s JavaScript, complete with the logic to toggle a CSS class
and swap the button text, but the button itself was never added to the form,
and the `.expanded` CSS class it depended on was never defined. The letter
textarea told people "no limit, write as much as your heart wants" while
giving them a fixed 150px box with no way to make it bigger except manually
dragging the resize handle, which most people don't know is there. Added the
button and the CSS rule so the feature that was already written actually
works.

**Dead code and orphaned files.** A `.sample-btn` event listener that matched
zero elements on the page (the actual buttons use a different class,
`sample-letter-button`, and already work through inline `onclick` handlers,
so this listener never did anything). An empty, unreferenced `style.css`. A
`strangers.css` that nothing links to, since the page using that category was
renamed to `allies.html` and its stylesheet renamed to `allies.css` without
the old file being cleaned up. All removed.

**A few stray emoji in user facing alerts.** "Admin verified", "You are not
authorized", "You have been logged out due to inactivity", cleaned up to read
as plain, professional system messages.

Nothing here needed a redesign. The actual architecture, Firebase Auth for
admin login, Realtime Database for letters, a proper security rules setup
that only exposes approved letters publicly, is sound. These were small,
real gaps between what the code implied it could do and what it actually did.

## Key features

**`read.html`**: five identity based categories as clickable cards, leading
to `parents.html`, `siblings.html`, `autistic.html`, `caregivers.html`, and
`allies.html`. Each category page pulls only its own approved letters from
Firebase and offers the Read Aloud button, built on the browser's native
Web Speech API, no external service, no API key, just `SpeechSynthesisUtterance`
with a preference for a female voice if the browser has one available, and a
pause and resume toggle rather than only play and stop.

**`submit.html`**: category selection, optional name and email, a title and
the letter itself, a required consent checkbox, and floating prompt bubbles
("What made you smile this week?", "What helps you feel safe?") for anyone
staring at a blank page. Two sample letters ship inline for inspiration.
Submissions save to Firebase with `approved: false` and stay invisible to
the public until an admin reviews them, and an EmailJS notification fires to
the admin team the moment a new letter comes in.

**`login.html`** and **`admin.html`**: Firebase email and password auth,
gated by an explicit admin allowlist in the database (a valid login isn't
enough on its own, the user's UID also has to exist under the `admins` node),
a dashboard that separates pending letters from approved ones, checkbox based
bulk approve and bulk delete, and an automatic logout after five minutes of
no mouse movement, keystrokes, or clicks.

## Firebase setup

This project uses Firebase Authentication, the Realtime Database, and
Firebase Rules. Paste this into the Realtime Database's Rules tab:

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

Anyone can read a letter that's marked approved. Only an admin (someone whose
UID exists under the `admins` node) can read unapproved letters or write to
the database directly. Give someone admin access by adding their UID:

```json
"admins": {
  "FIREBASE_AUTH_UID": true
}
```

Find a user's UID under the Firebase Authentication panel after they've
created an account.

## Running this locally

There's no build step. Open `index.html` (or any page) directly in a browser,
or serve the folder with anything that speaks static files, `python3 -m
http.server` works fine. Firebase features work as-is since each page's
config is inline, no environment setup needed to view the site. Refresh after
any change.

## Deploying

```bash
npm install -g firebase-tools
firebase login
firebase deploy
```

Test locally before deploying. Firebase Hosting serves whatever's currently
committed.

## Troubleshooting

| Problem | Likely cause |
|---|---|
| Modal not opening | The modal's HTML container is missing, or `showLetterModal()` isn't wired to the right element |
| Letters not displaying | Check the Realtime Database directly, confirm `approved: true` is actually set on the letter |
| Admin login not working | Confirm the Firebase config block matches the project, and the account exists under Authentication |
| Blank admin dashboard | The logged in user's UID isn't listed under the `admins` node in the database |
| Styling looks off on one page | Check that page's specific stylesheet (`parents.css`, `siblings.css`, etc.), each category page has its own |

## Notes for whoever maintains this next

Test changes on a local copy before touching production. Keep the tone
gentle and accessible, this is for families in a hard moment, not a generic
form. Only approve letters that are respectful and genuine. Don't change the
Firebase rules unless you're confident in exactly what they do, they're the
only thing standing between "approved letters are public" and "everything in
the database is public." Don't rename or remove a category without checking
every page that references it, `submit.html`'s dropdown, the matching
`category` value stored in Firebase, and the corresponding `approved*`
section in `admin.html` all have to agree, which is exactly the kind of
mismatch that produced the orphaned `strangers.css` this pass cleaned up.

## Glossary

- **UID**: the unique ID Firebase assigns to an authenticated user, used to
  check admin access.
- **Modal**: the popup that shows a full letter when you click an envelope.
- **Firebase**: the backend platform powering authentication and the letter
  database, no separate server to run.
- **`.read` rule / `.write` rule**: Firebase Realtime Database rules that
  control who can view or modify data, enforced server side, not something a
  user can bypass from the browser.

## Wishlist

Letting people attach an image to their letter (the upload logic already
exists in `submit.html`'s JavaScript, Firebase Storage handling and file type
and size validation are all written, but there's no file input in the form
yet for someone to actually pick a file). An analytics view for admins (most
read letters, submission volume by category over time). Multi language
support. Better mobile layout. More voice and speed control on Read Aloud.

## Live site

[bmcautismfriendly.github.io/SHARE-Space](https://bmcautismfriendly.github.io/SHARE-Space/)

## Contact

obidelek19@gmail.com
