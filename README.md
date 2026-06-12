<h1 align=center>Brot Theme | <a href="https://adityatelange.github.io/hugo-PaperMod/" rel="nofollow">Inspired by PaperMod</a></h1>

<h2 align=center>95% based on Papermod, which is based on Hugo-Paper</h4>
<b> Adjustments Made <b>

Rather than bounding articles based on the size of the main div, we let elements bound themselves.
This change is made primarily for the purpose of enabling the use of a CSS grid to display images, which would be larger than the typical bounded \<article\> size of the basic Hugo-Paper theme.

Branding and other things will be changed to better differentiate between this and the source material, but everything below this line is based on the README.md from papermod. Check them out! They did all the hard work. I'm just editing margins and max-widths.

\<picture\> elements now will natively support multiple image formats based on the {figure} shortcode
There's a new hamburger menu so that the site looks better on mobile devices.

Increasingly this theme is straying away from the hugo-PaperMod it was based on. Going to move to make this more of a media friendly site with large images, a more engaging /index.html file with the goal to make it a great personal site theme for photographer/developers.

## Auto-generated preview thumbnails

List and preview contexts (home, section lists like `/posts/` and `/gallery/`, tag pages, the gallery card stack) do **not** serve the original images. Hugo generates downscaled derivatives at build time and the templates emit responsive `<picture>` markup instead. Single content pages are untouched and still serve the originals (including any hand-made `.avif`/`.jxl` variants next to them).

### How it works

All of the logic lives in one partial: [`layouts/partials/preview_image.html`](layouts/partials/preview_image.html). It is used by:

- `layouts/partials/cover.html` — whenever a cover is rendered with `IsSingle: false` (i.e. every list/preview render)
- `layouts/_default/gallery.html` — for the decorative card-stack image behind each gallery card

For each image the partial emits:

- a WebP `<source>` with a `srcset` at widths **360 / 600 / 900 / 1200** (capped at the original's width)
- an `<img>` with a same-widths `srcset` in the image's **original format** (JPEG stays JPEG) as fallback
- a downscaled 720px `src` fallback for ancient browsers, plus `width`/`height` attributes to prevent layout shift

Derivatives are cached in `resources/_gen` — cache that directory in CI or every deploy re-encodes from scratch.

### Requirements / opt-out

- Images must be resolvable as Hugo resources. The parent site mounts `static` into `assets` for this (see `module.mounts` in its config); page-bundle images work too.
- WebP encoding requires Hugo **extended**. On non-extended builds the partial still emits resized originals, just without the WebP source.
- External URLs and non-raster formats (SVG, …) fall back to the raw `src` untouched.
- Setting `cover.responsiveImages = false` (per page or site-wide) bypasses thumbnail generation for covers.

### Changing the output format (AVIF, JPEG XL, …)

Format choices are confined to `preview_image.html` — the two `$res.Resize` calls and the `<source type="...">` line:

- **AVIF**: Hugo can encode it natively (verified on Hugo 0.163 extended) — change the resize format string from `webp` to `avif` (e.g. `printf "%dx avif" $w`) and the source type to `image/avif`, or add it as an additional `<source>` above the WebP one.
- **JPEG XL**: *not* supported by Hugo's image processing as a target format. Serving JXL previews would require pre-generating `.jxl` files outside Hugo, the same way the `figure` shortcode already picks up hand-made `.jxl`/`.avif` siblings.

Currently WebP + JPEG fallback is the deliberate choice: universal browser support with one extra encode per width.