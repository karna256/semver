# Semver Workflow Documentation

## Overview

This GitHub Actions workflow is designed to automate the process of versioning and releasing a project based on the title of a merged pull request. It follows semantic versioning principles to determine the new version number and creates a new release with the updated version.

## Trigger

The workflow is triggered when a pull request is closed on the `main`, `qa`, or `staging` branches. This ensures that versioning and releases are only handled for changes that are merged into these branches.

## Permissions

The workflow requires the following permissions:
- **deployments: write**: To manage deployments.
- **contents: write**: To push tags and create releases.
- **actions: write**: To perform actions within the workflow.

## Jobs

### 1. Resolve-Environment

This job determines the environment name based on the branch to which the pull request is merged. It sets an environment variable `env_name` that is used in subsequent jobs.

- **Runs on**: `ubuntu-latest`
- **Steps**:
  - **Resolve Environment**: This step checks the branch name and sets the `env_name` variable accordingly:
    - `main` branch sets `env_name` to `prod`.
    - `qa` branch sets `env_name` to `qa`.
    - `staging` branch sets `env_name` to `staging`.

### 2. Semver

This job handles the semantic versioning and creates a new release. It depends on the `Resolve-Environment` job to get the environment name.

- **Needs**: `Resolve-Environment`
- **Environment**: Uses the `env_name` output from the `Resolve-Environment` job.
- **Runs on**: `ubuntu-latest`
- **Environment Variables**:
  - `package_name`: Set to the value of `env_name`.

- **Steps**:
  - **Checkout**: Checks out the repository to the runner.
  - **Setup semver bash**: Installs the `semver` tool if the pull request is merged.
  - **Get version**: Fetches all tags, determines the last version tag, and calculates the new version based on the pull request title:
    - If the PR title contains "major", it bumps the major version.
    - If the PR title contains "feat", it bumps the minor version.
    - Otherwise, it bumps the patch version.
  - **Prepare tag**: Prepares the new tag using the calculated version.
  - **Push Tag**: Pushes the new tag to the repository.
  - **Create release**: Creates a new release with the new tag.

## Note:

The workflow uses the title of the pull request to determine the version bump. This can be easily modified to use branch names to determine version bumps. The current implementation seemed more appropriate for project's workflow since we merge `staging` branch to `main` and this would always bump the minor version. Additional logic is yet to be implemented to keep the versions for `staging` and `main` in sync. This will require a conditional check which will copy the version from staging to main if the version is bumped in staging.