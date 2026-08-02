# Content Manager setup (Decap CMS + DecapBridge)

The site now has a proper CMS at **`/cms/`** so Peachie can change photos, videos
and wording herself. It saves by committing to this repo, which triggers the
GitHub Pages workflow — edits are live about a minute later.

---

## 1. DecapBridge site — done

DecapBridge supplies the login system Decap CMS needs. (Decap normally relies on
Netlify Identity, which is being retired and never worked on GitHub Pages
anyway.)

The site **"Diverse Hair Designs By Peachie"** is registered, and its endpoints
are already in [cms/config.yml](cms/config.yml):

```yaml
auth_type: pkce
base_url: https://auth.decapbridge.com
auth_endpoint: /sites/47cee576-5549-4f1b-8e38-48dea534880d/pkce
auth_token_endpoint: /sites/47cee576-5549-4f1b-8e38-48dea534880d/token
```

This is DecapBridge's PKCE flow: the browser is redirected to DecapBridge to log
in and comes back with a short-lived token. No shared secret or personal access
token is ever stored in this repo or in the browser.

If the config ever needs regenerating, copy it from the DecapBridge dashboard
rather than editing those endpoints by hand.

## 2. Deploy

The CMS has to be live before anyone can log in — DecapBridge redirects back to
`https://diversehairdesignsbypeachienc.com/cms/`, so that page must exist. Push
to `main` and let the Pages workflow finish (~1 minute).

## 3. Invite Peachie

In the DecapBridge dashboard, invite her by email. She gets a link, sets a
password, and then logs in at:

**<https://diversehairdesignsbypeachienc.com/cms/>**

No GitHub account and no access tokens needed on her side.

---

## What she can edit

| Section in the CMS | What it controls | File written |
|---|---|---|
| **Photo Gallery** | The portfolio grid — add, delete, drag to reorder. **No limit**: as many before-and-after photos as she likes. First photo shows large; first 12 show before "Load More". | `gallery.json`, photos into `gallery/` |
| **Videos** | The "See it in Action" reel. Upload MP4s. **Capped at 6.** | `videos.json`, files into `videos/` |
| **Homepage** | Hero photo + photo strip, the sliding photo reel, all 10 service cards (photo, price, duration, description), reviews, the stylist photo and bio, contact details, newsletter text. | `site-data.json`, photos into `images/` |

## Limits

**Videos: maximum 6.** The reel is a 3-across grid, so 6 fills two tidy rows.
Adding a 7th and hitting Publish shows `Videos must have between 1 and 6
item(s).` and refuses to save. To swap one out she deletes a video first, then
adds the replacement. Note this means she is currently *at* the cap.

Enforced in `cms/config.yml` on the videos list:

```yaml
min: 1
max: 6
```

Decap only honours `max` when `min` is set as well — `max` on its own is
silently ignored. If the limit ever needs changing, both values have to stay
present. `index.html` also refuses to render more than 6, as a backstop for the
file being edited outside the CMS.

**Photos: no limit.** The gallery list has no `min`/`max`, so she can add every
before-and-after she wants. The homepage shows 12, then the rest behind "Load
More". Practical limits are page weight and repo size, not the CMS — see the
resizing note below.

## Things worth knowing

**Videos are stored in the repo.** GitHub hard-rejects any single file over
100 MB and starts complaining above ~1 GB total, so keep clips short — under
25 MB each is a good rule. The six original videos are still served from
Cloudflare Stream; each has a **Cloudflare Stream ID** field that should be left
alone. New videos just use the upload field and leave that blank.

**Photos should be resized before upload.** Some existing gallery files are 4–5 MB
straight off a phone, which makes the page slow to load. Anything around
1500px wide and under ~1 MB looks identical on screen and loads far faster.

**A few fields contain HTML.** The hero title, the "Our Story" title and the
newsletter heading use `<em>…</em>` for the italic words and `<br>` for a line
break. The CMS marks these with a hint. If those tags get deleted the text still
appears, it just loses the styling.

**Deleting a photo from the gallery does not delete the file.** It only removes
it from the list, so it can be added back. Files build up in `gallery/` over
time; they can be cleaned out with the media library's delete button.

## The old dashboard

The previous `admin.html` dashboard still exists and still works for text
content — it now links across to the Content Manager. Videos and the gallery
were removed from it, because they moved into their own files that it does not
load. Running both is fine as long as nobody edits the same section in both at
once. If you would rather retire it, delete `admin.html`; nothing else links to
it except the Content Manager button in its own header.

## How the site reads this data

`index.html` fetches `site-data.json`, `gallery.json` and `videos.json` on load
and applies each section independently. If a file is missing or malformed, the
hardcoded markup already in the page stays visible — the site degrades to its
current appearance rather than breaking.
