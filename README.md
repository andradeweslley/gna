# Gná - Notification Action for GitHub Actions Workflow

GitHub Action that sends a notification to [Hófvarpnir](https://github.com/andradeweslley/notify-system) via `POST /api/notify`. Use it in any workflow to notify your Hófvarpnir instance (e.g. deploy started, success, or failed).

## Usage

### Only one run in the workflow

```yaml
- name: Notify deploy started
  uses: andradeweslley/gna@v1
  with:
    api_key: ${{ secrets.API_KEY }}
    title: "Deploy started"
    message: ${{ github.event.head_commit.message }}
```

### Always run in the workflow (to always notify in workflow completion)

```yaml
- name: Notify deploy result
  if: always()
  uses: andradeweslley/gna@v1
  with:
    api_key: ${{ secrets.API_KEY }}
    title: "Deploy finished"
    message: ${{ github.event.head_commit.message }}
    status: ${{ job.status == 'success' && 'success' || 'failed' }}
```

## Inputs

| Input     | Required | Default | Description |
|----------|----------|---------|-------------|
| `api_key` | Yes     | -       | Bearer token for `/api/notify`. Store in repo secrets (e.g. `API_KEY` or `NOTIFY_API_KEY`). |
| `title`   | Yes     | -       | Notification title. |
| `message` | No      | `""`    | Notification body (sent as `body` in the API). |
| `status`  | No      | `started` | One of: `started`, `success`, `failed`, `info`, `warning`. |
| `priority` | No     | derived from `status` | One of: `low`, `normal`, `urgent`. When not set, it follows the `status`: `started` -> `low`, `success` -> `normal`, `failed` -> `urgent`. With `info` or `warning`, no priority is sent and the API applies its default. Recipients can mute individual statuses/priorities per device in Settings. |
| `warn_only` | No    | `true`  | Report an undelivered notification as a warning instead of failing the step. Set to `false` to fail the step. |

## Delivery failures

Notifications are informational: when a notification does not reach a device, the
step stays green and the reason is reported as a warning annotation. A notification
problem (such as a missing or wrong `api_key`) never fails your deploy:

| Reason | Meaning |
|--------|---------|
| no active push subscription | No device is subscribed — enable push in the dashboard. |
| push service rejected the notification | The push service refused it (often mismatched VAPID keys). |
| invalid API key | The `api_key` is wrong or was regenerated. |
| rate limited | More than 60 notifications in a minute. |

Set `warn_only: false` to fail the step instead, so a broken setup cannot look green.

If every one of the recipient's devices has muted this status or priority, the step
still succeeds (this isn't a delivery failure) but logs a `::notice::` annotation so
the run doesn't look identical to an actually-delivered notification.

## Secrets

- **`API_KEY`** (or `NOTIFY_API_KEY`): Your Hófvarpnir API key.

## License

MIT
