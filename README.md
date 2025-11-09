# release-it-gh-workflow

[![GitHub Release](https://img.shields.io/github/v/release/rcwbr/release-it-gh-workflow?logo=semver&style=flat-square)](https://github.com/rcwbr/release-it-gh-workflow/releases/latest)
[![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/rcwbr/release-it-gh-workflow/.self-push-workflow.yaml?logo=github&style=flat-square)](https://github.com/rcwbr/release-it-gh-workflow/actions/workflows/.self-push-workflow.yaml?query=branch%3Amain)

Reusable GitHub Actions workflow for [release-it](https://github.com/release-it/release-it)

<img width="849" alt="Screen Shot 2024-10-30 at 18 02 35" src="https://github.com/user-attachments/assets/771570ab-3f13-402a-9aee-df956c870db4">

## Usage

Use the workflow by including it in a [GitHub Actions workflow](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions). For example:

```yaml
name: Push workflow
on: push
jobs:
  release-it-workflow:
    uses: rcwbr/release-it-gh-workflow/.github/workflows/release-it-workflow.yaml@0.2.4
```

### Inputs usage

| Input | Required | Default | Type | Effect |
| --- | --- | --- | --- | --- |
| `always-dry-run` | &cross; | `false` | boolean | Whether to include a dry-run of release-it even on the default branch |
| `always-release` | &cross; | `false` | boolean | Whether to include a (not dry-run) run of release-it even on non-default branches |
| `app-id` | &cross; | `''` | string | GitHub App ID to act as, if using |
| `app-environment` | &cross; | `''` | string | Name of the GitHub environment that defines the `app-secret-name`, if stored within an environment |
| `artifact-name` | &cross; | `''` | string | Name of a GitHub Actions artifact to download (generally for assets) |
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
    uses: rcwbr/release-it-gh-workflow/.github/workflows/release-it-workflow.yaml@0.2.4
    permissions:
      contents: write
```

### Usage with local release-it config

By default, the workflow uses [release-it configuration](https://github.com/release-it/release-it/blob/main/docs/configuration.md) pre-defined in the release-it Docker image. For a repo with a locally-defined release-it config, this can be overridden by providing a local path to the `release-it-config` input. For example:

```yaml
jobs:
  release-it-workflow:
    uses: rcwbr/release-it-gh-workflow/.github/workflows/release-it-workflow.yaml@0.2.4
    with:
      release-it-config: ./.release-it.json
```

### Merge strategies settings-as-code usage

If using conventional-changelog (default), or other release-it plugins that require consistent git history, it is necessary to follow a [merge-commit merge strategy](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github). It is recommended to apply settings to enforce this using [Probot settings as code](https://github.com/repository-settings/app):

```yaml
repository:
  ...
  # Prevent strategies other than basic merge, as they interfere with conventional changelog version inference
  allow_squash_merge: false
  allow_rebase_merge: false
  # Instead, merge by merge commit
  allow_merge_commit: true
  # Clean up branches when PRs merge
  delete_branch_on_merge: true
```

### Usage with protected tags

With [rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) configured to protect tag creation, the release-it workflow will not have access to create tags and releases. Instead, it must authenticate to act as a [GitHub App](https://docs.github.com/en/apps/overview). The steps to instead manually configure an app for this are as follows:

Follow [this link](https://github.com/settings/apps/new?name=YOUR_REPO_NAME%20CI%20release-it&url=https://github.com/YOUR_REPO_URL&description=This%20app%20enables%20the%20YOUR_REPO_NAME%20GitHub%20Actions%20workflow%20to%20authenticate%20as%20an%20app%20to%20publish%20tags%20and%20releases&webhook_active=false&contents=write) to create an App:

```
https://github.com/settings/apps/new?name=YOUR_REPO_NAME%20CI%20release-it&url=https://github.com/YOUR_REPO_URL&description=This%20app%20enables%20the%20YOUR_REPO_NAME%20GitHub%20Actions%20workflow%20to%20authenticate%20as%20an%20app%20to%20publish%20tags%20and%20releases&webhook_active=false&contents=write
```

<details>
<summary><i>Alternatively, follow these manual steps to achieve the same</i></summary>

1. Navigate to [GitHub Settings > Developer Settings](https://github.com/settings/apps)
1. Create a New GitHub App
1. Under Webhook, disable Active
1. Under Permssions, set Contents to Read and Write

</details>

<br>

To configure workflow access to the App:

1. After creating the App, under Private keys, Generate a private key
1. Copy the App ID; this must be provided as the `app-id` input to the workflow
1. Select Install App, and Install to your user/organization
1. Select Only select repositories, and select the appropriate repository from the dropdown
1. If using repo settings-as-code, apply the settings for the environment (see [Protected tags settings-as-code usage](#protected-tags-settings-as-code-usage)). 
    <details>
    <summary><i>Otherwise, follow these steps</i></summary>

      1. From the repo Settings page, create a new environment to manage access to the app, via Environments > New environment
      1. Copy the environment name; this must be provided as the `app-environment` input to the workflow
      1. Enable desired protections; Deployment branches and tags set to the default branch recommended
    </details>
1. From the environment configuration page, choose Add environment secret, and paste the App private key .pem file contents generated earlier as the secret value
1. Copy the secret name; this must be provided as the `app-secret` job secret to the workflow

The app must be granted permissions to bypass the tag protections. To configure this, add it as a [bypass actor](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization#editing-a-ruleset) to the applicable tag protection ruleset (see [Protected tags settings-as-code usage](#protected-tags-settings-as-code-usage)).

Once the app is configured and inputs provided, it replaces the need for granting `contents: write` permissions to the workflow. An example workflow configuration for this use-case:

```yaml
on: push
jobs:
  release-it-workflow:
    uses: rcwbr/release-it-gh-workflow/.github/workflows/release-it-workflow.yaml@0.2.4
    with:
      app-id: 1049549 # release-it-gh-workflow release-it app
      app-environment: Repo release
    secrets:
      app-secret: ${{ secrets.RELEASE_IT_GITHUB_APP_KEY }} # Secret belonging to the Repo release environment
```

### Protected tags settings-as-code usage

It is recommended to configure repos for this workflow using [Probot settings as code](https://github.com/repository-settings/app). To configure tag protection rules, include the following in the `.github/settings.yml` file:

```yaml
rulesets:
  ...
  - name: Tags rules
    target: tag
    enforcement: active
    conditions:
      ref_name:
        include:
          - "~ALL"
        exclude: []
    rules:
      - type: creation
      - type: deletion
      - type: non_fast_forward
      - type: update
```

The above settings prevent any edit of tags, including creation. To grant release-it automation access to create tags, add it to the tags rule as a bypass actor, using the ID of the GitHub App created for this automation (see [Usage with protected tags](#usage-with-protected-tags)):

```yaml
rulesets:
  - name: Tags rules
    target: tag
    ...
    bypass_actors:
      - actor_id: 1049549 # release-it-gh-workflow release-it app
        actor_type: Integration
        bypass_mode: always
```

Lastly, provision an environment for managing access to the GitHub App secret. To limit that access to events on the `main` branch, use a custom branch policy:

```yaml
environments:
  - name: Repo release # Must match the app-environment workflow input
    deployment_branch_policy:
      custom_branches:
        - main
```

> :warning: Note that an environment configured as code as above will not be provisioned until pushed to the repo default branch. An initial merge will fail to release as the App secret must be manually applied to the environment after merging. However, the release workflow may be retried once all configuration is in place, and should then succeed.

### Protected branches settings-as-code usage

If combining protected branches with the [file bumper](https://github.com/rcwbr/release-it-docker#file-bumper-image-usage) release model, the settings must configure the release-it GitHub App (as created under [Usage with protected tags](#usage-with-protected-tags)) to bypass the default branch protection. Apply this under the default branch protection settings:

```yaml
rulesets:
  ...
  - name: Default branch rules
    target: branch
    conditions:
      ref_name:
        include:
          - "~DEFAULT_BRANCH"
        exclude: []
    ... # Add the following block, with the appropriate actor ID:
    bypass_actors:
      - actor_id: 1049549 # release-it-gh-workflow release-it app
        actor_type: Integration
        bypass_mode: always
```

### Required checks settings-as-code usage

To require successful workflow release-it dry-runs on PRs, branch protections and required checks may be configured via [Probot settings as code](https://github.com/repository-settings/app).

Apply basic branch protection settings similar to these:

```yaml
rulesets:
  ...
  - name: Default branch rules
    target: branch
    enforcement: active
    conditions:
      ref_name:
        include:
          - "~DEFAULT_BRANCH"
        exclude: []
    rules:
      - type: creation
      - type: deletion
      - type: non_fast_forward
      - type: pull_request
        parameters:
          dismiss_stale_reviews_on_push: true
          require_code_owner_review: true
          require_last_push_approval: false
          # Use codeowners vs. review count, so codeowners can merge without review
          required_approving_review_count: 0
          required_review_thread_resolution: true
```

And add a required status check for the release-it dry-run job:

```yaml
rulesets:
  ...
  - name: Default branch rules
      ...
      # Require workflow job as check to enable PR up-to-date rule
      - type: required_status_checks
        parameters:
          required_status_checks:
            - context: release-it workflow / Release-it dry-run
              integration_id: 15368 # GitHub Actions integration ID
          # Requires PR branches to be up-to-date with target
          strict_required_status_checks_policy: true
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
