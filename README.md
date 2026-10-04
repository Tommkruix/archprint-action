# archprint check

A GitHub Action that flags the architecture-rule violations a pull request **introduces**, with the evidence for
each rule, using [archprint](https://github.com/Tommkruix/archprint). It never reports the existing backlog, only
what the change adds, and it counts what the change fixed.

Each finding shows inline on the pull request, for example:

> **archprint: AP-001** A request handler must not import the database client directly. When this rule was
> adopted, 40 of 40 files it applies to followed it (confidence 91%).

## Set up

1. Adopt the rules your code already follows, once, and commit the result:

   ```bash
   npx archprint init
   git add .archprint && git commit -m "Adopt archprint rules"
   ```

2. Add the workflow:

   ```yaml
   # .github/workflows/archprint.yml
   name: archprint
   on: pull_request
   permissions:
     contents: read
   jobs:
     check:
       runs-on: ubuntu-latest
       steps:
         - uses: Tommkruix/archprint-action@v1
   ```

The action checks out the pull request itself (the head commit, with full history), so do not add a checkout step
before it.

## Block merges on new violations

Set `fail-on: new`, then mark the job a required status check in your branch protection:

```yaml
- uses: Tommkruix/archprint-action@v1
  with:
    fail-on: new
```

## Inputs

| Input | Default | What it does |
| --- | --- | --- |
| `fail-on` | `none` | `none` only warns. `new` fails the job when the pull request adds a violation. |
| `path` | the app in `.archprint/config.json` | App directory to check, for a monorepo. |
| `working-directory` | `.` | Directory that holds `.archprint/`. |
| `base` | the pull request's base commit | Branch or commit to compare against. |
| `archprint-version` | `0.9.0` | archprint version to run, pinned for reproducible results. |
| `node-version` | `22` | Node.js version. |

## What it checks, and what it does not

- Only the rules your team adopted with `archprint init` or `generate`, from `.archprint/rules.json`. Only the
  mechanical ones; rules that depend on guessing a folder's role are never enforced.
- Rules adopted or changed in the same pull request are listed in the summary but never counted against it. A rule
  removed in the pull request is flagged.
- Set up with archprint 0.8.x or earlier? Run `npx archprint generate` once to write `rules.json`; until then the
  check posts a notice that it did not run.

## Security

The action needs only `contents: read`, uses no secrets, and works on pull requests from forks. It runs on
`pull_request`, never `pull_request_target`. archprint reads your source; it never runs your code, and it does not
install your dependencies.

## License

[MIT](LICENSE)
