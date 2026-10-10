# AGENTS.md

Guidance for coding agents working in this repository.

## What this repo is

Course 1 of the **AI Engineering Fundamentals** educational program (YouTube). It holds
teaching materials and runnable demos: Jupyter notebooks that show a limitation first, then
the technique. There is no application, no build step, and no test suite — the deliverable is
the notebooks and their episode READMEs.

**Repository:** https://github.com/csmedeiros/ai-engineering-foundations
**Linear project:** [YT · Fundamentos de AI Engineering](https://linear.app/caiomedeiros/project/yt-fundamentos-de-ai-engineering-f7123c46f03d)

## Guidelines (do not break these)

- **Vanilla:** direct LLM API calls, no framework (no LangChain, etc.).
- Frameworks and deeper evals belong in an advanced course — do not add them here.
- Each episode shows the limitation **before** the technique.
- Keep changes consistent in style and structure across all three languages (see below).

## Repository layout

Content is split by **language** at the root, then by **episode**. Each episode has a
`README.md` and a `notebooks/` folder.

```
en/   # English
pt/   # Portuguese
es/   # Spanish
└── <NN>-ep-<slug>/
    ├── README.md
    └── notebooks/
        └── <NN>-<topic>.ipynb
```

Episodes (the same set in every language):

| # | Folder | Linear | Topic (EN) |
|---|--------|--------|------------|
| 01 | `01-ep-ai-engineering-map` | [CM-41](https://linear.app/caiomedeiros/issue/CM-41) | The AI Engineering map |
| 02 | `02-ep-llm-tokens-context-limits` | [CM-42](https://linear.app/caiomedeiros/issue/CM-42) | LLMs are not magic: tokens, context, and limits |
| 03 | `03-ep-prompt-vs-system` | [CM-43](https://linear.app/caiomedeiros/issue/CM-43) | Conversation memory & sliding window |
| 04 | `04-ep-tool-calling` | [CM-44](https://linear.app/caiomedeiros/issue/CM-44) | Tool calling |

> Note: `scripts/generate_notebooks.py` still contains an older episode set
> (`04-ep-basic-evaluation`, current-content Ep.01–04 text). The notebooks on disk are the
> source of truth; the generator is stale and currently out of sync with them. Do **not**
> run it as a way to "regenerate" notebooks — it would overwrite hand-edited content.
> Prefer editing the `.ipynb` files directly.

## The language rule (most important convention)

**Every teaching artifact must stay in lockstep across `en/`, `pt/`, and `es/`.** The three
trees mirror each other file for file:

- Same episode folders in each language.
- Same notebook filename in every language (e.g. `03-conversation-memory-sliding-window.ipynb`).
  Notebook files are named `<NN>-<topic>.ipynb`, with `<NN>` matching the episode number:
  `01-ai-engineering-map.ipynb`, `02-tokens-context-limits.ipynb`,
  `03-conversation-memory-sliding-window.ipynb`, `04-tool-calling.ipynb`.
- Same cell structure, cell order, and code — only the prose (markdown text, code comments,
  strings shown to the learner) is translated:

  | | en | pt | es |
  |--|----|----|----|
  | Setup heading | `## Setup — load environment variables` | `## Setup — carregar variáveis de ambiente` | `## Setup — cargar variables de entorno` |
  | Loader comment | `# Reads the .env at the repository root` | `# Lê o .env na raiz do repositório` | `# Lee el .env en la raíz del repositorio` |

When you change one language, make the equivalent change in the other two in the same commit.
Keep code identifiers, API payloads, model IDs, and URLs identical across languages.

## Notebook conventions

- **Format:** nbformat 4. Save with 1-space JSON indentation
  (`json.dumps(nb, indent=1, ensure_ascii=False)`), matching how Jupyter/VS Code persists
  files in this repo. Cells may omit the `id` field; that omission warning is pre-existing
  and acceptable.
- **Setup cell:** Ep. 02–04 notebooks start with a `## Setup` markdown cell followed by a code
  cell that loads environment variables:

  ```python
  from dotenv import load_dotenv

  # Reads the .env at the repository root
  load_dotenv()
  ```

  Keep this as the first thing a learner runs. `python-dotenv` is already in
  `requirements.txt`.
- **No committed secrets.** `.env` is gitignored. Never print or hardcode `NVIDIA_API_KEY`;
  read it with `os.environ`.
- **Outputs:** notebooks may carry execution outputs from authoring runs. Prefer clearing
  noisy/error outputs before committing unless intentionally demonstrating a failure
  (Ep. 03's `context length exceeded` demo is intentionally shown).
- **Localized error text** inside `SystemExit`/print strings is translated too (e.g.
  "Set NVIDIA_API_KEY and run again." / "Defina NVIDIA_API_KEY e rode de novo." / "Define
  NVIDIA_API_KEY y vuelve a ejecutar.").

## Model & API conventions

All live calls target the **NVIDIA NIM** API (OpenAI-compatible format):

- **Base URL:** `https://integrate.api.nvidia.com/v1` (env override: `NVIDIA_BASE_URL`)
- **Auth:** `Authorization: Bearer $NVIDIA_API_KEY`
- **Model:** default `meta/llama-3.1-8b-instruct` (env override: `NVIDIA_MODEL`)
- Prefer the stdlib (`urllib.request` + `json`) for HTTP. No SDKs.
- Free Endpoint models: [build.nvidia.com/models](https://build.nvidia.com/models?filters=nimType%3Anim_type_preview)

Some episodes (e.g. Ep. 02) also include **offline/demo** paths: an offline mock chat, LM
Studio on `http://localhost:1234/v1`, and Docling OCR behind a `RUN_* = False` flag. Keep
those flags off by default.

## Local setup

```bash
uv venv
source .venv/bin/activate        # Windows: .venv\Scripts\Activate.ps1
uv pip install -r requirements.txt
```

Then create `.env` at the repository root:

```bash
NVIDIA_API_KEY="your-key"
```

Run Jupyter from the repository root so `load_dotenv()` finds the root `.env`.

## Workflow for agents

1. **Read first.** Open the target notebook(s) and the matching episode `README.md` before
   editing. Notebooks are hand-maintained and have drifted from the generator.
2. **Change all three languages** when the change is content/structure, not
   language-specific tooling.
3. **Edit notebooks safely.** Round-trip through JSON (`python -c` / a small script) rather
   than hand-editing the raw text, and write back with `indent=1, ensure_ascii=False`. Keep
   the JSON valid — validate with `nbformat.validate(nbformat.read(path, as_version=4))`.
4. **Keep episode READMEs in sync** with notebook content when structure changes (e.g. when
   renaming a notebook, update the filename listed in that episode's `README.md`).
5. **Commit style:** short conventional prefixes — `feat(epNN)`, `docs(epNN)`, `fix(epNN)` —
   e.g. `feat(ep02-04): load .env vars in an initial cell for en/pt/es`.

## Do / Don't

- ✅ Mirror every content change across `en/`, `pt/`, `es/`.
- ✅ Keep demos vanilla and runnable with only `requirements.txt` + `.env`.
- ✅ Show the limitation before the technique.
- ❌ Don't introduce frameworks, SDK clients, or heavy abstractions.
- ❌ Don't commit `.env` or any API key.
- ❌ Don't run `scripts/generate_notebooks.py` to regenerate existing notebooks.
- ❌ Don't translate code semantics — only prose/comments/learner-facing strings.