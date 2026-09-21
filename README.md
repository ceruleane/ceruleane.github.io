# Your site

A small, fast personal site for writing: plain Markdown in, static HTML out.
It is built with [Hugo](https://gohugo.io), hosted for free on GitHub Pages, and renders math
with KaTeX at build time, so readers get plain HTML and no JavaScript.

```
hugo.toml               site settings: your name, description, links, menu
content/_index.md       the home / About page text
content/writings/       one folder per post: index.md plus any images
content/scribbles.md    the Scribbles page: one paragraph per quote or thought
assets/css/main.css     all the styling, in one file
layouts/                the HTML templates (you rarely need to touch these)
.github/workflows/      builds and publishes the site whenever you push
```

The rest of this file is the walkthrough, in order:

1. [Run it on your Mac](#1-run-it-on-your-mac)
2. [Make it yours](#2-make-it-yours)
3. [Write a post](#3-write-a-post)
4. [Put it on GitHub](#4-put-it-on-github)
5. [Connect your own domain](#5-connect-your-own-domain)
6. [Everyday workflow](#6-everyday-workflow)
7. [Odds and ends](#7-odds-and-ends)

---

## 1. Run it on your Mac

You need one program, Hugo (version 0.166.0 or newer). Pick either route:

**Installer (simplest).** Download `hugo_0.166.0_darwin-universal.pkg` from
<https://github.com/gohugoio/hugo/releases/tag/v0.166.0>, double-click it, and follow the prompts.

**Homebrew.** If Homebrew works for your user account:

```bash
brew install hugo
```

(On the Mac this was set up on, Homebrew's folder belonged to a different user account, so
`brew install` could not write to it. The installer route avoids that entirely.)

Then, in a terminal:

```bash
cd "/Users/personal/Claude Code/my-site"
hugo server -D
```

Open <http://localhost:1313>. The page reloads by itself whenever you save a file.
`-D` means "include drafts", which you want while writing. Press `Ctrl-C` to stop the server.

## 2. Make it yours

Open `hugo.toml` and change the top block:

- `title` is the name shown top-left and in browser tabs.
- `author` and `description` are used for the RSS feed and link previews.
- `[[params.social]]` entries become the links in the footer. Add, remove, or rename them freely.

Open `content/_index.md` and write your intro. That file *is* the home page and the "About" link.
Its first paragraph is shown slightly larger, so a short greeting works well there.

The two posts under `content/writings/` are samples. Delete their folders when you are done looking at them.

## 3. Write a post

Every post is a folder containing an `index.md` and whatever images it uses. To start one:

```bash
hugo new content writings/my-first-idea/index.md
```

That creates `content/writings/my-first-idea/index.md` with this at the top (the "front matter"):

```yaml
---
title: "My First Idea"
date: 2026-09-20T14:05:00+09:00
description: ""            # one line shown under the title in the list. Optional.
draft: true                # drafts show up in `hugo server -D` but are never published
---
```

Write below the `---` in Markdown. When it is ready, change `draft: true` to `draft: false`
(or delete the line). The folder name becomes the URL: `/writings/my-first-idea/`.

### Images and charts

Save the image file into the post's folder, then reference it by name. The text in square
brackets becomes the caption under the figure:

```markdown
![Monthly returns, 2020 to 2026](returns.png)
```

PNG, JPG, SVG, and GIF all work. In VS Code you can paste an image from the clipboard straight
into the Markdown file; it saves the file next to the post and inserts the line for you.

For an interactive chart (Plotly, Bokeh, Altair), export it as an HTML file into the post's
folder and embed it. With Plotly, `include_plotlyjs="cdn"` keeps the file small:

```python
fig.write_html("plot.html", include_plotlyjs="cdn")
```

```html
<iframe src="plot.html" width="100%" height="420"></iframe>
```

### Math

Use LaTeX between dollar signs. `$...$` is inline, `$$...$$` on its own lines is a display equation.

```markdown
The estimate is $\hat{\beta} = (X^\top X)^{-1} X^\top y$ with variance

$$
\operatorname{Var}(\hat{\beta}) = \sigma^2 (X^\top X)^{-1}
$$
```

Anything KaTeX supports works, including `\begin{aligned}` blocks. If you need a literal dollar
sign in prose, write `\$`. A typo in an equation shows up in red on the page rather than
breaking the build.

### Everything else

Standard Markdown: `## Heading`, `**bold**`, `*italic*`, `[text](url)`, `> quote`, `- list`,
tables, and fenced code blocks with a language name for syntax colouring. Footnotes are `[^1]`
in the text and `[^1]: note` anywhere below. The sample post "Math, charts, and code" shows all of it.

### Scribbles

`content/scribbles.md` is one page for quotes, one-line thoughts, and the like. There is nothing
to set up: each paragraph in the file becomes one scribble, shown with a thin line between them.
Add a new one by typing a blank line and then the text, at the top if you want newest first.
An attribution can just be part of the line:

```markdown
We suffer more often in imagination than in reality. — Seneca
```

## 4. Put it on GitHub

You need a GitHub account (<https://github.com/signup>). Then:

1. **Create an empty repository** at <https://github.com/new>. Name it `YOURUSERNAME.github.io`
   so the site gets the clean address `https://YOURUSERNAME.github.io/` before you add a domain.
   Any other name also works; the site would then live at `https://YOURUSERNAME.github.io/REPO/`.
   Keep it **Public** (Pages on private repositories needs a paid plan). Do not tick "add a README".

2. **Turn on Pages.** In the new repository: Settings → Pages → under "Build and deployment",
   set **Source** to **GitHub Actions**. Nothing else to choose.

3. **Push the site.** In the terminal, inside the `my-site` folder:

   ```bash
   git add .
   git commit -m "Initial site"
   git remote add origin https://github.com/YOURUSERNAME/YOURUSERNAME.github.io.git
   git push -u origin main
   ```

   If you have the GitHub CLI signed in, steps 1 and 3 collapse into:

   ```bash
   git add . && git commit -m "Initial site"
   gh repo create YOURUSERNAME.github.io --public --source=. --push
   ```

4. **Watch it build.** The Actions tab shows a run called "Build and deploy site". It takes
   about a minute. When it is green, the site is live at `https://YOURUSERNAME.github.io/`.

If the first run failed because Pages was not switched on yet, switch it on and click
"Re-run all jobs".

## 5. Connect your own domain

### Buy the domain

Any registrar works. Two with fair prices and no upsells:

- **Cloudflare Registrar** (<https://www.cloudflare.com/products/registrar/>) sells at wholesale
  cost, roughly US$10 a year for a `.com`. Needs a free Cloudflare account.
- **Porkbun** (<https://porkbun.com>) is similar and simpler if you do not want a Cloudflare account.

Turn on auto-renew and WHOIS privacy (both are usually free and on by default). Ten minutes
after paying you can manage DNS records in the registrar's dashboard.

### Point the domain at GitHub

In your registrar's DNS settings, add these records (delete any default A or CNAME records
for `@` and `www` first). `@` means the bare domain.

| Type  | Name | Value                     |
|-------|------|---------------------------|
| A     | @    | 185.199.108.153           |
| A     | @    | 185.199.109.153           |
| A     | @    | 185.199.110.153           |
| A     | @    | 185.199.111.153           |
| CNAME | www  | YOURUSERNAME.github.io    |

Those four IPs are GitHub's; the current list is always at
<https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site>.
On Cloudflare, set each record's proxy status to **DNS only** (grey cloud, not orange).

### Tell GitHub about it

1. Repository → Settings → Pages → **Custom domain**: type `yourdomain.com` (no `www`, no `https://`) and Save.
2. GitHub checks the DNS. This can take a few minutes or up to an hour the first time. When the
   check passes, tick **Enforce HTTPS**. (If the box is greyed out, wait and reload.)
3. Go to the Actions tab and run "Build and deploy site" once more (Run workflow → Run workflow),
   so the site is rebuilt with the new address baked into its links and RSS feed.

Now `https://yourdomain.com` serves the site and `www.yourdomain.com` redirects to it.

Optional but worthwhile: in your GitHub profile settings → Pages → "Add a domain", verify the
domain. That stops anyone else from ever pointing it at their own Pages site.

## 6. Everyday workflow

```bash
hugo new content writings/some-idea/index.md   # start a post
hugo server -D                                 # preview at http://localhost:1313
```

Write, look, adjust. When it is ready, set `draft: false`, then:

```bash
git add .
git commit -m "Post: some idea"
git push
```

About a minute later it is live. For a quick typo fix you can also edit the file directly on
github.com (pencil icon); the site rebuilds on its own. If you prefer buttons to commands,
GitHub Desktop or VS Code's Source Control panel do the same commit-and-push.

## 7. Odds and ends

- **RSS feed** is at `/index.xml` (linked in the footer). Readers can subscribe with any feed app.
- **Dark mode** follows the reader's system setting. If your charts have white backgrounds and
  look harsh in dark mode, either export them with transparent backgrounds or delete the
  `@media (prefers-color-scheme: dark)` blocks in `assets/css/main.css` for a light-only site.
- **Font, colours, column width** are the variables at the top of `assets/css/main.css`. The site
  uses the system sans-serif font; a serif stack sits commented out next to it if you prefer that.
- **Menu** items are the `[[menus.main]]` entries in `hugo.toml`. To add a standalone page
  (say, a reading list), create `content/reading.md` with a `title:` and add a menu entry with
  `pageRef = '/reading'`.
- **Updating Hugo**: install the newer version locally, then change `HUGO_VERSION` in
  `.github/workflows/deploy.yml` to match.
- **Something failed on GitHub?** The Actions tab shows the log; the error message names the
  file and line. Running `hugo` locally in the site folder reproduces the same build.
