## Implement Slack Notifications for GitHub Actions (Failures on Main Branch Only)

### Description

As a DevOps engineer, I want to implement Slack notifications in the GitHub Actions pipeline that trigger only on failures for the main branches of both Terraform and Packer pipelines. This will ensure that the team is promptly alerted to critical issues that need immediate attention.

### Acceptance Criteria:

**1.** Create a new slack channel using the naming convention of the existing POD B and just add a suffix of CI/CD, so the notification goes into this channel.

**2.** **Slack Integration:**

- Integrate Slack notifications into the existing GitHub Actions pipeline using a Slack webhook URL.

- Ensure notifications are only sent when the pipeline fails.

**3.** **Branch Condition:**

- Configure the notification to trigger only on failures for the main branch.

**4.** **Secret Management:**

- Store the Slack webhook URL securely using GitHub Secrets.

**5.** **Notification Content:**

- The Slack message should include relevant details such as the repository name, branch name, commit SHA, and a link to the failed GitHub Actions run.

**6.** **Documentation:**

- Update project documentation to include instructions for setting up and modifying Slack notifications in the GitHub Actions pipeline


### Instructions 

1. Create a new slack channel:

- On the slack workspace, Click on the **`+`** button and create a channel.
- Use the naming convention of the existing POD A and just add a suffix of CI/CD.

2. Obtain a Webhook URL:

- Visit the [slack api website](https://api.slack.com/)
- Click on **Your Apps** and click on **create an app**.
- In the dialogue box, select the option that creates the app from scratch.
- Name the app and select the relevant workspace
- On the slack api page, click on incoming webhooks and set it up for the newly created channel.
- Copy the webhook URL

3. Add the Slack Webhook URL to Github Secrets:

- Go to your repository settings on GitHub.
- Navigate to Settings > Secrets and variables > Actions.
- Add a new secret:
x Name: SLACK_WEBHOOK_URL
x Value: Paste the Slack Webhook URL.
