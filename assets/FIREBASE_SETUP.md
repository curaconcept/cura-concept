# Firebase + Firestore setup (Contact form)

This site is static (GitHub Pages), so the contact form writes inquiries to **Firestore**.

## 1) Paste your Firebase config

In `index.html`, search for `const firebaseConfig = {` and replace the placeholder values with the `firebaseConfig` object from **Firebase Console → Project settings → Your apps → Web app**.

## 2) Firestore rules (public write, no reads)

In **Firebase Console → Firestore Database → Rules**, replace rules with something like:

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /inquiries/{docId} {
      allow read: if false;

      allow create: if
        request.time < timestamp.date(2100, 1, 1) &&
        request.resource.data.keys().hasOnly([
          "fullName",
          "email",
          "projectType",
          "message",
          "createdAt",
          "userAgent",
          "referrer",
          "page"
        ]) &&
        request.resource.data.fullName is string &&
        request.resource.data.fullName.size() >= 2 &&
        request.resource.data.fullName.size() <= 100 &&
        request.resource.data.email is string &&
        request.resource.data.email.size() >= 5 &&
        request.resource.data.email.size() <= 200 &&
        request.resource.data.projectType is string &&
        request.resource.data.projectType.size() >= 2 &&
        request.resource.data.projectType.size() <= 20 &&
        request.resource.data.message is string &&
        request.resource.data.message.size() >= 10 &&
        request.resource.data.message.size() <= 4000;

      allow update, delete: if false;
    }
  }
}
```

Notes:
- These rules **block all reads**, so submissions stay private.
- The site’s **honeypot** and “minimum time on page” checks are implemented client-side (they help reduce spam but are not a guarantee).

## 3) Firestore index/collection

The form writes to the collection:
- `inquiries`

You’ll see new documents appear in Firestore when the form is submitted.
