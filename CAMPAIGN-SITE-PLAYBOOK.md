---
document: campaign-site-playbook
purpose: >
  Complete transferable procedure for building a public, free, spectator-facing
  website for a tabletop campaign out of a shared Obsidian vault.
  Written to be read by an AI assistant starting with zero prior context.
audience: AI assistant (and Luke)
status: proven end-to-end on one build (Spelljammer: Intergalactic, Sept 2026)
reuse: campaign-agnostic and aesthetic-agnostic. See section 8 for what to swap.
---

# Campaign Site Playbook

## 0. How to use this document

You are picking up a procedure that has already been run once, successfully,
start to finish. Everything here is verified against a real build, not theory.

**Read sections 1 through 3 before doing anything.** Section 7 is the most
valuable part of the file. It is a list of traps that each cost between twenty
minutes and two hours to diagnose the first time. Every one of them will happen
again on a new build. Do not rediscover them.

Section 8 tells you exactly what is campaign-specific and must be replaced.
Everything not listed there is reusable as-is.

---

## 1. What is being built

A free, public, read-only website where spectators and players can read session
recaps, browse character profiles, look up setting lore, and see campaign art.

**Source of truth is a shared Obsidian vault.** The players write recaps in it.
The site is downstream of the vault, never the other way around. The vault is
the filing cabinet, the site is the storefront.

**Hard constraints from the user:**

- Free. No paid hosting, no Obsidian Publish.
- The vault must keep working normally in Obsidian for the players.
- Styling is a deliberate retro aesthetic, applied to the site only, never to
  the vault.

---

## 2. The stack

| Layer | Choice | Why |
|---|---|---|
| Static site generator | **Quartz v5** | Free, Obsidian-native, handles wikilinks, backlinks, graph, search |
| Vault to repo | **Quartz Syncer** (Obsidian plugin) | Publishes flagged notes straight to GitHub |
| Hosting | **GitHub Pages** via GitHub Actions | Free, no account beyond GitHub |
| Database views | **Obsidian Bases** + `github:quartz-community/bases-page` | Auto-built index pages from frontmatter |
| Theme | One SCSS file | See section 8 |

**Two locations, two sync mechanisms. This is the single most important
structural fact in this document.**

```
C:\Vaults\<VaultName>\              -> published by Quartz Syncer (Obsidian)
C:\Projects\<repo-name>\            -> published by git (terminal)
```

- Vault notes, images the notes reference, and `.base` files: **Syncer owns them.**
- `quartz.config.yaml`, `quartz/styles/custom.scss`, `quartz/static/*`,
  `content/index.md`: **git owns them.**

When you change both in one turn, the user must do both. Say so explicitly
every time. Assuming one covers the other has caused real breakage.

---

## 3. Build order

Do not reorder these. Later steps depend on earlier ones existing.

1. **Install** Node LTS, Obsidian, Git.
2. **Scaffold Quartz** — `npx quartz create`. Choose the plain/obsidian template,
   not a themed one, so a prebuilt theme does not fight the custom CSS later.
   Note the default branch is **`v5`**, not `main`.
3. **Deploy pipeline** — delete the template's maintainer workflows, write one
   clean `deploy.yml`. See section 6.
4. **Connect Syncer** — point it at the repo, branch, and `content` folder.
5. **Vault preparation** — the biggest phase. Folder architecture, frontmatter,
   internal links, writing any missing content. See section 4.
6. **Auto-built pages** — `.base` files for index pages.
7. **Content features** — image gallery, embedded playlist, etc.
8. **Theme** — the aesthetic layer. See section 8.
9. **Polish and launch** — favicon, social cards, credits, mobile check.

Deliver these to the user **as numbered chunks with explicit steps**, one chunk
per message, each ending with a "Done when:" line stating the observable result.
The user has said repeatedly that this format is what makes the build workable.

---

## 4. Vault architecture

The pattern that worked. Names are campaign-flavoured and should be renamed, the
**shape** is what transfers.

```
<Vault>/
  Episode Notes/
    Act 1 - <name>/
      Arc 1 - <name>/        <- recaps live at the leaf
  <Characters>/              <- was "Nebula Net"
    Crew/                    <- player characters
    Profiles/                <- recurring NPCs
  <Lore>/                    <- was "Giffopedia"
    <Factions>/
    <Locations>/
  Field Manual/              <- house rules, player-facing mechanics
  Gallery/                   <- art originals, NOT published (see 7.10)
  Images/                    <- images referenced by notes
  <Landing>.md
  Progress Report.md
  Soundtrack.md
```

### Recap frontmatter schema

Every recap gets this. The fields drive the auto-built index pages.

```yaml
---
publish: true
type: session
kind: episode        # episode | interlude | special | ova
episode: 27          # only on numbered episodes
act: "Act 1 - <name>"
arc: "Arc 3 - <name>"
sort: 27             # GLOBAL reading order, 1..N, interludes interleaved
---
```

