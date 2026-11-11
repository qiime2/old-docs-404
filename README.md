# old-docs-404

The not-found site for https://docs.qiime2.org, where the old (Sphinx) [QIIME 2](https://qiime2.org) documentation was served until November 2026.
The current documentation is at https://amplicon-docs.qiime2.org.

## What it does

The site holds one page, [`site/404.html`](site/404.html), which Cloudflare Pages serves with status 404 at every address except those that [`site/_redirects`](site/_redirects) sends elsewhere:

| Address | Response |
| --- | --- |
| `/` | 302 to https://amplicon-docs.qiime2.org/ |
| `/en/...` | 302 to the same path on amplicon-docs.qiime2.org |
| The two data files the Moving Pictures tutorial downloads (`/2024.10/data/tutorials/feature-classifier/85_otus.qza` and `ref-taxonomy.qza`) | 302 to their copies in the Zenodo record |
| `/2024.10/...` | 404 page with the link to that page's archive.org copy |
| `/<older version>/...` | 404 page with archive.org's copies of that page, the same page in 2024.10 on archive.org, and the Zenodo archive, in that order |
| `/<version>/data/...` | 404 page pointing to the Zenodo archive only |
| anything else | 404 page saying "Page not found" |

A version is anything matching `^/(2\.0\.\d+|20\d\d\.\d+)(/|$)`.
The page can't tell whether an old address was ever a real page, so its text for those addresses is conditional ("if this page was part of ...").
The script that picks the section, and the reasons for each choice, are in comments at the top and bottom of the page.

The redirects are 302s rather than 301s because browsers keep a 301 indefinitely, and the current documentation may move to docs.qiime2.org later.

## Test links

Each of these should show the response in the table above.

| Case | Link |
| --- | --- |
| Root | https://qiime2-old-docs-404.pages.dev/ |
| Current-docs address | https://qiime2-old-docs-404.pages.dev/en/stable/ |
| Page in 2024.10, the last release | https://qiime2-old-docs-404.pages.dev/2024.10/plugins/available/diversity/core-metrics/ |
| Page in an older release | https://qiime2-old-docs-404.pages.dev/2019.1/tutorials/moving-pictures/ |
| Oldest release | https://qiime2-old-docs-404.pages.dev/2.0.6/ |
| Tutorial data file | https://qiime2-old-docs-404.pages.dev/2024.10/data/tutorials/moving-pictures/table.qza |
| Moving Pictures data file (works once the Zenodo record is published) | https://qiime2-old-docs-404.pages.dev/2024.10/data/tutorials/feature-classifier/85_otus.qza |
| Anything else | https://qiime2-old-docs-404.pages.dev/no-such-page/ |

To check status codes and redirect targets instead:

```
for u in \
  https://qiime2-old-docs-404.pages.dev/ \
  https://qiime2-old-docs-404.pages.dev/en/stable/ \
  https://qiime2-old-docs-404.pages.dev/2024.10/data/tutorials/feature-classifier/85_otus.qza \
  https://qiime2-old-docs-404.pages.dev/2024.10/data/tutorials/feature-classifier/ref-taxonomy.qza \
  https://qiime2-old-docs-404.pages.dev/2024.10/plugins/available/diversity/core-metrics/ \
  https://qiime2-old-docs-404.pages.dev/2017.2/ \
  https://qiime2-old-docs-404.pages.dev/2024.10/data/tutorials/moving-pictures/table.qza \
  https://qiime2-old-docs-404.pages.dev/no-such-page/ ; do
  printf '%s\n  ' "$u"; curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' "$u"
done
```

Expected: the first four answer 302 (two to amplicon-docs.qiime2.org, two to zenodo.org), and the rest 404.
For a pull request, replace the host with its preview address; after the cutover, with docs.qiime2.org.

## Deployment

The Cloudflare Pages project `qiime2-old-docs-404` (https://qiime2-old-docs-404.pages.dev), in the Cloudflare account that holds the qiime2.org zone, is connected to this repository, with no build command and `site` as the build output directory.
Each push to `main` deploys within a minute or two, and each pull request gets its own preview address.
From the cutover on, `docs.qiime2.org` is a custom domain of the project.

## Changing it

- `ARCHIVE_DATE` in the page's script is the day by which every 2024.10 page had a fresh archive.org capture.
  Links ask archive.org for the capture nearest that date, so the old documentation must go offline at least a week after it; otherwise archive.org's captures of this 404 page could be nearer.
- The Zenodo DOI, 10.5281/zenodo.24082639, appears twice in the page, and the record id (24082639) in the two data-file redirects.
  Both work only once the record is published; check that the record id matches then.

## History

The page was first written for amplicon-docs ([qiime2/amplicon-docs#36](https://github.com/qiime2/amplicon-docs/pull/36)), to be served by Read the Docs once the current documentation moved to docs.qiime2.org.
In November 2026 that move was deferred and the page moved here, so that the old documentation can be retired independently of it.
