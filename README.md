# The Operator Doctrine

How to get true work out of AI agents, in any field. Second edition, October 2026.

**Read it online:** https://macollins27.github.io/operator-doctrine/

**Or download:**

| Format | Use | File |
|---|---|---|
| **PDF** | Read it on your phone, print it, send it to a friend | [`operator-doctrine.pdf`](operator-doctrine.pdf) |
| **HTML** | Open in any browser | [`operator-doctrine.html`](operator-doctrine.html) |
| **Markdown** | Hand the file to your agent: *"Read this and operate by it"* | [`operator-doctrine.md`](operator-doctrine.md) |

## What this is

A method for putting AI agents to work on things that matter, written for the person responsible for the result, whether or not that person can read what the agent produces. One rule, the eight places an answer can live, the test of every check, the seven ways agents fail and what each became, who checks whom, what reaches a person, stewardship, cost, and what is not proven. Every rule carries the incident that paid for it.

The first edition (June 2026) is kept under `editions/` with a superseded marker.

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — use, share, adapt, even commercially. Attribute Maxwell Collins.

## Source files

- `operator-doctrine.md` — the source-of-truth document.
- `operator-doctrine.html` — rendered HTML (also served at the Pages URL above as `index.html`).
- `operator-doctrine.pdf` — printable PDF generated from the HTML.
- `style.css` — print-ready stylesheet used to generate the HTML/PDF.

To regenerate after editing the markdown:

```bash
pandoc operator-doctrine.md --standalone --toc --toc-depth=2 \
  --css=style.css --syntax-highlighting=pygments \
  --metadata title="The Operator Doctrine" \
  --metadata author="Maxwell Collins" \
  -o operator-doctrine.html
cp operator-doctrine.html index.html
weasyprint operator-doctrine.html operator-doctrine.pdf
```

Tools: `pandoc` and `weasyprint` (both `brew install`-able on macOS).
