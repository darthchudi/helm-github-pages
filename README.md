# helm-github-pages [![CircleCI](https://circleci.com/gh/int128/helm-github-pages.svg?style=shield)](https://circleci.com/gh/int128/helm-github-pages)

Let's publish your Helm Charts on GitHub Pages using CircleCI.

## Getting Started

### Prerequisite

- You have a GitHub account
- You have a CircleCI account

### 1. Create a repository for publishing your charts

Create a new repository on GitHub.

Open the repository settings and make sure the repository is published as follows:

![github-pages-settings.png](github-pages-settings.png)

It assumes that the repository URL is `https://github.com/YOUR_NAME/helm-charts` in this tutorial.

### 2. Create a repository for the chart

Make a Helm chart as follows:

```sh
git init
mkdir charts
cd charts
helm create example
```

Create `.circleci/config.yml` with the following content:

```yaml
version: 2
jobs:
  build:
    docker:
      - image: alpine
    steps:
      - checkout
      - run:
          name: helm-github-pages
          command: wget -O - https://raw.githubusercontent.com/int128/helm-github-pages/master/publish.sh | sh
          environment:
            # Repository for publishing Helm charts
            - GITHUB_PAGES_REPO: https://github.com/YOUR_NAME/helm-charts
```

Alternatively, you can store [publish.sh](publish.sh) into `.circleci` directory and call it as follows:

```yaml
      - run:
          name: helm-github-pages
          command: .circleci/publish.sh
```

Finally your repository should look like:

- Repository
  - `.circleci/`
    - `config.yml`
  - `charts/`
    - `example/`
      - `Chart.yaml`
      - ...

Then push your changes.

```sh
git remote add origin https://github.com/YOUR_NAME/example
git push origin master
```

### 3. Setup CircleCI

Open CircleCI and start building.

You should configure a checkout key in order to write charts into the `gh-pages` branch of the repository.

1. Open settings of your repository on CircleCI.
1. Open the **Checkout SSH keys** in the Permissions section.
1. Click the **Create and add user key** button.

If the master branch is pushed, syntax checking and publishing are performed.
Otherwise, only syntax checking is performed.

### 4. Verify the publishing

You can add the Helm repository as follows:

```sh
helm repo add YOUR_NAME https://YOUR_NAME.github.io/helm-charts
helm repo update
helm repo list
```

Verify that your chart is available.

```sh
helm inspect YOUR_NAME/examples
```

## Furthermore

### Configurations

You can set the following environment variables:

Name | Value | Default
-----|-------|--------
`GITHUB_PAGES_REPO` | URL of the repository for publishing | Same repository: `$CIRCLE_REPOSITORY_URL`
`GITHUB_PAGES_BRANCH` | Branch name for publishing | `gh-pages`
`HELM_CHARTS_SOURCE` | Helm Charts source directory | `./charts`
`HELM_VERSION` | Helm version | `2.8.1`

### How it works

This script does the following steps:

1. Check out the `gh-pages` branch of the repository for publishing.
1. `helm lint` for each chart.
1. `helm package` for each chart.
1. `helm repo index` for all charts.
1. Push the branch.

It assumes running on the Alpine image and CircleCI docker environment.

## Contribution

This is an open source software licensed under Apache License 2.0.
Feel free to open issues or pull requests.

