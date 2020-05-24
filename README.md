# Ok To Test
> Use GitHub Actions secrets in pull requests from forks 🍴🔑

## About

GitHub Actions purposely limits the [secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets) available to pull requests from forks for security reasons:
- [`GITHUB_TOKEN`](https://help.github.com/en/actions/configuring-and-managing-workflows/authenticating-with-the-github_token#permissions-for-the-github_token) is read-only
- [Other secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#using-encrypted-secrets-in-a-workflow) aren't available at all

Though this provides peace of mind, many projects depend on the fork pull request model. If you've configured a GitHub Actions test workflow to trigger on pull requests, and those tests require secrets, the secrets aren't available and the workflow fails.

No longer with this workaround, which shows an example [Prow](https://prow.k8s.io/command-help)-like `/ok-to-test` slash command configuration. 🥳 This project is not affiliated with GitHub.

<p align="center">
    <img src="https://user-images.githubusercontent.com/2993937/82742525-6f9ef900-9d2d-11ea-86a1-e0e6b978faaf.png" width="600" />
</p>

## Example

1. A fork pull request is opened.
2. A [unit test workflow](.github/workflows/unit.yml) runs. Secrets are not available to this workflow.
3. Someone with [write access](https://help.github.com/en/github/getting-started-with-github/access-permissions-on-github) looks over the pull request code. ⚠️ Before proceeding, they should be sure the code isn't doing anything malicious like secret logging. ⚠️
4. They comment `/ok-to-test` on the pull request.
5. A `repository_dispatch` API request is sent to this repository. See guidance [below](#Authentication) on how to authenticate.
6. An [integration test workflow](.github/workflows/integration.yml) runs. Secrets are available to this workflow! 💫
7. The pull request status check is updated to reflect the success or failure of the integration test workflow.

Note that this sequence also works for branch based pull requests, as you'd expect!

## Setup

- Add an authentication [token](#Authentication) as a secret
- Optional: create and install a GitHub App on the repo(s) if you choose that [authentication method](#Authentication)

## Authentication

Choose one of these authentication methods for the `repository_dispatch` helper action, `peter-evans/slash-command-dispatch`, in [`ok-to-test.yml`](.github/workflows/ok-to-test.yml):
- [Personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) with `repo` scope
- [OAuth "app" token](https://developer.github.com/v3/#oauth2-token-sent-in-a-header) with `repo` scope
- ⭐️ Preferred: [GitHub App installation access token](https://developer.github.com/apps/building-github-apps/authenticating-with-github-apps/#authenticating-as-an-installation) with `contents: write` and `metadata: read` permissions

GitHub Apps have distinct identities on GitHub – no seat taken up by a machine account, no potential for leaking your personal credentials, and no rate limit sharing!

## Credits

- [Prow](https://prow.k8s.io/command-help) for the idea for `ok-to-test`
- A few handy community actions, [`peter-evans/slash-command-dispatch`](https://github.com/peter-evans/slash-command-dispatch), [`tibdex/github-app-token`](https://github.com/tibdex/github-app-token), and [`actions/github-script`](https://github.com/actions/github-script)

## Contributing

Pull requests are welcome!

## License

[MIT](LICENSE)
