# TerraForRhino — website

Static site. No build step, no dependencies, no framework. One HTML file plus a `media/` folder.

```
/
├── index.html      ← the whole site (layout + a CONFIG block you edit)
├── README.md
└── media/          ← your images and videos
    ├── hero.webp
    ├── paths.mp4
    ├── paths.webp
    └── ...
```

## Publish

1. Push this repo to GitHub (public).
2. Settings → Pages → Source: **Deploy from a branch** → `main` / `/ (root)` → Save.
3. Wait ~1 minute. Site is live at `https://<user>.github.io/<repo>/`.

Every `git push` to `main` republishes. That is the entire deployment pipeline.

## Adding content

Open `index.html`, find the `const SITE = {` block near the bottom. Everything editable lives there:

- `links`: Food4Rhino page, forum, email, socials (`""` hides one).
- `tools`: the toolbox cards. `icon` is one of `pin, terrain, path, platform, stairs, wall, sculpt, scatter, sun, library`; `tint` is `sky, honey, leaf` or `sun`.
- `steps`: the "Five moves" strip. Each step takes a looping MP4 plus a poster still:

```js
{ title:"Grade paths", body:"One sentence.",
  media:{ type:"video", src:"media/paths.mp4", poster:"media/paths.webp" }, alt:"What the clip shows" }
```

- `lessons`, `releases`, `band`: tutorials, changelog and the closing blurb.

If a browser can't play a video, the poster still is shown instead. Drop the files in `media/`, commit, push. Done.

## Media rules (this is where the money and the load time go)

**Never ship a GIF.** A 10-second screen recording is ~25 MB as a GIF and ~1.5 MB as an MP4, and it looks better. Convert everything:

```bash
# GIF → MP4
ffmpeg -i in.gif -movflags faststart -pix_fmt yuv420p \
  -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" out.mp4

# screen recording → web MP4 (1280 wide, no audio)
ffmpeg -i in.mp4 -an -vf "scale=1280:-2,fps=24" \
  -c:v libx264 -crf 26 -preset slow -movflags faststart out.mp4

# poster frame (shown before the video loads)
ffmpeg -i out.mp4 -vframes 1 -q:v 4 out.jpg

# stills → WebP, max 1600px
ffmpeg -i in.png -vf "scale='min(1600,iw)':-1" -q:v 78 out.webp
```

Targets: MP4 under 3 MB, WebP under 300 KB.

Optimise **before** the first commit. Git keeps every version of every binary forever — a 40 MB GIF committed once and deleted later still lives in the repo history.

## Custom domain (optional)

1. Buy the domain (Cloudflare Registrar or Porkbun, roughly £9–12/year at cost).
2. DNS: four `A` records for the apex → `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`. Plus a `CNAME` for `www` → `<user>.github.io`.
3. GitHub → Settings → Pages → Custom domain → enter it → tick **Enforce HTTPS**.

## Cost

Hosting: £0. GitHub Pages is free for public repos, with soft limits of 1 GB site size and 100 GB bandwidth per month — far beyond what this needs if the media rules above are followed.
