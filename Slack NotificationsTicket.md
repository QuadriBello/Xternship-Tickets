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

**1. Create a new slack channel:**

- On the slack workspace, Click on the **`+`** button and create a channel.
- Use the naming convention of the existing POD A and just add a suffix of CI/CD.

**2. Obtain a Webhook URL:**

- Visit the [slack api website](https://api.slack.com/)
- Click on **Your Apps** and click on **create an app**.
- In the dialogue box, select the option that creates the app from scratch.
- Name the app and select the relevant workspace
- On the slack api page, click on incoming webhooks and set it up for the newly created channel.
- Copy the webhook URL

**3. Add the Slack Webhook URL to Github Secrets:**

- Go to your repository settings on GitHub.
- Navigate to Settings > Secrets and variables > Actions.
- Click on **Add a new secret**
- Enter the Secret Name: **`SLACK_WEBHOOK_URL`** and paste the Slack Webhook URL.

**4. Update the Packer Pipeline:**

- Add steps in the pipeline to notify the team via Slack when a failure occurs on the main branch.
- The **`Notify Slack on Failure`** step is added at the end of both the **`packer-test`** and **`deploy-to-production`** jobs. It checks if the pipeline has failed and triggers a notification if the failure occurs on the main branch.
- The Slack Webhook URL is securely stored in GitHub Secrets and is passed into the environment variables.

**5. Usecase for Step:**

```
#  This step checks if the pipeline has failed and triggers a notification if the failure occurs on the main branch.  
      - name: Notify Slack on Failure
        if: always() && github.ref == 'refs/heads/main'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          REPO_NAME: ${{ github.repository }}
          BRANCH_NAME: ${{ github.ref_name }}
          COMMIT_SHA: ${{ github.sha }}
          ACTION_URL: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
        run: |
          if [ "${{ job.status }}" == 'failure' ]; then
            curl -X POST -H 'Content-type: application/json' \
              --data "{\"text\":\"❌ Packer Pipeline failed in repository $REPO_NAME on branch $BRANCH_NAME (commit: $COMMIT_SHA). View details: $ACTION_URL\"}" \
              $SLACK_WEBHOOK_URL
          fi
```

**i.** **`if: always() && github.ref == 'refs/heads/main'`**:
   - `always()`: Ensures that this step runs regardless of the success or failure of the previous steps. This is important as the pipeline may stop executing once a failure occurs. 
   - `github.ref == 'refs/heads/main'`: Restricts the execution of the Slack notification to the `main` branch only.

**ii.** **Environment Variables**:
   - `SLACK_WEBHOOK_URL`, `REPO_NAME`, `BRANCH_NAME`, `COMMIT_SHA`, and `ACTION_URL` are set using GitHub context expressions. These provide the necessary information to format the Slack message.

**iii.** **`if [ "${{ job.status }}" == 'failure' ]; then ... fi`**:
   - This shell command checks if the job's status is `"failure"`.
   - If the job has failed, it sends a POST request to Slack using `curl` with the formatted message.