`sort` is the important one. It is the true reading order including side
material, and it is what makes an index page list episodes the way a reader
should actually consume them. Compute it once, deliberately.

### Editing rules for existing recaps

The user wrote these. **They are absolute.**

- **Never change the content.** No rewriting, no tightening, no fixing style.
  Only mechanical corrections explicitly approved.
- **Only link to pages that actually exist.** Never create a link that would
  spawn a new empty page. Strip pre-existing links that point nowhere.
- **One link per target per file**, placed at the earliest mention.
- **No links inside read-aloud text.** Plain `>` blockquotes are read-aloud and
  are off limits. `> [!callout]` blocks are normal content and may be linked.
- Typos get fixed only after listing them and getting approval. Deliberate
  misspellings, puns, in-character comment-section errors and British spellings
  stay.

**Method that made this safe across 43 files:** script every transform
deterministically, then verify with a flatten-and-diff. Strip frontmatter,
flatten all wikilinks to their display text, normalise whitespace, diff old
against new. The only differing lines should be the approved corrections. If
anything else moved, the script is wrong.

---

## 5. Auto-built index pages (Bases)

`.base` files are YAML. Build them by hand rather than in Obsidian's GUI, which
is fiddly and easy to misconfigure.

```yaml
views:
  - type: table
    name: Table
    filters:
      and:
        - file.inFolder("<Folder>")
    order:                    # REQUIRED. Without it the table renders empty.
      - file.name
      - file.folder
    groupBy:
      property: file.folder
      direction: ASC
    sort:
      - property: file.name
        direction: ASC
```

Put each `.base` **inside the folder it indexes**, so it nests in the Explorer
rather than stranding at the root.

---

## 6. Deploy workflow

`.github/workflows/deploy.yml`. Delete every workflow the template ships first,
they deploy Quartz's own docs and are gated to the upstream repo.

```yaml
name: Deploy Quartz site to GitHub Pages
on:
  push:
    branches: [v5]
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with: {fetch-depth: 0}
      - uses: actions/setup-node@v6
        with: {node-version: 24}
      - run: npm ci
      - run: npx quartz plugin install
      - run: npx quartz build
      - uses: actions/upload-pages-artifact@v3
        with: {path: public}
  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/deploy-pages@v4
        id: deployment
```

### The push routine

**Never use `npx quartz sync`.** It commits before it pulls, so any note
published from Obsidian since the last pull looks like a deleted file, and it
will commit that deletion. This destroyed the whole content folder once.

Use plain git, in this order:

```
git pull
git add -A
git commit -m "<message>"
git push
```

---

## 7. Known traps

Each of these was hit for real. Expect all of them again.

**7.1 Syncer strips custom frontmatter.** Turn ON **Include all frontmatter** in
Syncer's Frontmatter settings, or `type`, `act`, `arc` and `sort` never reach the
site and every Base renders empty.

**7.2 Syncer only offers files whose mtime changed.** Flipping a setting changes
no files, so nothing appears as publishable. To force a republish, rewrite the
files with identical bytes to bump their timestamps.

**7.3 Syncer wants to delete `index.md` on every publish.** `content/index.md`
lives only in the project, never in the vault, so Syncer reads it as an orphan.
**Always uncheck the Deleted group before publishing.** It will offer again every
single time.

**7.4 Malformed frontmatter in the vault.** Notes can carry a broken nested block
where `"--- publish: true ---"` is a property *name*, meaning there is no real
`publish` property. It looks fine in Obsidian and silently blocks publishing.
Normalise all frontmatter programmatically before trusting it.

**7.5 Quartz double-prefixes root-relative paths.** Writing
`/<repo-name>/static/x.jpg` produces `/<repo-name>/<repo-name>/static/x.jpg`.
Write `/static/x.jpg` and let Quartz add the base path.

**7.6 Quartz strips `class` from `<input>` elements.** Any CSS targeting an input
by class silently matches nothing. Target structurally instead:
`.wrapper > input[type="checkbox"]`.

**7.7 Quartz's router and popovers hijack internal anchors.** A link to `#foo`
gets a hover preview and its click is intercepted, so `:target` CSS never fires.
For in-page interactive widgets use a hidden checkbox plus `<label for>`, which
Quartz does not touch. Add `data-no-popover="true"` to any anchor that should not
get a preview.

**7.8 Markdown wraps loose raw HTML in `<p>`, which breaks sibling selectors.**
A `<div>` cannot live inside a `<p>`, so the browser hoists it out and separates
it from its neighbour. Wrap each related HTML unit in its own outer `<div>` so
markdown leaves the whole thing alone.

**7.9 Flex children will not shrink below their natural size.** An image inside a
flex column ignores `max-height` until you add `min-height: 0`.

**7.10 A folder and a same-named page collide.** `content/Gallery/` and
`content/Gallery.md` both claim `/gallery`, and the auto-generated folder page
wins. Either rename the page, or add the folder to `ignorePatterns` in
`quartz.config.yaml`. Note that `publish: false` governs notes, not folders. A
folder becomes a page regardless of what is inside it.

