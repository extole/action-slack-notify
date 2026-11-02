# Slack Notify

Posts a GitHub Actions notification to a Slack incoming webhook. It is a composite action: one bash
script, [`slack-notify`](slack-notify), that needs `jq` and `curl`, and no Docker. Both are on
GitHub-hosted runners and on Extole's self-hosted runners.

The repository is public so that a workflow on a GitHub-hosted runner can call it without a token.
Every Slack notification from an Extole workflow goes through it: directly, as below, or through
tech's `extole-slack-notify` composite action, which calls it.

## Usage

```yml
- name: Send Slack failure alert
  if: ${{ failure() }}
  uses: extole/action-slack-notify@master
  env:
    SLACK_WEBHOOK: ${{ secrets.EXTOLE_ALERTS_SLACK_WEBHOOK_URL }}
    SLACK_CHANNEL: '#production-issues'
    SLACK_COLOR: ${{ job.status }}
    SLACK_TITLE: 'The monitor failed'
    SLACK_MESSAGE: 'Open the run to see which check.'
```

## Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `SLACK_WEBHOOK` | required | The incoming webhook to post to |
| `SLACK_TITLE` | required | The title of the message field |
| `SLACK_MESSAGE` | required | The message, under the title |
| `SLACK_CHANNEL` | the webhook's channel | A channel (`#name`) or a user (`@name`) |
| `SLACK_COLOR` | `good` | `success`, `failure` or `cancelled`, so `${{ job.status }}` works, or any Slack attachment color |
| `SLACK_USERNAME` | the workflow's name | The name the message is posted under |
| `SLACK_ICON` | the Extole avatar | The icon the message is posted under |
| `SLACK_MSG_AUTHOR` | `$GITHUB_ACTOR` | The GitHub login shown as the author. A blank value shows no author |
| `SLACK_FOOTER` | a link to the repository | The footer |
| `SLACK_LINK_NAMES` | `true` | Whether Slack turns `@name` and `#channel` into links |

The message carries the run's event and a link to the run, then the title and the message.

The step fails when Slack refuses the message. Add `continue-on-error: true` to the calling step
when a notification must not fail the job.

## Tests

`./slack-notify-test` checks the payload against a stub `curl`. The `test` workflow runs it, then
calls the action against a local webhook, on every push and pull request.

This repository started as a fork of
[rtCamp/action-slack-notify](https://github.com/rtCamp/action-slack-notify), under its
[MIT license](LICENSE).
