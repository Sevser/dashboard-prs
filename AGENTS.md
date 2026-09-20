# Agent instructions

## Project intent

Build a local-first GitHub pull request analytics dashboard. Apache DevLake will collect and normalize repository data; a custom application will expose PR lead time, review timing, throughput, and aging open PRs.

Read `README.md` for the planned architecture, metric definitions, implementation sequence, and current status. Keep it accurate as the project evolves.

## Working approach

- Deliver small, working increments, starting with local DevLake infrastructure and one repository.
- Inspect existing files before making changes and preserve unrelated user work.
- The application stack is undecided. Do not describe proposed frameworks or commands as already implemented.
- Prefer straightforward solutions and avoid adding services or abstractions without a concrete need.
- Update setup documentation whenever configuration or developer commands change.

## Architecture and data access

- Use Docker Compose for the local DevLake environment and pin the selected release.
- Use DevLake for GitHub ingestion and scheduled synchronization.
- Inspect the schema of the installed DevLake version before writing queries; do not assume table or field names.
- Keep application queries read-only and use a dedicated read-only database account.
- Access the database through the application backend, never directly from the browser.
- Persist collected data with Docker volumes. Do not remove volumes or reset imported data without an explicit request.
- Bind local infrastructure ports to loopback unless broader access is requested.

## Metric correctness

- Follow the definitions in `README.md`; document intentional changes before presenting different calculations under the same label.
- Distinguish PR open-to-merge time from deployment-based lead time for changes.
- Make date-window semantics, timezone, draft handling, and bot exclusions explicit.
- Preserve missing values and distinguish an empty dataset from a failed or incomplete sync.
- Show sample counts with aggregate duration metrics and make the underlying PRs inspectable.
- Use UTC for stored timestamps and calculations; make display timezone choices explicit.
- Label fixture or demo data clearly. Never present it as live GitHub data.

## Credentials

- Never commit tokens, real `.env` files, encryption secrets, or database credentials.
- Add appropriate ignore rules when introducing local configuration; commit only placeholder examples.
- Keep secrets out of logs, browser bundles, screenshots, and test fixtures.
- Configure GitHub authentication locally through DevLake and use the minimum permissions supported by the chosen collection method.

## Validation

- Run checks appropriate to the change and report what was actually verified.
- For Compose changes, validate the resolved configuration without exposing secrets and check service health when Docker is available.
- For metric calculations, test meaningful cases such as unreviewed PRs, unmerged PRs, missing timestamps, and date boundaries.
- Verify important aggregates against a small sample of source PRs when connected data is available.
- For UI changes, cover loading, empty, error, and populated states, and check usability at narrow widths.
- Documentation-only changes do not require application tests.
