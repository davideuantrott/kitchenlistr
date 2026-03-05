# Feature Spec: Recipe Image Upload with Client-Side Compression

> **Status:** Planned — not yet implemented
> **Effort:** Medium (~2–3 hours)
> **Prerequisite:** Enable Firebase Storage in the Firebase Console for this project

---

## Summary

Recipes currently support images via a manually entered URL only. This adds a real file upload so users can take or choose a photo directly from their device. Images are compressed in the browser before upload — no external libraries needed.

---

## How Compression Works (Canvas API)

```
File selected by user
  → FileReader reads it as a data URL
    → draw onto a hidden <canvas>, scaled to max 1200px on longest side
      → canvas.toBlob('image/jpeg', 0.82) produces a compressed Blob
        → upload Blob to Firebase Storage
          → get download URL → store in Firestore imageUrl field
```

**Target output:** ≤200 KB for a typical food photo (down from 3–8 MB on modern phones).

| Input | Output |
|---|---|
| 4000×3000px, 4 MB | ~1200×900px, ~120–180 KB |
| 1920×1080px, 1.5 MB | ~1200×675px, ~80–120 KB |
| Already ≤1200px | Unchanged |

---

## Implementation Steps

### 1. Firebase Storage SDK — `index.html` (~line 5316)
Add to the existing Firebase CDN imports:
```javascript
import { getStorage, ref, uploadBytes, getDownloadURL, deleteObject }
  from "https://www.gstatic.com/firebasejs/10.8.0/firebase-storage.js";
```
Initialise after `db` and `auth`:
```javascript
const storage = getStorage(app);
```

### 2. `compressImage()` utility — `index.html` (utility functions section)
```javascript
function compressImage(file, maxPx = 1200, quality = 0.82) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = e => {
            const img = new Image();
            img.onload = () => {
                const scale = Math.min(1, maxPx / Math.max(img.width, img.height));
                const canvas = document.createElement('canvas');
                canvas.width  = Math.round(img.width  * scale);
                canvas.height = Math.round(img.height * scale);
                canvas.getContext('2d').drawImage(img, 0, 0, canvas.width, canvas.height);
                canvas.toBlob(
                    blob => blob ? resolve(blob) : reject(new Error('Canvas toBlob failed')),
                    'image/jpeg', quality
                );
            };
            img.onerror = reject;
            img.src = e.target.result;
        };
        reader.onerror = reject;
        reader.readAsDataURL(file);
    });
}
```

### 3. Recipe form UI — `index.html` (~line 4532)
Replace the plain `<input type="url">` with a combined control:
- `<input type="file" accept="image/*">` — opens camera/gallery on mobile
- Keep `<input type="url" id="recipe-image-url">` for manual URL entry (shows/hides based on whether a file is chosen)
- Small `<img id="recipe-image-preview">` thumbnail shown once a file/URL is set

Backward compatible — `imageUrl` in Firestore remains a plain string URL either way.

### 4. `uploadRecipeImage()` — `index.html` (near `saveRecipe()`)
```javascript
async function uploadRecipeImage(file, recipeId) {
    const compressed = await compressImage(file);
    const storageRef = ref(storage, `recipes/${currentUser.uid}/${recipeId}.jpg`);
    await uploadBytes(storageRef, compressed, { contentType: 'image/jpeg' });
    return await getDownloadURL(storageRef);
}
```
One file per recipe at `recipes/{uid}/{recipeId}.jpg` — re-uploading overwrites (no orphaned files).

### 5. Update `saveRecipe()` — `index.html` (~line 6393)
```javascript
// Before existing imageUrl line:
let imageUrl = document.getElementById('recipe-image-url').value.trim();
const fileInput = document.getElementById('recipe-image-file');
if (fileInput?.files[0]) {
    showSyncStatus('syncing', 'Uploading photo…');
    imageUrl = await uploadRecipeImage(fileInput.files[0], id);
}
```

### 6. `deleteRecipeStorageImage()` — `index.html`
```javascript
async function deleteRecipeStorageImage(recipeId) {
    try {
        await deleteObject(ref(storage, `recipes/${currentUser.uid}/${recipeId}.jpg`));
    } catch (_) { /* file may not exist — ignore */ }
}
```
Call in `deleteRecipeFromFirebase()` and before uploading a replacement image.

---

## Firebase Console Steps (manual, one-time)

1. **Enable Storage:** Firebase Console → Build → Storage → Get Started
2. **Add security rules:**

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /recipes/{uid}/{recipeId} {
      allow read: if request.auth != null;
      allow write: if request.auth.uid == uid;
      allow delete: if request.auth.uid == uid;
    }
  }
}
```

---

## Verification Checklist
- [ ] File picker opens on click; camera offered on mobile
- [ ] Preview thumbnail appears immediately after selection
- [ ] On save: network tab shows upload to `firebasestorage.googleapis.com`; payload ≤200 KB
- [ ] Firestore `imageUrl` field contains a Storage download URL
- [ ] Recipe card and modal display the uploaded image
- [ ] Re-uploading replaces the old file (check Firebase Storage browser)
- [ ] Deleting a recipe removes the Storage file
- [ ] Manually pasting a URL still works when no file is selected
