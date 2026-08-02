# AGENTS.md

## Cursor Cloud specific instructions

### What this repository is

This is a **documentation-only repository**. It contains the architecture, domain, and
design documentation (Markdown + PNG/draw.io diagrams under `docs/`) for a *future*
"Plataforma Central de Seguridad". There is **no application source code, build system,
package manager, automated test suite, or CI here**. `docs/07-engineering/ci-cd.md` and
`docs/07-engineering/repository-structure.md` are explicit stubs that describe the *future*
implementation repo, not this one.

Consequences:
- There are **no project dependencies to install**. The startup update script is a no-op.
- There is nothing to "compile". The "product" is the rendered documentation itself.
- The docs are authored as GitHub-flavored Markdown. Many section index links are written
  as directory links (e.g. `diagrams/`, `c4/`, `docs/01-governance/adr/`). GitHub resolves
  these to the folder's `README.md`. A strict Markdown site generator will only log these as
  informational "unrecognized relative link" notices — they are an existing authoring
  convention, **not** broken links. Do not "fix" them by editing the docs.

### Lint / test for the docs

The closest equivalent to a test is a relative link + image integrity check (every
non-directory relative link and every image reference must resolve on disk; directory links
are valid when the directory or its `README.md` exists). There is no committed tooling for
this; run an ad-hoc check when validating doc edits.

### Running / previewing the docs (the "application")

Preview tooling (MkDocs) is **not** part of the repo and is intentionally **not** installed
by the update script. Install it once per fresh VM into an isolated venv, then serve the
`docs/` tree. Because the repo has no `mkdocs.yml`, `index.md`, or `docs_dir` layout, point
MkDocs at a small staging source dir that symlinks the repo `README.md` as the home page:

```bash
# one-time install (isolated; does not touch the repo)
sudo apt-get install -y python3.12-venv          # if venv is missing
python3 -m venv /tmp/docsvenv
/tmp/docsvenv/bin/pip install mkdocs mkdocs-material

# staging source so the root README.md becomes the site home page
mkdir -p /tmp/site_src
ln -sf /workspace/docs      /tmp/site_src/docs
ln -sf /workspace/README.md /tmp/site_src/index.md

# minimal config (docs_dir points at the staging dir)
cat > /tmp/mkdocs.yml <<'YML'
site_name: Plataforma Central de Seguridad - Arquitectura
docs_dir: /tmp/site_src
site_dir: /tmp/docs_site
theme:
  name: material
  language: es
YML

/tmp/docsvenv/bin/mkdocs serve -f /tmp/mkdocs.yml -a 127.0.0.1:8000
```

Then open `http://127.0.0.1:8000/`. MkDocs live-reloads on edits to files under `docs/` and
`README.md` (the symlinks are followed), so the edit -> preview loop works for authoring.

Note: MkDocs is only a convenience previewer. It is not required by the repo, is not
committed, and its "unrecognized relative link" notices are expected (see above).
