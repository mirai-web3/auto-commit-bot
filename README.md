# Auto Commit Bot 🤖

Automatically commit changes to your repository through GitHub Actions with customizable schedules.

## Features ✨
- Automated commits on schedule (Mon & Wed at 7 AM, 9 AM, 11 AM UTC)
- Random meaningful commit messages
- Web-only setup (no local configuration needed)
- Safe credential handling
- Manual trigger capability

## Setup Guide 🛠️

### Step 1: Create Workflow File
1. In your repository, go to the **Actions** tab
2. Click **New workflow**
3. Choose **set up a workflow yourself**
4. Name the file `.github/workflows/auto-commit.yml`
5. Paste this content:

```yaml
name: auto-commit-bot

on:
  schedule:
    - cron: "0 7,9,11 * * 1,3" # 7 AM, 9 AM, 11 AM UTC on Mon/Wed
  workflow_dispatch: # Allow manual triggering

jobs:
  auto_commit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Update timestamp
        run: |
          echo "$(date -u +'%Y-%m-%dT%H:%M:%SZ')" > LAST_UPDATED

      - name: Setup Git Config
        run: |
          git config --global user.email "your@email.com"
          git config --global user.name "yourusername"

      - name: Generate commit message
        id: commit-msg
        shell: bash
        run: |
          messages=(
            "chore(bot): 🚀 Project update"
            "chore(bot): 🌟 Code improvements"
            "chore(bot): 📦 Dependency sync"
            "chore(bot): 🎨 Format tweaks"
            "chore(bot): 🐛 Fix minor issues"
            "chore(bot): 🔄 Routine sync"
            "chore(bot): ⚡ Performance tweaks"
            "chore(bot): ✨ Content update"
            "chore(bot): 🔒 Security updates"
            "chore(bot): 🌐 Localization"
          )
          idx=$(( RANDOM % ${#messages[@]} ))
          echo "message=${messages[idx]}" >> $GITHUB_OUTPUT

      - name: Commit changes
        id: commit
        shell: bash
        run: |
          git add -A
          if ! git diff-index --quiet HEAD; then
            git commit -m "${{ steps.commit-msg.outputs.message }}"
            echo "commit_made=true" >> $GITHUB_OUTPUT
          fi

      - name: Push changes
        if: steps.commit.outputs.commit_made == 'true'
        uses: ad-m/github-push-action@v0.8.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          tags: false
          force: false
```

6. Click **Start commit** and commit directly to `main`/`master` branch

### Step 2: Update Workflow Permissions
1. Go to **Settings** → **Actions** → **General**
2. Under **Workflow permissions**:
   - Select **Read and write permissions**
   - Check **Allow GitHub Actions to create and approve pull requests**
3. Click **Save**

### Step 3: Manual Trigger (Optional)
1. Go to **Actions** tab
2. Select **auto-commit-bot** workflow
3. Click **Run workflow**
4. Select branch and click green **Run workflow** button

## FAQ ❓

**Q: Why aren't commits showing up?**<br>
A: 1) Check workflow runs for errors 2) Ensure cron schedule matches UTC time 3) Verify workflow permissions

**Q: How to change commit schedule?**<br>
A: Edit the cron pattern in the workflow file using [crontab.guru](https://crontab.guru)

**Q: Is this safe for public repos?**<br>
A: Yes! The GITHUB_TOKEN is automatically secured by GitHub.

**Q: Can I customize commit messages?**<br>
A: Yes! Edit the `git commit -m` line in the workflow file.

## Notes 📌
- Commits will only occur when file changes exist
- All times are in UTC
- Emojis are randomized using WhatTheCommit API
- No personal credentials required
