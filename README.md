# Academic website (Jekyll)

A minimalist academic / portfolio site inspired by [philipzrh.com](https://philipzrh.com).
Content is Markdown-driven — **add a project by dropping a `.md` file in `_projects/`.**

## Add a new project

1. Create a file `_projects/2025-my-paper.md`.
2. Add front matter and a body:

   ```markdown
   ---
   title: "My Paper Title"
   category: research        # "research" = Research section; anything else = Other Projects
   year: 2025
   authors: "Me, Coauthor"
   venue: "NeurIPS"
   thumbnail: /assets/img/projects/my-paper.png
   summary: "One line shown on the homepage card."
   links:
     - name: Paper
       url: https://arxiv.org/abs/xxxx
     - name: Code
       url: https://github.com/you/repo
   ---

   Full write-up in Markdown here (shows on the project's own page).
   ```

3. Drop the thumbnail in `assets/img/projects/`.

That's it — it appears automatically, newest year first.

## Edit the rest

| What            | Where                     |
|-----------------|---------------------------|
| Name, bio, links| `_config.yml` (`author:`) |
| Profile photo   | `assets/img/profile.jpg`  |
| News            | `_data/news.yml`          |
| Education       | `_data/education.yml`     |
| Experience      | `_data/experience.yml`    |
| Styling         | `assets/css/style.css`    |

## Run locally

```bash
bundle install
bundle exec jekyll serve
# open http://127.0.0.1:4000
```

## Deploy to GitHub Pages

1. Push to a repo named `<username>.github.io` (user site) or any repo (project site).
2. In **Settings → Pages**, set **Source = GitHub Actions**.
3. If it's a *project* site (not `<username>.github.io`), set `baseurl: "/<repo-name>"`
   and `url:` in `_config.yml`.

The included workflow (`.github/workflows/pages.yml`) builds and deploys on every push to `main`.
