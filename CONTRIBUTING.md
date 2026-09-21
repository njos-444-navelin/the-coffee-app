# Contributing

Thank you for helping build the open source of truth for coffee. Read `MANIFESTO.md` first; every change is measured against it. `docs/ARCHITECTURE.md` holds the stack and the rules that follow from it.

## Licensing

- Code is licensed under the GNU Affero General Public License, version 3 or any later version (`AGPL-3.0-or-later`, see `LICENSE`).
- Documentation in this repository (`docs/`, the Markdown files at the root) is licensed under `CC-BY-SA-4.0`.
- Reference data produced by the app is `ODbL-1.0` and wiki content is `CC-BY-SA-4.0`; see `MANIFESTO.md`. Those licenses will be added to `LICENSES/` when the first such content lands in this repository.

By contributing you agree that your contribution is licensed under the same license as the file it changes ("inbound = outbound"). There is no Contributor License Agreement and there never will be: you keep the copyright to what you write.

Every source file carries a two-line [SPDX](https://spdx.dev/) header, the only license notice we use:

```ts
// SPDX-FileCopyrightText: 2026 The Coffee App contributors
// SPDX-License-Identifier: AGPL-3.0-or-later
```

Use the file type's own comment syntax (`<!-- -->` in HTML and Svelte, `/* */` in CSS). Files that cannot carry a comment (JSON, lockfiles, images) are covered by `REUSE.toml`. CI runs [`reuse lint`](https://reuse.software/) and fails on any file without license information.

## Developer Certificate of Origin

Every commit must be signed off, which certifies the [Developer Certificate of Origin](https://developercertificate.org/): that you wrote the change or otherwise have the right to submit it under the project's license.

```bash
git commit -s
```

This adds a `Signed-off-by: Your Name <you@example.com>` line matching your git identity. CI rejects pull requests with unsigned commits. Forgot one? `git commit --amend -s` fixes the last commit; `git rebase --signoff main` fixes a whole branch.

## Pull requests

Work on a branch, keep the change focused, and explain the why in the description. Product-shaped changes should reference the manifesto principle they serve.
