# Jira-Gated Jenkins CI/CD Pipeline

This repo demonstrates how to:
- Trigger a Jenkins pipeline only if a Jira issue is **"In Progress"**
- Update the Jira issue status to:
  - ✅ Done on success
  - ❌ Blocked on failure

## Setup
- Add your Jira credentials to Jenkins
- Include Jira ticket (e.g., KAN-4) in commit messages
- Configure GitHub webhook

