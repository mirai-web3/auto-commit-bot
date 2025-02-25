# Auto-Commit Bot 🤖

[![Auto-Commit Workflow](https://github.com/mirai-web3/auto-commit-bot/actions/workflows/auto-commit.yml/badge.svg)](https://github.com/mirai-web3/auto-commit-bot/actions/workflows/auto-commit.yml)

A GitHub Actions bot that automatically makes commits to keep your repository active. Perfect for maintaining streak counts or preventing archived status.

## Features ✨
- Automated commits on schedule
- Random meaningful commit messages
- Smart change detection (only commits when needed)
- Manual trigger capability
- Timezone-aware scheduling (UTC)
- Secure credential handling

## How It Works ⚙️
1. Runs on scheduled days/times (Mon & Wed at 7 AM, 9 AM, 11 AM UTC)
2. Updates a timestamp file (`LAST_UPDATED`)
3. Generates a random meaningful commit message
4. Commits only if changes exist
5. Pushes changes securely using GitHub token

## Setup Guide 🛠️

### 1. Create Workflow File
Create `.github/workflows/auto-commit.yml` with this content:

```yaml
name: auto-commit-bot

on:
  schedule:
    - cron: "0 7,9,11 * * 1,3"
  workflow_dispatch:

jobs:
  auto_commit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Update timestamp
        run: echo "$(date -u +'%Y-%m-%dT%H:%M:%SZ')" > LAST_UPDATED
        
      - name: Setup Git
        run: |
          git config --global user.email "your-email@example.com"
          git config --global user.name "username"
          
      - name: Commit changes
        id: commit
        run: |
          git add -A
          if ! git diff-index --quiet HEAD; then
            git commit -m "$(./generate-commit-message.sh)"
          fi
          
      - name: Push changes
        if: steps.commit.outputs.commit_made == 'true'
        uses: ad-m/github-push-action@v0.8.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### 2. Configure Settings
- Replace `your-email@example.com` with your preferred email
- Adjust the cron schedule if needed

### 3. Commit and Push
Add the workflow file to your repository:
```bash
git add .github/workflows/auto-commit.yml
git commit -m "Add auto-commit workflow"
git push
```

## Customization 🎨
- **Modify Schedule**: Edit the cron expression
- **Change Commit Messages**: Update the messages array in the workflow
- **Adjust Timezone**: Add `TZ: Your/Timezone` to the job

Example Schedule Formats:
```cron
# Every day at 9 AM UTC
"0 9 * * *"

# Weekdays at 12 PM UTC
"0 12 * * 1-5"
```

## FAQ ❓
**Q: Will this affect my commit history?**  
A: Yes, but it creates realistic-looking commits with meaningful messages.

**Q: How to trigger manually?**  
A: Go to Actions → auto-commit-bot → Run workflow

**Q: Is this secure?**  
A: Yes, uses GitHub's built-in token with minimal permissions.

**Q: Why no commits sometimes?**  
A: The bot only commits if file changes are detected.

# Step-by-Step Guide

## 1. Create Workflow File
1. In your repository, create `.github/workflows/autocommit.yml`
2. Copy/paste the workflow configuration from the README
3. Customize:
   - Email address in `git config`
   - Commit messages array
   - Schedule times

## 2. Understand the Schedule
- Original schedule: Mon/Wed at 7,9,11 AM UTC
- Modify using [crontab guru](https://crontab.guru):
  - `0 7,9,11 * * 1,3` = At 07:00, 09:00, and 11:00 on Monday and Wednesday

## 3. Commit the Workflow
```bash
git add .github/workflows/autocommit.yml
git commit -m "chore: Add auto-commit workflow"
git push origin main
```

## 4. Monitor Execution
1. Go to your repo's **Actions** tab
2. Check runs under "auto-commit-bot"
3. Click any run to see detailed logs

## 5. Manual Trigger Test
1. In Actions tab, find "auto-commit-bot"
2. Click "Run workflow" (right sidebar)
3. Select branch and click green button

## 6. Verify Commit
1. Check repository commits
2. Look for messages like:
   - "chore(bot): 🚀 Project update"
   - "chore(bot): 🔒 Security updates"
3. Confirm `LAST_UPDATED` file changes

## 7. Customize (Optional)
- **Change Commit Frequency**: Edit cron schedule
- **Modify Messages**: Update the messages array
- **Add Files**: Make bot update multiple files
- **Add Timezone**:
  ```yaml
  env:
    TZ: America/New_York
  ```

## 8. Troubleshooting
**No Commits Happening?**
- Check workflow runs for errors
- Ensure `LAST_UPDATED` is being modified
- Verify cron schedule syntax

**Permission Errors?**
- Ensure workflow file is in correct location
- Check GitHub Actions permissions in repo settings

**Duplicate Commits?**
- Reduce schedule frequency
- Add more unique messages