**7.11 The theme engine paints over `body`.** It loads after `custom.scss`, so
background rules need `!important` and the wrapper divs need to be made
transparent explicitly.

**7.12 Two writers, one branch.** If Syncer's Bases integration is on, both
Syncer and your local git will write `.base` files, and `git pull` will abort on
untracked-file collisions. Pick one owner per file type and stick to it.

**7.13 Compress art before committing.** A 64-piece gallery arrived as 188 MB
with several 10-15 MB files and some over 100 megapixels. Generate a ~420px
thumbnail and a ~1800px display copy of each, JPEG quality 78-82. That build
came out at 19.6 MB with no visible loss.

**7.14 YouTube embeds can be blocked by the rights holder.** Official music
videos frequently disallow embedding and show "This video is unavailable" while
playing fine on YouTube. This cannot be worked around and should not be. Use
Spotify, which embeds reliably, and keep a plain link as a fallback. Note that
Spotify gives 30-second previews to signed-out listeners.

**7.15 Animated `.ani` cursors do not work in browsers.** Extract frame one from
the RIFF container and write it back as a static `.cur`. Also strip multi-size
cursors down to the 32px entry, because Firefox ignores anything larger.

**7.16 When writing files to the user's machine, use a fresh path per revision.**
Reusing the same staged path can silently re-send the previous version. A
revision that "did not take" with no error is this.

---

## 8. Swapping the campaign and the aesthetic

Everything above is reusable. These are the parts that are not.

### Campaign-specific, must be replaced

- Vault folder names and their contents.
- `act` / `arc` / `sort` values, and the whole Progress Report.
- All page copy: landing page, lore entries, character profiles.
- Repo name, `baseUrl`, and `pageTitle` in `quartz.config.yaml`.
- Artist credits and the gallery manifest.
- Any embedded playlist.

### The aesthetic is contained in five places

To change the entire look, you replace these and nothing else:

1. **`quartz/styles/custom.scss`** — the whole theme. One file.
2. **`quartz/static/fonts/`** — webfonts.
3. **`quartz/static/cursors/`** — custom cursors, optional.
4. **`quartz/static/<wallpaper>`** — page background, optional.
5. **`theme.colors` and `typography` in `quartz.config.yaml`.**

### How to build a theme layer for any aesthetic

The previous build used 98.css for a Windows 98 look. The *method* generalises
to any component library or any hand-built style.

- A component kit styles markup you write yourself. It knows nothing about
  Quartz. You are writing a **connecting layer** that points Quartz's own
  elements at the kit's styles. That layer is unavoidable, and it is wiring,
  not design.
- **Do not paste a whole component library in.** It will set `h1` to 5rem, force
  `table` to never wrap, and restyle `body`, all of which you then have to undo.
  Lift the specific recipes you need and leave the rest.
- **Parameterise the colours yourself.** 98.css hardcodes every value and exposes
  no variables. Copy its recipes into mixins that read from your own custom
  properties, and both light and dark schemes fall out of one variable block.
- **Separate chrome from body text.** Interface fonts are for interface. Long
  recaps need a comfortable reading face at a comfortable size. Pixel and
  display fonts only render crisply at their design size or integer multiples.
- Quartz's dark mode selector is `:root[saved-theme="dark"]`. To force a single
  mode, set the theme plugin's `mode`, disable the darkmode plugin, and hardcode
  the palette, all three.

### Quartz class names worth knowing

Confirmed against rendered output. Verify again if Quartz's version changes.

```
#quartz-root.page > #quartz-body
  .left.sidebar     h2.page-title, .search, .explorer .explorer-content
  .center           .page-header h1.article-title, p.content-meta,
                    article.popover-hint, .page-footer
  .right.sidebar    .graph, .toc, .backlinks
.callout > .callout-title > .callout-title-inner, .callout-content
```

---

## 9. Working agreements with Luke

- **Deliver work in numbered chunks**, one per message, each with a "Done when:"
  line. He has asked for this format repeatedly and it is what keeps the build
  moving.
- **Never alter his writing.** See section 4.
- **Write in his voice, not his transcript.** When asked to produce prose in the
  style of something he wrote, write something new in that register. Do not
  transcribe his sample back at him.
- **His prose rules**, which apply to every word written for the site:
  no em dashes, no colons inside prose sentences, concrete nouns and active
  verbs, and above all **never write in threes**. No three parallel items,
  clauses or fragments. This is his strictest rule and he notices.
- **Register matters.** He asks for professional-email plainness in reference
  material and casual directness in player-facing copy. Ask which if unclear.
- **Tell him exactly which sync path each change needs**, vault or git or both.
- **Diagnose before redesigning.** Three separate bugs in this build were
  misattributed on first guess and cost extra rounds. Fetch the live page, read
  the actual markup, confirm the cause, then fix.
