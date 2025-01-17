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


## Enable Multi-Registry Publishing

The workflows are already configured for pushing to [Docker Hub](https://hub.docker.com/) and [Github Packages (GHCR)](https://docs.github.com/en/packages/learn-github-packages/introduction-to-github-packages). All you need to do is setup some variables/secrets in your repository.

* Go to repository settings -> secrets and variables -> actions
  * Add repository secrets
    * `DOCKER_USERNAME` - your dockerhub username
    * `DOCKER_PASSWORD` - your dockerhub password
  * Add repository **variables**
    * `DOCKERHUB_SLUG` - the full name of the dockerhub image IE `foxxmd/my-project-name`
    * `GHCR_SLUG` - the full name of the GHCR image IE `ghcr.io/foxxmd/my-project-name`
    * If neither of the above variables are included then the job will not run
* Go to repository settings -> Actions -> General
  *  Action permissions -> **Allow all actions...**
  * Workflow permissions -> **Read and write permissions**
  * Save

The workflow will only push to each registry if the corresponding `_SLUG` variable is not empty, so you need to fill in the `_SLUG` variable for the registries you want to push to and can ignore the others.

### Additional/Custom Registry

Each workflow is annotated with comments specifying where it needs to be modified if you want to include additional registries. Look for `add/modify additional registries here` and `add additional registry slugs here` to find those spots. You will need to setup your own repo var/secrets.