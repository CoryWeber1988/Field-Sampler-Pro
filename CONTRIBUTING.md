# Contributing to Field Sampler

Thanks for your interest in improving this tool. It's a young, single-maintainer project, so please keep pull requests focused and reasonably small — easier to review, easier to merge.

## Reporting bugs

Open an [issue](../../issues) and include, where possible:

- What you were trying to do, and what happened instead.
- The tab and step involved (Plan or Collect, and which tool — boundary, imagery, elevation, zoning, grid, sample points, export, etc.) and the file format, if relevant (e.g. "shapefile .zip sample plan", "KML points", ".fsproj project").
- Browser, OS, and device (phone/tablet/desktop) — GPS and touch behaviour differ a lot between them.
- A sample file that reproduces the problem, if you're able to share one — **please scrub or use non-sensitive test data**, since issues are public. Field boundaries and sample locations can identify a farm, so move or crop them if that matters to you. A few features is usually enough.
- Any error shown in the browser console (F12 → Console tab).

## Suggesting features

Open an issue describing the use case — what you're trying to accomplish, and why the current tool doesn't cover it. Screenshots of the relevant tab, or an example of the file structure you need to support, help a lot.

## Submitting changes

1. Fork the repo and create a branch off `main`.
2. This is a single self-contained `index.html` file — there's no build step, so you can open it directly in a browser to test your changes. GPS features need a secure context, so for those serve it over localhost (`python3 -m http.server 8000`).
3. Before opening a pull request:
   - Check the browser console for errors on a real (or realistic sample) field, covering the Landing → Plan → Send to Collection → Collect → Export flow if your change touches that pipeline.
   - If you changed boundary handling, imagery/elevation loading, zoning, grid, sample point generation, project save/open (`.fsproj`) or export logic, please test with a small realistic field and confirm a save/reopen round-trip and the exports you touched still work.
   - Keep the file dependency-free beyond the libraries already bundled in `index.html` (see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)) unless there's a strong reason to add something new — the point of this tool is that it's one file anyone can open with no install. Any new library must have a licence compatible with AGPLv3 and be added to the notices file.
   - If you change the help text, update both `HELP.md` and the `helpPage` block in `index.html` (between the `HELP BEGIN` / `HELP END` comments) so they stay in sync.
4. Describe what changed and why in the pull request description. For anything that changes numeric or spatial output (zoning, grid, composites, sample point generation, terrain layers, indices), a brief before/after on a test field is very helpful.

## Scope

Field Sampler is deliberately a simple planning and field-collection tool. Heavier analysis (covariate-based site selection, cLHS, Random Forest optimisation and similar) is out of scope. If you're proposing a new sampling method, index, or data source, a link to the agronomic or statistical reasoning behind it — and, for a data source, its licence and whether it allows browser (CORS) access without an API key — is helpful context for review.

## Licensing of contributions

This project is licensed under AGPLv3 (see [LICENSE](LICENSE)). By submitting a pull request, you agree to the terms of the [Contributor License Agreement](CLA.md) (or, if contributing on behalf of a company, the [Entity CLA](CLA-ENTITY.md)) — you keep your copyright, but grant the Project Owner a broad license to use, distribute, and relicense your contribution, including under different license terms in the future if the Project's license ever changes. A CLA Assistant bot checks this automatically: on your first pull request it will ask you to reply with *"I have read the CLA Document and I hereby sign the CLA"* — your signature is then recorded and you won't be asked again.

## Questions

Open an issue, or reach out to Cory Weber at coryweber1988@gmail.com.
