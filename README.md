# The Operator Doctrine

How to operate AI to get the best possible output — the synthesized successor to the behaviors catalog and the operator pattern catalog.

**Read it online:** https://macollins27.github.io/operator-doctrine/

**Or download:**

| Format | Use | File |
|---|---|---|
| **PDF** | Read it on your phone, print it, send it to a friend | [`operator-doctrine.pdf`](operator-doctrine.pdf) |
| **HTML** | Open in any browser | [`operator-doctrine.html`](operator-doctrine.html) |
| **Markdown** | Hand the file to your agent: *"Read this and operate by it"* | [`operator-doctrine.md`](operator-doctrine.md) |

## What this is

A single, self-contained doctrine for operating AI agents at a high level — written for two readers at once: an AI agent ingesting it as an operating manual, and a human who wants to learn how to operate AI to get what they actually want.

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
