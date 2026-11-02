# Concordance Explorer

A single self-contained HTML page for comparing two DESeq2 contrasts. Drop in
two result tables and it reports the overlap, the directional concordance
against a transcriptome-wide null, and draws the scatter, with export to SVG,
PNG, CSV and a plain-text statistics file.

Everything runs in the browser. The gene tables are read with the File API and
never uploaded anywhere, so the page is safe to host publicly even while the
data behind it is unpublished. The only outbound request the page makes is the
Google Fonts stylesheet; delete that `<link>` and it falls back to system fonts
with no network traffic at all.

## Running it

Open `index.html` in a browser. That is the whole installation — no build step,
no dependencies, no server.

## Publishing it on GitHub Pages

```bash
git init
git add index.html README.md
git commit -m "Concordance Explorer"
git branch -M main
git remote add origin git@github.com:<user>/concordance-explorer.git
git push -u origin main
```

Then in the repository: **Settings → Pages → Source: Deploy from a branch**,
branch `main`, folder `/ (root)`. The page appears at
`https://<user>.github.io/concordance-explorer/` within a minute or two.

A public repository gets Pages on any GitHub plan. A private repository needs
GitHub Pro, Team or Enterprise; the published site is public either way, so
keep unpublished data out of the repository itself.

To update the page later, edit `index.html`, commit and push. Pages redeploys
on every push to the branch.

## Input format

Any CSV or TSV with one row per gene. The page looks for these columns by name,
case-insensitively, and lets you pick a different one from a dropdown if the
guess is wrong:

| needed  | recognised names                                                 |
|---------|------------------------------------------------------------------|
| gene    | `gene`, `gene_id`, `symbol`, `id`, `name`, or the first column    |
| log2FC  | `log2FoldChange`, `log2FC`, `logFC`, `lfc`                        |
| padj    | `padj`, `FDR`, `adj.P.Val`, `qvalue`                              |

`baseMean`, `stat` and `pvalue` are used when present and ignored when absent.
Unquoted and quoted fields both parse; the delimiter is detected from the
header.

## What it computes

- The universe is every gene with a non-missing `padj` **and** `log2FoldChange`
  in both tables, so the overlap is not inflated by genes only one analysis
  tested.
- Overlap significance is the one-sided hypergeometric test.
- Concordance is tested against the sign-agreement rate across that whole
  universe, not against 50%. Two contrasts sharing a reference sample or a
  direction of effect already agree more often than chance, and the
  genome-wide rate absorbs that.
- Threshold-free sign agreement is reported for each contrast's DEGs
  separately, with a binomial test against the same null.
- Pearson and Spearman correlations across four subsets: all tested genes,
  each contrast's DEGs, and the shared set.

## Notes

- A `git` repository inside a Dropbox folder can be corrupted by sync while
  Dropbox is writing to `.git`. Clone or move this directory somewhere outside
  Dropbox before running `git init`.
- The page opens on simulated example data, labelled as such in the plot
  header. It clears as soon as a real file is loaded.
