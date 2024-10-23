# release-it-gh-workflow

Reusable GitHub Actions workflow for release-it

## Usage

Use the workflow by including it in a [GitHub Actions workflow](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions). For example:

```yaml
name: Push workflow
on: push
jobs:
  release-it-workflow:
    uses: rcwbr/release-it-gh-workflow.github/workflows/release-it-workflow.yaml@0.1.0
```

### Usage with protected branches

With [rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) configured to protect the default branch, the release-it workflow will not have access to create tags and releases. Instead, it must authenticate to act as a [GitHub App](https://docs.github.com/en/apps/overview). The steps to configure an app for this are as follows:

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
1. Copy the secret name; this must be provided as the `app-secret-name` input to the workflow






### Core action

### Conventional changelog

### Custom config

### Dry-run

## Output

### Job summary

### Releases
