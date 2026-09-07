# Delivery System Webpage

Static image and video delivery pages for Seaside Branding, plus the browser-based tools used to create new galleries.

This repository is intentionally framework-free. There is no build step, package manifest, application server, or shared JavaScript bundle. Each page is an HTML document containing its own markup, styles, and scripts. Published galleries are deployed as ordinary static files and reference their media with relative paths.

## Contents

- [How the repository works](#how-the-repository-works)
- [Entry points](#entry-points)
- [Published galleries](#published-galleries)
- [Gallery Studio](#gallery-studio)
- [Video Studio](#video-studio)
- [Generated package format](#generated-package-format)
- [Runtime dependencies](#runtime-dependencies)
- [Local development](#local-development)
- [Publishing a gallery](#publishing-a-gallery)
- [Editing a published gallery](#editing-a-published-gallery)
- [Analytics and browser storage](#analytics-and-browser-storage)
- [Error pages and hosting](#error-pages-and-hosting)
- [Maintenance checklist](#maintenance-checklist)
- [Known limitations](#known-limitations)

## How the repository works

The repository has three kinds of content:

1. The root `index.html` redirects visitors to `https://seasidebranding.com`.
2. The `develop/` pages are authoring applications. They accept local files, build a live preview, and download a complete gallery as a ZIP archive.
3. The named directories are published galleries. Their `index.html` files are the public page, and their adjacent `images/` or `media/` directory contains the assets displayed by that page.

Every URL is therefore a directory-style static URL:

```text
/afterwavesociety/       -> afterwavesociety/index.html
/develop/                -> develop/index.html
/develop/video/          -> develop/video/index.html
```

The pages do not communicate with a backend. Gallery metadata, styling, and media are either in the HTML itself, in local sibling files, or in third-party CDN resources loaded by the browser.

## Entry points

| Path | Purpose | Title |
| --- | --- | --- |
| `/` | Redirect to the Seaside Branding website | Redirecting... |
| `/develop/` | Create an image gallery | Seaside Branding Gallery Maker |
| `/develop/video/` | Create a mixed image/video gallery | Seaside Branding Video Gallery Maker |
| `/adoptaocasa/` | Published media delivery page | AOC |
| `/afterwavesociety/` | Published photo gallery | AFTERWAVE SOCIETY |
| `/ibsb-winter-fair/` | Published photo gallery | IBSB Winter Fair |
| `/petrecere-aurelia/` | Published photo gallery | Ziua Mea! |
| `/poze-extra-aurelia/` | Published photo gallery | Poze extra |
| `/yec/` | Published photo gallery | Youth Entrepreneurs Club |

The repository also contains root-level HTTP error documents: `400.html`, `401.html`, `403.html`, `404.html`, `405.html`, `408.html`, `410.html`, `429.html`, `500.html`, `501.html`, `502.html`, `503.html`, and `504.html`.

## Published galleries

Published pages are hand-authored or generated static output. The current media layout is:

| Directory | Asset folder | Current local asset count | Media type |
| --- | --- | ---: | --- |
| `adoptaocasa/` | `media/` | 4 | Video thumbnail plus video media |
| `afterwavesociety/` | `images/` | 107 | Images |
| `ibsb-winter-fair/` | `images/` | 58 | Images |
| `petrecere-aurelia/` | `images/` | 151 | Images |
| `poze-extra-aurelia/` | `images/` | 49 | Images |
| `yec/` | `images/` | 32 | Images |

Most photo galleries use this pattern:

```html
<div class="item">
	<a href="images/shot-1.jpg" data-pswp-width="2738" data-pswp-height="1825">
		<img src="images/shot-1.jpg" alt="Gallery Image" loading="lazy">
	</a>
</div>
```

The `href` is the full-size asset opened by PhotoSwipe. The `data-pswp-width` and `data-pswp-height` values must match the asset's intrinsic dimensions closely enough for the lightbox layout. `loading="lazy"` keeps large galleries from downloading every image immediately.

`adoptaocasa/` is a video-oriented hand-authored page. It uses `media/asset-1-thumb.jpg` for the grid preview and `media/asset-1.mp4` for playback in a full-screen video overlay.

Published pages commonly include:

- responsive CSS masonry columns;
- a client/event heading and a download link;
- PhotoSwipe for image viewing and individual downloads;
- a Seaside Branding footer with terms, privacy, and contact links;
- Tailwind CSS and Google Fonts loaded from CDNs.

## Gallery Studio

Open [`develop/index.html`](develop/index.html) through a local HTTP server. The application runs entirely in the browser.

### Workflow

1. Drop image files onto **Import Gallery**, or use the file picker.
2. Enter the client name and links/details in the editable heading fields.
3. Use **Studio settings** to adjust typography, colors, spacing, presets, favicon, download behavior, and the optional external download URL.
4. Add more files or remove individual assets from the preview.
5. Optionally enable **Scramble Images** and choose a seed. The same seed produces a repeatable order in the baked output.
6. Select **Bake Production Package**. The browser creates and downloads a ZIP file.

### Image processing

During import, images remain local browser object URLs for the preview. During baking, every image is drawn to a canvas and exported as a JPEG with:

- a maximum width or height of 2048 pixels;
- JPEG quality `0.7`;
- sequential names such as `images/shot-1.jpg`.

The original files are not uploaded anywhere by this application. The ZIP is assembled locally with JSZip.

### Editor state

The live state includes the asset list, theme, custom favicon, external download URL, button label, visibility of the download button, and scramble settings. Saved style presets are stored in browser `localStorage` under `studio_templates_final_v3`; they are local to the current browser profile and are not part of the repository.

The **Download All** button in the preview opens the configured external URL in a new tab. It is a link placeholder, not an archive service.

## Video Studio

Open [`develop/video/index.html`](develop/video/index.html) to create a mixed-media gallery.

It shares the image studio's theme controls, presets, editable heading, favicon handling, PhotoSwipe viewer, and ZIP export flow. Its media-specific behavior is different:

- image files are compressed to JPEGs using the same 2048px and quality `0.7` rules;
- video files keep their original bytes and extension;
- a thumbnail is captured from approximately one-third into the video, scaled to a maximum dimension of 900 pixels, and encoded as a JPEG;
- if a frame cannot be captured, the studio generates a simple video placeholder thumbnail;
- baked video files are written as `media/asset-N.<extension>` and thumbnails as `media/asset-N-thumb.jpg`;
- the published page uses inline video playback alongside PhotoSwipe image viewing.

The video studio does not currently expose the image-studio scramble controls.

## Generated package format

An image-studio export has this shape:

```text
gallery.zip
|- index.html
|- favicon.png
|- icon.png
`- images/
	 |- shot-1.jpg
	 |- shot-2.jpg
	 `- ...
```

A video-studio export has this shape:

```text
gallery.zip
|- index.html
|- favicon.png
|- icon.png
`- media/
	 |- asset-1.jpg
	 |- asset-1-thumb.jpg
	 |- asset-2.mp4
	 |- asset-2-thumb.jpg
	 `- ...
```

The generated `index.html` embeds the selected theme values and references the package's local assets with relative paths. The branding icons are copied into the root of the package. The generated page still loads Tailwind, PhotoSwipe, fonts, and Google Analytics from external URLs, so it is not fully offline despite containing its media.

## Runtime dependencies

There is no `npm install` step. The HTML pages load these browser dependencies from CDNs:

- Tailwind CSS: `cdn.tailwindcss.com`;
- PhotoSwipe 5.4.2 CSS and ES module;
- JSZip 3.10.1 in both authoring tools;
- Lucide icons in both authoring tools;
- Google Fonts for selected typography;
- Google Analytics `gtag.js` after consent.

An internet connection is required for the complete authoring experience and for the external assets used by published pages. CDN version changes should be treated as application changes because there is no lockfile or bundling layer.

## Local development

Use a static HTTP server from the repository root. On Windows with Python installed:

```powershell
py -m http.server 8000
```

Then open:

- `http://localhost:8000/develop/` for the image studio;
- `http://localhost:8000/develop/video/` for the video studio;
- `http://localhost:8000/afterwavesociety/` or another gallery path to inspect a published page.

Serving over HTTP is preferable to opening the files directly with `file://`: the authoring pages use ES module imports and browser APIs that are subject to origin and security rules.

There are no automated tests or build commands in this repository. A useful manual smoke test is:

1. Load each studio page without console errors.
2. Import one landscape image, one portrait image, and, for Video Studio, one playable video.
3. Open the lightbox, use individual download, edit the heading, and change a preset.
4. Bake the package and inspect the ZIP contents.
5. Extract the ZIP under the local server and verify that `index.html` loads all media.
6. Check the page at a narrow mobile viewport and a desktop viewport.

## Publishing a gallery

1. Create and test the gallery ZIP in the appropriate studio.
2. Extract it locally and verify `index.html` plus every referenced asset.
3. Choose a stable directory name using lowercase letters and hyphens, for example `client-event/`.
4. Add the extracted directory to the repository. Keep its `index.html` and media folder together.
5. Confirm relative paths, download links, image dimensions, and external URLs.
6. Commit and deploy through the repository's configured static hosting.
7. Open the deployed directory URL in a clean/private browser window and test the lightbox and downloads.

Do not rename or move an existing published directory without updating the URL shared with the client. Do not place unrelated media outside the gallery's own asset folder; generated HTML expects relative paths.

## Editing a published gallery

For a small content change, edit that gallery's `index.html` directly. Keep these values synchronized when adding or replacing images:

- the asset path in both `href` and `src`;
- the PhotoSwipe width and height attributes;
- the accessible `alt` text;
- the visible item order;
- the download link and visible button label.

For a significant redesign, regenerate the page from a studio export so the generated markup and media naming remain consistent. Remember that a new export can overwrite manual changes made to the generated HTML.

## Analytics and browser storage

The authoring tools and generated galleries show an analytics consent banner. The choice is saved in `localStorage` as `seaside_analytics_consent` with the value `accepted` or `declined`. Google Analytics is loaded only after acceptance, using measurement ID `G-JHWR7XVT1R`.

Studio style presets use the separate `studio_templates_final_v3` localStorage key. Clearing browser storage removes those custom presets and the analytics choice for that browser profile; it does not change repository files or published galleries.

## Error pages and hosting

The root-level numbered HTML files are static custom error responses. The hosting provider must be configured to use them for the matching HTTP status codes; simply having the files in the repository does not guarantee that every provider will select them automatically.

`CNAME` controls the custom domain for the static host. `LICENSE` contains the repository's licensing terms. Keep both under version control when changing deployment settings.

## Maintenance checklist

When adding or changing a gallery:

- use relative paths for assets inside the gallery directory;
- verify every media file exists with matching casing;
- provide accurate PhotoSwipe dimensions;
- use meaningful `alt` text;
- test lightbox, individual download, and the main download URL;
- test mobile and desktop layouts;
- check the browser console for failed CDN or media requests;
- review the generated ZIP before deployment;
- avoid committing temporary extracted files or source originals unless they are intended to be public.

When changing a studio:

- test both image import and ZIP generation;
- test a failed or unsupported media file;
- test custom fonts, presets, favicon, and external URL behavior;
- test generated output after extraction, not only the live preview;
- verify that any CDN or localStorage key change is intentional.

## Known limitations

- The application has no backend, authentication, access control, or server-side asset storage.
- Anyone who knows a published URL can request its static files; delivery privacy must be handled by the hosting or download provider.
- Download links point to externally configured URLs and are not generated by the gallery.
- CDN availability and third-party API changes can affect both authoring and published pages.
- Large imports can consume substantial browser memory because previews use object URLs and baking uses canvas and ZIP buffers.
- The studio's generated HTML is a template snapshot, not a reusable component shared with existing galleries.
- There is no automated regression suite; browser smoke testing is currently the verification method.