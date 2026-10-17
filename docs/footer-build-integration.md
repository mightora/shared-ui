# Shared footer build integration

The source of truth for all Mightora footer links is [footer.yaml](../../footer.yaml).
Static sites should package a validated copy of that file in each deployment.

## GitHub Actions usage

After building your site, **before** uploading it to GitHub Pages or Azure Storage:

```yaml
- name: Bundle shared Mightora footer
  uses: mightora/shared-ui/.github/actions/sync-footer@main
  with:
    output-directory: dist
```

Change `dist` to the actual published directory: `_site`, `public`, etc.
The directory **must exist** before the action runs. The action downloads
`https://raw.githubusercontent.com/mightora/shared-ui/main/footer.yaml`,
validates sections and absolute links, and writes `<output-directory>/footer.yaml`.
Failures stop the deployment rather than publishing a broken or stale config.

You can specify `ref:` to use a tag/commit instead of main; `@main` here
refers to the action source. To guarantee a fixed action version, pin that too
after creating a release tag or use a commit SHA.

## Runtime behaviour

`<mightora-footer>` now attempts the bundled, local `footer.yaml` first.
If that is missing or invalid it fetches the existing `yaml-url` attribute
or the default jsDelivr source:

https://cdn.jsdelivr.net/gh/mightora/shared-ui@main/footer.yaml

For sites whose pages are served under nested paths, specify an explicit
`local-yaml-url` pointing to the site-root footer (including a GitHub Pages
project base path, if applicable), e.g. `/my-project/footer.yaml`.

A central footer update is used by the next deployment of each app; it does
**not** automatically rebuild other repositories. Existing offline editions
must continue to bundle their own assets and make no external requests.

## Validating the shared action

The `test-sync-footer.yml` workflow smoke-tests bundling and checks JavaScript
syntax. Individual app pipelines continue to run their existing test suites.
