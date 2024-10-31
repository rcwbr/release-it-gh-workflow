# release-it-gh-workflow

Reusable GitHub Actions workflow for [release-it](https://github.com/release-it/release-it)

<img width="849" alt="Screen Shot 2024-10-30 at 18 02 35" src="https://github.com/user-attachments/assets/771570ab-3f13-402a-9aee-df956c870db4">

## Usage

Use the workflow by including it in a [GitHub Actions workflow](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions). For example:

```yaml
name: Push workflow
on: push
jobs:
  release-it-workflow:
    uses: rcwbr/release-it-gh-workflow/.github/workflows/release-it-workflow.yaml@0.1.0
```

### Inputs usage

| Input | Required | Default | Type | Effect |
| --- | --- | --- | --- | --- |
| `always-dry-run` | &cross; | `false` | boolean | Whether to include a dry-run of release-it even on the default branch |
| `always-release` | &cross; | `false` | boolean | Whether to include a (not dry-run) run of release-it even on non-default branches |
| `app-id` | &cross; | `''` | string | GitHub App ID to act as, if using |
| `app-environment` | &cross; | `''` | string | Name of the GitHub environment that defines the `app-secret-name`, if stored within an environment |
| `default-branch` | &cross; | `refs/heads/main` | string | The branch from which to release on commit |
| `release-it-config` | &cross; | `/.release-it.json` | string | The path to the release-it config file |
| `release-it-extra-args` | &cross; | `''` | string | Arbitrary extra args to provide to release-it CLI calls |
| `release-it-image` | &cross; | `ghcr.io/rcwbr/release-it-docker-conventional-changelog:0.1.2` | string | The release it action to use |

The job also accepts a secret `app-secret`, the secret key of the App identified by `app-id` (see [usage with protected tags](#usage-with-protected-tags))

### Usage with default job authentication

Many release-it configurations (including those used by default in the workflow, see [usage with local release-it config](#usage-with-local-release-it-config)) push Git tags and potentially [releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases) to the repository. This requires access to push repo contents; while a safer option includes tag protection settings (see [usage with protected tags](#usage-with-protected-tags)), the simplest solution is to provide this access via the [`contents: write` permission](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#jobsjob_idpermissions). For example:

```yaml
jobs:
  release-it-workflow:
    uses: rcwbr/release-it-gh-workflow/.github/workflows/release-it-workflow.yaml@0.1.0
    permissions:
      contents: write
```

### Usage with local release-it config

By default, the workflow uses [release-it configuration](https://github.com/release-it/release-it/blob/main/docs/configuration.md) pre-defined in the release-it Docker image. For a repo with a locally-defined release-it config, this can be overridden by providing a local path to the `release-it-config` input. For example:

```yaml
jobs:
  release-it-workflow:
    uses: rcwbr/release-it-gh-workflow/.github/workflows/release-it-workflow.yaml@0.1.0
    with:
      release-it-config: ./.release-it.json
```

### Usage with protected tags

With [rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) configured to protect tag creation, the release-it workflow will not have access to create tags and releases. Instead, it must authenticate to act as a [GitHub App](https://docs.github.com/en/apps/overview). This may be configured via settings-as-code (see [Protected tags settings-as-code usage](#protected-tags-settings-as-code-usage)). The steps to instead manually configure an app for this are as follows:

1. Navigate to [GitHub Settings > Developer Settings](https://github.com/settings/apps)
1. Create a New GitHub App
1. Under Webhook, disable Active
1. Under Permssions, set Contents to Read and Write
1. After creating the app, under Private keys, Generate a private key
1. Copy the App ID; this must be provided as the `app-id` input to the workflow
1. Select Install App, and Install to your user/organization
1. Select Only select repositories, and select the appropriate repository from the dropdown
1. From the repo Settings page, create a new environment to manage access to the app, via Environments > New environment
1. Copy the environment name; this must be provided as the `app-environment` input to the workflow
1. Enable desired protections; Deployment branches and tags set to the default branch recommended
1. Choose Add environment secret, and paste the App private key .pem file contents generated earlier as the secret value
1. Copy the secret name; this must be provided as the `app-secret` job secret to the workflow

The app must be granted permissions to bypass the tag protections. To configure this, add it as a [bypass actor](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization#editing-a-ruleset) to the applicable tag protection ruleset.

Once the app is configured and inputs provided, it replaces the need for granting `contents: write` permissions to the workflow. An example workflow configuration for this use-case:

```yaml
on: push
jobs:
  release-it-workflow:
    uses: rcwbr/release-it-gh-workflow/.github/workflows/release-it-workflow.yaml@0.1.0
    with:
      app-id: 1033419 # release-it-docker CI release-it app
      app-environment: Repo release
    secrets:
      app-secret: ${{ secrets.RELEASE_IT_GITHUB_APP_KEY }} # Secret belonging to the Repo release environment
```

### Protected tags settings-as-code usage

TODO

Using the [Probot settings GitHub App](https://probot.github.io/apps/settings/), specify the tag protection rules in the `.github/settings.yml` file:

```yaml
rulesets:
  - name: Tags rules
    target: tag
    ...
    bypass_actors:
      - actor_id: 1033419
        actor_type: Integration
        bypass_mode: always
```

## Output

### Job summary

By default, the workflow for a PR defines a [job summary](https://github.blog/news-insights/product-news/supercharging-github-actions-with-job-summaries/) including the dry-run output from a release-it run against the PR contents. This can be found by navigating to the checks for the PR and opening the Summary page.

<img width="849" alt="Screen Shot 2024-10-30 at 18 02 35" src="https://github.com/user-attachments/assets/771570ab-3f13-402a-9aee-df956c870db4">

### Releases

By default, the release-it configuration used by the workflow (see [Usage with local release-it config](#usage-with-local-release-it-config)) generates a [GitHub release](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases) on PR merge to the default branch. These can be found in the repo's Release or Tags pages.

## Contributing

### CI/CD

This repo uses the reusable workflow it defines to release itself. This is specified via the `.github/workflows/.self-push-workflow.yaml` file.
