# workflows

Shared, reusable GitHub Actions workflows. No secrets live here.

## slack-notify

Posts a deployment result (success, failure or cancelled) to Slack through an incoming webhook.

### 1. Add the webhook to a project

Create an incoming webhook in Slack, then store it as a secret of the project that deploys:

```powershell
gh secret set SLACK_WEBHOOK_URL --repo nuwan94/<project>
```

`gh` prompts for the value, so the URL is never typed into a file or chat. Without the secret the notify job skips quietly.

### 2. Call it after the deploy job

```yaml
  notify:
    if: always()
    needs: [build, deploy]
    uses: nuwan94/workflows/.github/workflows/slack-notify.yml@v1
    with:
      result: ${{ needs.deploy.result == 'success' && 'success' || (contains(needs.*.result, 'cancelled') && 'cancelled' || 'failure') }}
      environment: GitHub Pages
      url: ${{ needs.deploy.outputs.url }}
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

Inputs: `result` (required), `project` (default: repository name), `environment` (default: production), `url` (optional link).

For a GitHub Pages deploy, expose the page URL from the deploy job:

```yaml
  deploy:
    outputs:
      url: ${{ steps.deployment.outputs.page_url }}
```

### Notes

- Pin to a tag (`@v1`) so a change here never breaks other projects by surprise.
- The message text is built with `jq` from environment variables, so commit messages cannot inject anything into the shell or the JSON.
