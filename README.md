# Helm Charts on GitHub Pages (`gh-pages`)

GitHub Action that publishes Helm charts to a Helm repository hosted on GitHub
Pages.

## Usage - Basic

This GitHub Action will package your Helm charts and deploy them to GitHub
Pages for you! Here's a basic workflow example:

```yml
name: Helm Publish

on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: dellintosh/helm-gh-pages@master
        with:
          chart_path: charts/my-app
          gh_pages_URL: https://octocat.github.io/charts
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

* `chart_path` (_required_): Path to the application chart you wish to publish.
* `gh_pages_URL` (_required_): URL pointing to the output `gh-pages` site.
* `tag_filter` (_optional_): Git tag filter
* `GITHUB_TOKEN` (_required_): Environment Variable pulled from secrets. This is used to publish Helm chart assets to the `gh-pages` branch.

In order to use this action your Git repository should have a `gh-pages` branch
created.

## Filter by git tag

Publish the Helm chart located at `charts/my-app` when the Git tag contains the `chart-` prefix:

```yaml
...
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: dellintosh/helm-gh-pages@master
        with:
          chart_path: charts/my-app
          gh_pages_URL: https://octocat.github.io/charts
          tag_filter: chart-
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## How it Works

Assuming your GitHub repository has a Helm chart named `app` located at
`chart/app` the release procedure could be:

```bash
# make changes to the chart/app and bump the version inside Chart.yaml
$ git commit -m "Bump chart version to 1.0.0"
$ git push origin master

# release v1.0.0
$ git tag v1.0.0
$ git push origin v1.0.0
```

When you push the tag, GitHub will start the workflow and the helm-gh-pages
will do the following:

-   check out the `v1.0.0` tag
-   validate the chart by running Helm lint
-   package the chart to `/github/home/pkg/app-1.0.0.tgz`
-   check out the `gh-pages` branch
-   copy the `app-1.0.0.tgz` from `/github/home` to `/github/workspace`
-   update the Helm repository index using the GitHub pages URL
-   commit the chart package and the Helm repository index
-   push the changes to `gh-pages` using the `GITHUB_TOKEN` secret

In couple of seconds GitHub will publish the change to GitHub Pages and your
chart v1.0.0 will be available for download.

## Credits

This GitHub Action is a modified version of [stefanprodan](https://github.com/stefanprodan/gh-actions) original `gh-actions/helm-gh-pages` Action. Sadly, his
version was not updated when GitHub released Actions v2, so I modified it to
work with the v2 release.
