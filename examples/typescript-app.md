## Hard constraints

- TypeScript 6.x, strict. No `any`, no `as` casts except at the raw-JSON validation boundary, no `@ts-expect-error`.
- ESM only. `"type": "module"`. Node >= 22.
- **Zero comments in source code.** No `//`, no `/* */`, no JSDoc, anywhere under `src/` or `tests/`. All explanation lives in `docs/`. Code must be self-documenting through naming.
- **No emoji anywhere** — not in source, not in CLI output, not in docs, not in commit messages, not in the README. Weather conditions are rendered as text labels only.
- No em dash (`—`) anywhere in the repository. Use common hyphen (`-`) or comma (`,`) instead.
- All user-facing output is in the asked language. All documentation is in English. All code is in English.
- Every type or interface is declared under `src/types/` every time. It does not matter if it crosses a module boundary. Logic modules import them with `import type`.
- When appropriate, GitHub action should reuse workflow. Reusable workflow are named as `*.inc.yml` and are included in a workflow file named `*.flow.yml` (e.g., `tests.inc.yml` is included in `tests.flow.yml`).

---

## Example repository layout

Here is an example of a TypeScript application repository layout :

```
.
├── .github/
│   └── workflows/
│       ├── tests.inc.yml
│       └── tests.flow.yml
├── docs/
│   ├── architecture.md
│   ├── usage.md
│   ├── external-apis.md
│   ├── contributing.md
│   └── release.md
├── src/
│   ├── controllers/
│   │   └── post.ts
│   ├── services/
│   │   └──  post.ts
│   ├── models/
│   │   └── post.ts
│   └── types/
│       └── post.ts
├── tests/
├── eslint.config.js
├── tsconfig.json
├── vitest.config.ts
├── package.json
├── CLAUDE.md
├── README.md
├── LICENSE
└── .gitignore
```

---


## 9. Tooling configuration

### tsconfig.json

TypeScript 6 enables `strict` by default and removed the ES5 target; keep the flags above explicit anyway so the build is not sensitive to compiler defaults.

### ESLint

Flat config (`eslint.config.js`) with `typescript-eslint` type-checked rules. Add a rule or a lint step that fails the build if any comment is found under `src/` — the no-comment constraint must be machine-enforced, not merely stated.

## 10. Git, CI and release

### Branches

- `develop` — default branch, working branch. Lint, typecheck, test, build. **No publication.**
- `main` — production. Merges into `main`.

Initialize both branches. `main` is the repository default.

### Commits

Conventional Commits, enforced by convention and documented in `docs/contributing.md`. `fix:` yields a patch, `feat:` a minor, `BREAKING CHANGE:` in the footer a major.

## Acceptance criteria

- `npm ci && npm run lint && npm run typecheck && npm test && npm run build` passes clean from a fresh clone.
- `grep -rE '//|/\*' src/` returns no comment lines.
- No emoji byte sequence anywhere in the repository.
