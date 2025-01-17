# Github Actions Docker Publish Template

The workflows here provide a jumping off point for building and publishing docker images from your project using Github Actions. They provide some opinionated defaults and optimizations to help you easily get started with versioning, tagging, and building images.

**Features**

* Optional, multi-arch (x86/ARM) builds using parallel [native runners](https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners#standard-github-hosted-runners-for-public-repositories) to achieve fast builds
  * This is a substitute for [docks/build-push-action `platform` option](https://github.com/docker/build-push-action?tab=readme-ov-file#customizing) to avoid slow emulation
* Publish to multiple registries
* Templated for [reusable workflow](https://docs.github.com/en/actions/sharing-automations/reusing-workflows) to test app before building
* Image tagging based on source of workflow trigger
  * When code is **pushed** to a branch => tag is `{branch}-{shortSHA}` EX `coolFeat-edad367`
  * When code is **pushed** to default branch (master/main) => tag is `edge-{shortSHA}` EX `edge-edad367`
  * When a **Release** or repo **tag** is published => tag is the repo tag value EX `0.0.1` AND `latest` tag
    * `latest` tag is excluded if repo tag is not semver IE ignores RC/beta/pre-release tags
* Image tag is also passed as the [Build `ARG`](https://docs.docker.com/build/building/variables/#arg-usage-example) `APP_BUILD_VERSION` to your Dockerfile
* Workflow for publishing **Pull Request**
  * Only builds if PR has the label `safe to test` to avoid security issues when using [`pull_request_target`](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#pull_request_target)
  * Images are tagged with `{prNumber}` EX `pr-152` and `APP_BUILD_VERSION` is `{prNumber}-{shortSHA}` EX `pr-152-edad367`
  * Comments on PR with image:tag after image is published and updates when PR is updated