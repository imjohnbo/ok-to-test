# Ok To Test

> _Example workflow configuration_ showing how to use GitHub Actions secrets in pull requests from forks 🍴🔑

## Summary

An [`Ok To Test`](https://github.com/imjohnbo/ok-to-test/blob/master/.github/workflows/ok-to-test.yml) workflow is configured so that when someone with write access to this repository comments `ok-to-test sha=<head-sha>` on a pull request from a fork, a "privileged" [`Integration tests`](https://github.com/imjohnbo/ok-to-test/blob/master/.github/workflows/integration.yml) workflow needing [secrets](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets#about-encrypted-secrets) is triggered. In parallel, a "non-privileged" [`Unit tests`](https://github.com/imjohnbo/ok-to-test/blob/master/.github/workflows/unit.yml) workflow not needing secrets is triggered on any pull request.

## About

GitHub Actions purposely limits the [secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets) available to pull requests from forks for security reasons:

- [`GITHUB_TOKEN`](https://help.github.com/en/actions/configuring-and-managing-workflows/authenticating-with-the-github_token#permissions-for-the-github_token) is read-only
- [Other secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#using-encrypted-secrets-in-a-workflow) aren't available at all

Though this provides peace of mind, many projects depend on the fork pull request model. If you've configured a GitHub Actions test workflow to trigger on pull requests, and those tests require secrets, the secrets aren't available and the workflow fails.

No longer with this workaround, which shows an example [Prow](https://prow.k8s.io/command-help)-like `/ok-to-test sha=<head-sha>` slash command configuration! 🥳

This project is not affiliated with GitHub.

<p align="center">
    <img src="https://user-images.githubusercontent.com/2993937/101568108-0b2d4980-39a0-11eb-9e87-d838ae934097.png" width="600" />
</p>

## Setup

This is a [template repository](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/creating-a-repository-from-a-template#about-repository-templates) with three example workflows. Start by [creating a new repository](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/creating-a-repository-from-a-template#creating-a-repository-from-a-template) ("Use this template"). Then, consider for your use case:

1. [Which type of token](#authentication) you'll use to emit the `repository_dispatch` event in [`Ok To Test`](https://github.com/imjohnbo/ok-to-test/blob/master/.github/workflows/ok-to-test.yml). Set the secrets in your repository accordingly, e.g. [I used a GitHub App](https://github.com/imjohnbo/ok-to-test/blob/master/.github/workflows/ok-to-test.yml#L20-L21) and had to save secrets called `APP_ID` and `PRIVATE_KEY`. Remember: if you also choose GitHub App authentication (preferred), you must create _and install_ it on the repo(s) in which this configuration will run. See [Creating A GitHub App](#creating-a-github-app) for a basic overview of how to do this.
1. Which workflow(s) need [secrets](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets#about-encrypted-secrets). In this example, it's [`Integration tests`](https://github.com/imjohnbo/ok-to-test/blob/master/.github/workflows/integration.yml), and I would need to fill in my tests [here](https://github.com/imjohnbo/ok-to-test/blob/master/.github/workflows/integration.yml#L36).
1. Which workflow(s) do not need [secrets](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets#about-encrypted-secrets). In this example, it's [`Unit tests`](https://github.com/imjohnbo/ok-to-test/blob/master/.github/workflows/unit.yml). These types of workflows can simply trigger on pull request.
1. The Permissions required for your `GITHUB_TOKEN`. The workflows used to implement `ok-to-test` require the ability to: add reactions to your pull request comments, and update the status of your pull request checks. Currently GitHub Actions' built-in `GITHUB_TOKEN` is [`read`-only by default](https://github.blog/changelog/2023-02-02-github-actions-updating-the-default-github_token-permissions-to-read-only/). The example workflows in this repo explicitly grant the necessary `write` permissions to the jobs that require them. You can read more about this in the [GitHub Docs](https://docs.github.com/en/actions/security-guides/automatic-token-authentication), which also describe how to update the defaults.

## Usage

As someone with write access, comment `/ok-to-test sha=<head-sha>` on an incoming pull request to set off this [Rube Goldberg machine](https://en.wikipedia.org/wiki/Rube_Goldberg_machine) 😄. The head `sha` is the first seven characters of the most recent commit of the incoming pull request. [For example](https://github.com/imjohnbo/ok-to-test/pull/5#issuecomment-635368312), `/ok-to-test sha=742c71a`.

## Example

1. A fork pull request is opened.
2. A [unit test workflow](.github/workflows/unit.yml) runs. Secrets are not available to this workflow.
3. Someone with [write access](https://help.github.com/en/github/getting-started-with-github/access-permissions-on-github) looks over the pull request code. ⚠️ Before proceeding, they should be sure the code isn't doing anything malicious like secret logging. ⚠️
4. They comment `/ok-to-test sha=<head-sha>` on the pull request.
5. A `repository_dispatch` API request is sent to this repository. See guidance [below](#authentication) on how to authenticate.
6. An [integration test workflow](.github/workflows/integration.yml) runs, checking out the merge commit if the head sha hasn't changed since the comment was made. Secrets are available to this workflow! 💫
7. The pull request status check is updated to reflect the success or failure of the integration test workflow.

Note that this sequence also works for branch based pull requests, as you'd expect!

## Authentication

Choose one of these authentication methods for the `repository_dispatch` helper action, `peter-evans/slash-command-dispatch`, in [`ok-to-test.yml`](.github/workflows/ok-to-test.yml):

- [Personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) with `repo` scope
- [OAuth "app" token](https://developer.github.com/v3/#oauth2-token-sent-in-a-header) with `repo` scope
- ⭐️ Preferred: [GitHub App installation access token](https://developer.github.com/apps/building-github-apps/authenticating-with-github-apps/#authenticating-as-an-installation) with `contents: write` and `metadata: read` permissions. See [Creating A GitHub App](#creating-a-github-app) for a basic overview of how to do this.

GitHub Apps have distinct identities on GitHub – no seat taken up by a machine account, no potential for leaking your personal credentials, and no rate limit sharing!

### Creating a GitHub App

Here we are using a GitHub App as an authentication entity. No code will be added to the GitHub App, instead we will be using its permissions when we execute our GitHub Workflows. rather than a traditional coding application. Below are some brief instructions on how to setup a GitHub App for this purpose, note that there are other methods of creating a GitHub App such as with a manifest file (e.g. one similar to [`app.yml`](/app.yml)). _(The instructions below are for setting up an app within your user, but you can also do it for your organization.)_

1. Go to `Settings > Developer Settings > GitHub Apps`, and select `New GitHub App`.
1. Enter a name for your app (this needs to be unique across GitHub), and fill in the required URL fields. You can fill in these URLs with fake values - they do not need to resolve, so you can use:
    - Homepage URL = `http://example.com`
1. You can ignore the 'Callback URL' field, and untick `'Webook' > Active`
1. Under `Repository Permissions`, set:
    - Contents = 'Read and Write'
    - Metadata = 'Read-only'
1. Click 'Create GitHub app'
1. Click 'Generate Private key' (this will be downloaded to your computer), and take a note of the `App ID` field
1. Install the GitHub App into your user or organization, by clicking 'Install' under the 'Install App' tab, and choose whether you want to give the app access to all of your user's / org's repositories, or just specific ones
1. Go to the repository that you want to use `ok-to-test` with, and then `Settings > Secrets and variables > Actions` and create two new secrets:
    - `APP_ID` with the value for the `App ID` field that you noted earlier
    - `PRIVATE_KEY`, copying and pasting in the full contents of the Private Key file that you generated and downloaded earlier

## Credits

- [Prow](https://prow.k8s.io/command-help) for the idea for `ok-to-test`
- A few handy community actions, [`peter-evans/slash-command-dispatch`](https://github.com/peter-evans/slash-command-dispatch), [`tibdex/github-app-token`](https://github.com/tibdex/github-app-token), and [`actions/github-script`](https://github.com/actions/github-script)

## Contributing

Pull requests are welcome!

## License

[MIT](LICENSE)
