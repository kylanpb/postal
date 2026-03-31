# Postal

Postal is a static carousel preview app built for GitHub Pages. The admin creates draft posts in the browser, uploads clips to Cloudinary, and publishes post metadata as JSON files inside `posts/`. The viewer loads one of those JSON files with `viewer.html?post=<id>` and streams each clip directly from Cloudinary.

## Chosen architecture

Postal now uses option 1 from your list: static JSON files in the repo.

Why this is the best fit:
- It stays fully free and backend-free.
- GitHub Pages can serve JSON files directly, so every device gets the same published post data.
- Cloudinary handles the heavy part, which is streaming the actual video assets.
- Post URLs stay stable and simple: `viewer.html?post=<post-id>`.
- The published data is easy to version, review, and edit in git.

Why not the other two:
- Encoding 8 to 10 video URLs plus metadata into the URL gets unwieldy fast and makes links brittle.
- External JSON storage adds another service dependency and another failure point when a repo file already solves the problem cleanly.

## File structure

```text
index.html
viewer.html
styles/
  admin.css
  viewer.css
scripts/
  shared.js
  admin.js
  viewer.js
posts/
  example-post.json
.nojekyll
```

## Admin workflow

1. Open `index.html`.
2. Enter your Cloudinary `cloud name` and an `unsigned upload preset`.
3. Optionally enter a Cloudinary folder and your GitHub Pages base URL.
4. Create a post draft.
5. Upload videos or paste existing public Cloudinary video URLs.
6. Download the generated JSON.
7. Save that file as `posts/<post-id>.json`.
8. Commit and push to GitHub Pages.

The admin stores draft metadata in `localStorage` only for convenience while editing. Published viewer data comes only from files in `posts/`.

## Cloudinary setup

1. In Cloudinary, create a folder for Postal uploads if you want separation, for example `postal/`.
2. Create an unsigned upload preset for video uploads.
3. Restrict it to the target folder if possible.
4. Use the preset name and cloud name in the Postal admin.

Postal uploads directly from the browser to:

```text
https://api.cloudinary.com/v1_1/<cloud-name>/video/upload
```

That keeps the site static, but it also means the unsigned preset is intentionally public to the browser. Keep its permissions narrow.

## GitHub Pages deployment

1. Put this project in a GitHub repository.
2. Add a `.nojekyll` file at the repo root so GitHub Pages serves the files as plain static assets.
3. In GitHub, open `Settings -> Pages`.
4. Set the source to deploy from your main branch root, or your preferred Pages branch.
5. Wait for the site URL to appear.
6. Set that GitHub Pages URL in the admin so Postal can generate shareable viewer links.

Example:

```text
https://your-user.github.io/postal/viewer.html?post=launch-20260331-ab12
```

## Viewer behavior

- The viewer fetches `./posts/<post-id>.json`.
- Only the active video is played.
- Videos loop while active.
- Inactive videos are paused.
- Preload stays minimal: active and adjacent clips use `metadata`, others use `none`.
- Touch swipe, arrow buttons, keyboard arrows, and tap-to-play are supported.

## Trade-offs and limitations

- Publishing still requires a git commit because GitHub Pages cannot be written to directly from a static site without adding authenticated GitHub API logic.
- Unsigned Cloudinary uploads are convenient, but they should be scoped carefully because the preset is exposed to the browser.
- The viewer depends on public Cloudinary URLs, so unpublished or access-restricted Cloudinary assets will not play.
- Manual URL clips do not auto-detect duration until you publish or preview them elsewhere; uploaded clips do include Cloudinary metadata.
