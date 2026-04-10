# GitHub Setup & Automated APK Builds

Complete guide to set up GitHub Actions for automatic Android APK compilation.

## What is GitHub Actions?

GitHub Actions is a free continuous integration/continuous deployment (CI/CD) service that automatically:
- Builds your Android app on every push
- Generates APK files
- Stores them as downloadable artifacts
- Creates releases with APKs

## Prerequisites

- GitHub account (free at https://github.com)
- Android project files (already provided)

## Step-by-Step Setup

### Step 1: Create GitHub Account (if needed)

1. Go to https://github.com
2. Click "Sign up"
3. Follow the registration process
4. Verify your email

### Step 2: Create New Repository

1. Log in to GitHub
2. Click "+" icon (top right)
3. Select "New repository"
4. Fill in details:
   - **Repository name**: `invoice-generator-android`
   - **Description**: "Native Android app for Invoice Generator"
   - **Visibility**: Public (for free Actions) or Private (requires GitHub Pro for private repos with Actions)
   - **Initialize with**: Leave unchecked
5. Click "Create repository"

### Step 3: Upload Project Files

#### Option A: Using Git Command Line (Recommended)

```bash
# Navigate to your project directory
cd invoice-generator-android

# Initialize git repository
git init

# Add all files
git add .

# Create initial commit
git commit -m "Initial commit: Invoice Generator Android app"

# Add remote repository
git remote add origin https://github.com/YOUR_USERNAME/invoice-generator-android.git

# Rename branch to main (if needed)
git branch -M main

# Push to GitHub
git push -u origin main
```

#### Option B: Using GitHub Desktop

1. Download GitHub Desktop from https://desktop.github.com
2. Open GitHub Desktop
3. Click "File" → "Clone repository"
4. Select your new repository
5. Click "Clone"
6. Copy your project files into the cloned folder
7. Commit and push changes

#### Option C: Using Web Interface

1. Go to your repository on GitHub
2. Click "Add file" → "Upload files"
3. Drag and drop your project files
4. Click "Commit changes"

### Step 4: Verify Workflow File

The workflow file (`.github/workflows/build.yml`) should already be included in the project.

To verify:
1. Go to your GitHub repository
2. Click "Actions" tab
3. You should see "Build Android APK" workflow

### Step 5: Trigger First Build

The build automatically triggers when you:
- Push code to `main` or `develop` branch
- Create a pull request
- Manually trigger via "Run workflow"

**Manual trigger:**
1. Go to "Actions" tab
2. Click "Build Android APK" workflow
3. Click "Run workflow"
4. Select branch (main)
5. Click "Run workflow"

### Step 6: Download APK

1. Go to "Actions" tab
2. Click the latest workflow run
3. Scroll down to "Artifacts"
4. Download:
   - `debug-apk` - For testing
   - `release-apk` - For production (if build succeeded)

## Workflow Details

### What the Workflow Does

```
┌─────────────────────────────────────────┐
│  Code pushed to GitHub                  │
└────────────┬────────────────────────────┘
             ↓
┌─────────────────────────────────────────┐
│  GitHub Actions triggered               │
└────────────┬────────────────────────────┘
             ↓
┌─────────────────────────────────────────┐
│  1. Checkout code                       │
│  2. Set up Java 17                      │
│  3. Build debug APK                     │
│  4. Build release APK                   │
│  5. Upload artifacts                    │
└────────────┬────────────────────────────┘
             ↓
┌─────────────────────────────────────────┐
│  APKs available for download            │
└─────────────────────────────────────────┘
```

### Build Times

- **First build**: 5-10 minutes (downloads dependencies)
- **Subsequent builds**: 2-3 minutes (cached dependencies)

### Artifacts Retention

- Artifacts stored for 30 days
- Automatically deleted after 30 days
- Can be extended in workflow settings

## Creating Releases

### Automatic Release on Tag

1. Create a git tag:
```bash
git tag v1.0.0
git push origin v1.0.0
```

2. GitHub automatically:
   - Creates a release
   - Attaches APK files
   - Makes them downloadable

### Manual Release

1. Go to "Releases" tab
2. Click "Create a new release"
3. Fill in:
   - **Tag version**: `v1.0.0`
   - **Release title**: `Release v1.0.0`
   - **Description**: Release notes
4. Upload APK files manually
5. Click "Publish release"

## Troubleshooting

### Build Failed

**Check logs:**
1. Go to "Actions" tab
2. Click failed workflow run
3. Click "Build Android APK" job
4. Scroll through logs to find error

**Common errors:**

| Error | Solution |
|-------|----------|
| `Gradle sync failed` | Check `build.gradle.kts` syntax |
| `Java not found` | Workflow uses Java 17 automatically |
| `Permission denied` | Run `chmod +x gradlew` locally first |
| `Out of memory` | GitHub runners have 7GB RAM (usually enough) |

### APK Not Generated

- Check build logs for compilation errors
- Verify `build.gradle.kts` is correct
- Ensure `AndroidManifest.xml` is valid
- Check for missing dependencies

### Workflow Not Triggering

- Verify `.github/workflows/build.yml` exists
- Check branch name (main or develop)
- Go to "Actions" → "Build Android APK" → "Run workflow"

## Advanced Configuration

### Build Only on Main Branch

Edit `.github/workflows/build.yml`:

```yaml
on:
  push:
    branches: [ main ]  # Only main branch
```

### Add Notifications

Add to workflow to notify on Slack:

```yaml
- name: Slack Notification
  uses: slackapi/slack-github-action@v1
  if: always()
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
```

### Build Multiple Variants

```yaml
- name: Build all variants
  run: ./gradlew assembleDebug assembleRelease
```

### Upload to Firebase App Distribution

```yaml
- name: Upload to Firebase
  uses: wzieba/Firebase-Distribution-Github-Action@v1
  with:
    serviceCredentialsFile: firebase-credentials.json
    groups: testers
    file: app/build/outputs/apk/debug/app-debug.apk
```

## Security Best Practices

### Protect Sensitive Data

1. **Never commit secrets** (API keys, passwords)
2. **Use GitHub Secrets** for sensitive values:
   - Go to Settings → Secrets and variables → Actions
   - Click "New repository secret"
   - Add secret (e.g., `BACKEND_URL`)
   - Reference in workflow: `${{ secrets.BACKEND_URL }}`

### Signing Release APK

To sign release APKs automatically:

1. Create keystore locally:
```bash
keytool -genkey -v -keystore release.keystore \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias release
```

2. Encode to base64:
```bash
base64 release.keystore > keystore.txt
```

3. Add as GitHub Secret:
   - Copy content of `keystore.txt`
   - Settings → Secrets → New secret
   - Name: `RELEASE_KEYSTORE`
   - Value: Paste base64 content

4. Update workflow to use keystore

## Monitoring Builds

### View Build History

1. Go to "Actions" tab
2. See all workflow runs
3. Click run to see details

### Build Status Badge

Add to `README.md`:

```markdown
![Build Status](https://github.com/YOUR_USERNAME/invoice-generator-android/workflows/Build%20Android%20APK/badge.svg)
```

## Sharing APKs

### Share Download Link

1. Go to latest workflow run
2. Download APK
3. Share file with others

### Share via Release

1. Create release (see above)
2. Share release URL
3. Others can download APK directly

### Share via QR Code

1. Upload APK to file hosting (Google Drive, Dropbox, etc.)
2. Generate QR code for download link
3. Share QR code

## Next Steps

1. ✅ Create GitHub account
2. ✅ Create repository
3. ✅ Upload project files
4. ✅ Verify workflow file exists
5. ✅ Trigger first build
6. ✅ Download APK
7. ✅ Test on device
8. ✅ Create releases

## Useful Links

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Android Build Documentation](https://developer.android.com/studio/build)
- [Gradle Documentation](https://gradle.org/docs/)
- [GitHub Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## Support

### Common Questions

**Q: Is GitHub Actions free?**
A: Yes, free for public repositories. Private repos get 2,000 free minutes/month.

**Q: How long does build take?**
A: 2-10 minutes depending on dependencies and changes.

**Q: Can I build on my schedule?**
A: Yes, use `schedule` trigger in workflow.

**Q: Can I build multiple APK variants?**
A: Yes, add build tasks to workflow.

**Q: How do I sign release APKs?**
A: Use GitHub Secrets to store keystore, then configure in workflow.

---

**Last Updated**: April 11, 2026  
**Status**: Ready to Use  
**Free Tier**: Yes (public repos)
