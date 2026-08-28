# Rebuilding Lineage 2 in Godot — Devlog

A small, markdown-based blog built with [Jekyll](https://jekyllrb.com/) and hosted on
GitHub Pages. Each post is one step of the journey of pulling apart an old Lineage 2
Interlude client and rebuilding its world in Godot 4.

## How to add a new post

1. Create a file in `_posts/` named `YYYY-MM-DD-some-title.md`.
2. Put this front matter at the top:

   ```yaml
   ---
   layout: post
   title: "My post title"
   date: 2026-08-28
   thumbnail: /assets/img/thumb-something.png
   description: "One or two lines that show up under the title on the home page."
   ---
   ```

3. Write the body in normal markdown. Drop images in `assets/img/` and link them like:

   ```markdown
   ![alt text]({{ "/assets/img/my-image.png" | relative_url }})
   ```

The home page (`index.html`) picks up every post in `_posts/` automatically, sorts them
in order, and shows the thumbnail + title + description.

## Preview locally (optional)

You don't need this to publish — GitHub builds the site on push. But if you want to see
it before pushing:

```bash
bundle install
bundle exec jekyll serve
```

Then open `http://localhost:4000/l2-godot-blog/`.

## Publishing

1. Push to GitHub.
2. In the repo, go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to *Deploy from a branch*, pick the
   `master` branch and the `/ (root)` folder, and save.
4. After a minute the site is live at `https://<your-user>.github.io/l2-godot-blog/`.

If you ever rename the repo, update `baseurl` in `_config.yml` to match.
