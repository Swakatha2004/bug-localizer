# 🐛 BugLocalizer

> **Pinpoint the exact buggy statement in your codebase — automatically.**  
> A hybrid TF-IDF + Sentence Embedding engine with a sleek evaluation dashboard.

---

## What is this?

BugLocalizer takes a **natural language bug report** and a **list of code statements**, then ranks which statement is most likely the root cause — using a hybrid of classical keyword matching and modern semantic embeddings.

No LLM API costs. No cloud dependency. Runs entirely on your machine.

---

## How it works

```
Bug Report (text)
       │
       ▼
┌─────────────────────────────────────────────────┐
│  Preprocessing                                  │
│  • Lowercasing, punctuation stripping           │
│  • Statement tokenization                       │
└─────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────┐    ┌──────────────────────────┐
│  TF-IDF (40%)    │    │  Sentence Embeddings (60%)│
│  Keyword overlap │    │  all-MiniLM-L6-v2        │
│  cosine sim      │    │  semantic cosine sim      │
└──────────────────┘    └──────────────────────────┘
       │                         │
       └────────── + ────────────┘
                   │
                   ▼
          Final Ranked Score
          ┌──────────────┐
          │  Top-1 / Top-3│
          │  Predictions  │
          └──────────────┘
```

**Scoring formula:**

```
final_score = 0.4 × tfidf_score + 0.6 × embedding_score
```

The embedding model picks up *semantic meaning* even when exact keywords don't match — so a bug report saying *"null pointer crash"* can still match a statement like `object.method()`.

---

## Project structure

```
bugLocalizer/
├── index.html              # Evaluation dashboard (open in browser)
├── bug_localizer.ipynb     # Main notebook (Google Colab)
├── evaluation_results.csv  # Output from the notebook
└── README.md               # You are here
```

---

## Quickstart

### 1. Run the notebook

Open `bug_localizer.ipynb` in [Google Colab](https://colab.research.google.com).

Prepare your CSV with these columns:

| Column | Type | Description |
|---|---|---|
| `bug_report` | string | Natural language description of the bug |
| `statements` | list (JSON) | Python list of code statements as strings |
| `actual_bug_index` | int | Index of the true buggy statement |

Example row:

```csv
bug_report,statements,actual_bug_index
"app crashes when user logs out","['user.logout()', 'db.flush()', 'session = None', 'return redirect()']",2
```

Run all cells. The notebook will output `evaluation_results.csv`.

---

### 2. View the dashboard

Open `index.html` in any browser — no server needed.

1. Upload your `evaluation_results.csv`
2. Click **▶ Run Evaluation**
3. View accuracy metrics, batch charts, and per-test results

The dashboard works offline. Zero setup.

---

## Metrics explained

| Metric | What it means |
|---|---|
| **Top-1 Accuracy** | The model's #1 pick is the actual buggy statement |
| **Top-3 Accuracy** | The actual buggy statement appears in the model's top 3 |
| **Batch Accuracy** | Top-3 accuracy plotted in batches of 20 — reveals consistency |
| **Failures** | Cases where the buggy statement didn't appear in top 3 at all |

Top-3 accuracy is the more practical metric — in real use, a developer inspects the top few candidates, not just the #1 pick.

---

## Model

This project uses [`all-MiniLM-L6-v2`](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2) from the `sentence-transformers` library.

- 384-dimensional embeddings
- Fast inference — ~14,000 sentences/second on CPU
- Trained on over 1 billion sentence pairs
- No GPU required

---

## Dependencies

```
pandas
scikit-learn
matplotlib
sentence-transformers
```

Install everything at once:

```bash
pip install pandas scikit-learn matplotlib sentence-transformers
```

If running locally (outside Colab), also remove the `files.upload()` cell and load your CSV directly:

```python
data = pd.read_csv("your_test_data.csv")
```

---

## Dashboard deployment

The `index.html` dashboard is fully standalone — no backend, no build step.

**Host it free on GitHub Pages:**

1. Create a new public GitHub repository
2. Upload `index.html` to the root
3. Go to **Settings → Pages → Deploy from branch → main / root**
4. Your dashboard is live at `https://YOUR-USERNAME.github.io/REPO-NAME`

---

## Tuning the weights

The 40/60 TF-IDF/embedding split was chosen empirically. You can experiment:

```python
# In the notebook — try different blends
final_scores = 0.3 * tfidf_scores + 0.7 * embedding_scores   # more semantic
final_scores = 0.5 * tfidf_scores + 0.5 * embedding_scores   # balanced
final_scores = 0.0 * tfidf_scores + 1.0 * embedding_scores   # pure embeddings
```

A higher embedding weight generally helps for bug reports that use natural language rather than code terms. Increase the TF-IDF weight if your bug reports contain exact function names or variable names.

---

## Limitations

- Works best when bug reports and code statements share some vocabulary or semantic domain
- Performance drops on very short bug reports (< 5 words)
- Not fine-tuned on code — using a code-specific model like `codebert-base` may improve results
- Statements are treated independently — no control flow or data flow context

---

## License

MIT — free to use, modify, and distribute.

---

<p align="center">Built with sentence-transformers · TF-IDF · Chart.js · PapaParse</p>
