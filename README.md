# Goodreads Reading Habits Story

## Live experience
- **Interactive story**: [Link to experience](https://nchandrasekharr.github.io/booksviz/)
- **Primary entry point**: [`index.html`](index.html) — a single, self-contained scrollytelling page (inline CSS & JS) ready for GitHub Pages.

## Project background
In June I joined a collaborative workshop focused on open data storytelling. Our group explored a public Goodreads dataset sourced from Kaggle and partnered with the OpenAI Codex agent to accelerate both the exploratory analysis and the narrative design. The result is an interactive, New York Times–inspired reading experience that digs into three big questions about how people read and rate books:

1. **Does size matter?** Do page counts correlate with reader ratings?
2. **The power of a punchy title**: Are shorter titles rewarded with better scores?
3. **Hidden gems**: Which little-known books punch above their weight once we account for low visibility?

Throughout the workshop we iterated on copy, layout, accessibility, and animation in close dialogue with Codex. The agent helped wrangle the dataset down to a manageable sample, scripted the preprocessing pipeline, and shaped the D3-based visual interactions while we steered editorial choices.

## What you will find
- A scrollytelling structure with sticky headlines, smooth fade-ins, and narrative steps that frame each chart.
- Three responsive scatter plots rendered with D3 v7, with tooltips for book titles and ratings.
- A dedicated toggle to surface hidden gems and accompanying copy that spotlights five standout titles.
- Inline data compressed with LZString so the page remains fast on GitHub Pages.
- Detailed footer notes that explain the methodology, credit the collaborators, and cite Kaggle as the data source.

## Data & methodology
- **Source**: [Goodreads Books Dataset on Kaggle](https://www.kaggle.com/datasets/zygmunt/goodbooks-10k) (mirrored locally as `GoodReads_100k_books.csv.xz`).
- **Key fields retained**: `rating`, `totalratings`, `pages`, `title`.
- **Cleaning steps**:
  1. Drop records with missing values in the required columns.
  2. Apply 1.5×IQR outlier removal independently on pages, ratings, and log10(total ratings + 1).
  3. Sample up to 10,000 books (uniform random sampling) for fast rendering.
- **Hidden gems definition**: Books with below-median visibility (`log10(totalratings + 1)`) but top-tier ratings, spotlighting the highest-rated five.
- **Compression**: The cleaned sample is stored in `data_compressed.txt` as an LZString-encoded JSON array with two-decimal precision. The page decompresses this client-side to feed the visuals.

To rebuild the compressed payload, install `pandas` and `lzstring`, then run the snippet below (adjust the input path if needed):

```bash
pip install pandas lzstring
python3 - <<'PY'
import json
import numpy as np
import pandas as pd
from lzstring import LZString

raw = pd.read_csv('GoodReads_100k_books.csv.xz', usecols=['rating', 'totalratings', 'pages', 'title'])
clean = raw.dropna().copy()
for col in ['pages', 'rating', 'totalratings']:
    q1 = clean[col].quantile(0.25)
    q3 = clean[col].quantile(0.75)
    iqr = q3 - q1
    clean = clean[(clean[col] >= q1 - 1.5 * iqr) & (clean[col] <= q3 + 1.5 * iqr)]
if len(clean) > 10000:
    clean = clean.sample(10000, random_state=7)
clean['log_totalratings'] = np.log10(clean['totalratings'] + 1)
records = json.loads(clean.drop(columns=['log_totalratings']).to_json(orient='records', double_precision=2))
encoded = LZString().compressToBase64(json.dumps(records, separators=(',', ':')))
with open('data_compressed.txt', 'w') as f:
    f.write(encoded)
with open('processed.json', 'w') as f:
    json.dump(records, f)
print('records', len(records))
PY
```

## Repository structure
```
booksviz/
├── index.html            # Self-contained story with inline styles & scripts
├── data_compressed.txt   # LZString-compressed, two-decimal JSON payload
├── processed.json        # Uncompressed sample (debug/reference only)
├── GoodReads_100k_books.csv.xz  # Original Kaggle dataset (compressed)
└── README.md             # You are here
```

## How to run it yourself
1. Fork or clone this repository.
2. Open `index.html` directly in your browser, or serve the folder with any static server.
3. To publish on GitHub Pages, enable Pages for the repository and point it to the `work` branch (or `main` after merging). The live page will be available at `https://<your-username>.github.io/booksviz/`.
4. If you need to regenerate the dataset, follow the preprocessing steps above and replace `data_compressed.txt` with your new output.

## Workshop notes
- The June workshop was a hands-on lab exploring AI-assisted data storytelling.
- OpenAI Codex handled rapid data transformations and templated D3 components, allowing the human team to focus on editorial framing, annotation, and visual polish.
- Iterating with Codex encouraged tight feedback cycles: we reviewed live previews, adjusted copy, and refined the interaction model until the experience matched our NYT-inspired brief.

## Credits
- **Story & design**: Chandra and Codex, inspired by The New York Times.
- **Data source**: Goodreads dataset via Kaggle.
- **Tools**: D3.js v7, LZString, IntersectionObserver, SVG.

Enjoy exploring the reading habits story, and feel free to remix it for your own datasets.
