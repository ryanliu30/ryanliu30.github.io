# Ryan Liu's Website

Jekyll site served by GitHub Pages. Template by [Martin Saveski](https://github.com/msaveski/www_personal).

## Updating content
Edit the files in `_data/` unless you are changing the look of the site.

| File | Content |
|------|---------|
| `_data/main_info.yaml` | Name, title, email, photo, social links |
| `_data/publications.yaml` | Papers; optional `paper_pdf`, `slides`, `poster`, `video`, `code`, `data` links |
| `_data/experience.yaml` | Resume timeline (`work` on the left, `school` on the right) |
| `index.html` | Bio and honors text |
| `assets/cv/cv_web.pdf` | CV linked from the Vita section |

## Local preview
```
export PATH="/opt/homebrew/opt/ruby/bin:$(/opt/homebrew/opt/ruby/bin/gem environment gemdir)/bin:$PATH"
jekyll serve
```
Then open http://localhost:4000.
