# pdum_ts_copier

A [Copier](https://copier.readthedocs.io) template for a **pnpm + TypeScript monorepo** of
standalone npm packages that are **versioned in lockstep**, built with **Vite** (library mode),
tested with **Vitest**, and linted/formatted with **Biome**. Releases run entirely in CI.

## Usage

```sh
# one-time: an alias mirroring pdum_create (Python)
alias pdum_ts_create='uvx --with jinja2_time copier copy --trust --vcs-ref=HEAD https://github.com/habemus-papadum/pdum_ts_copier'

pdum_ts_create pdum_widgets    # generate into ./pdum_widgets
```

`--trust` is required because the template runs post-generation tasks (`pnpm install`, `git init`,
`gh repo create --push`). Answer the prompts and you get a ready-to-develop, ready-to-release
monorepo with one example package.

## What it generates

```
<repo>/
├── package.json            private workspace root (shared scripts + dev tooling)
├── pnpm-workspace.yaml     packages: ["packages/*"]
├── biome.json              lint + format
├── tsconfig.base.json      shared strict TS config
├── vitest.config.ts        aggregates every package's tests
├── packages/<pkg>/         the first package (Vite lib + tsc .d.ts)
├── scripts/
│   ├── versioning.mjs      lockstep version engine (tag-as-truth; CI-managed)
│   ├── new-package.mjs     `pnpm new-package <name>` scaffolder
│   └── _skeleton/          template used by new-package.mjs
└── .github/workflows/
    ├── ci.yml              typecheck + lint + test + build gate
    └── release.yml         manual bump → tag → publish → GitHub release → +dev
```

## Design notes

- **Lockstep versioning** — every package shares one version, enforced in CI by
  `node scripts/versioning.mjs current`. The next version is computed from the highest `vX.Y.Z`
  git tag at release time (tag-as-truth); between releases the tree carries an `X.Y.Z+dev` marker
  that npm rejects, guarding against accidental publishes.
- **CI is the only publish path** — no local release script, no tag trigger. This makes a
  local/CI double-publish impossible.
- Model translated from the CI release pipeline in `pdum_rfb`, stripped of all Python/PyPI.

## After first generation

- Set an npm automation token as the `NPM_TOKEN` Actions secret in the generated repo.
- The npm scope (`@habemus-papadum` by default) and GitHub owner are copier prompts — change them
  at generation time.
