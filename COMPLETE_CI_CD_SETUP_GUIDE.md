# Complete CI/CD Setup Guide for Expo React Native Projects

A comprehensive guide to set up a complete CI/CD pipeline for Expo React Native projects using GitHub Actions and EAS (Expo Application Services).

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Project Setup](#project-setup)
3. [EAS Configuration](#eas-configuration)
4. [GitHub Actions Setup](#github-actions-setup)
5. [Environment Variables](#environment-variables)
6. [Workflow Files](#workflow-files)
7. [Testing and Validation](#testing-and-validation)
8. [Troubleshooting](#troubleshooting)
9. [Migration to Other Projects](#migration-to-other-projects)

## 🎯 Prerequisites

### Required Tools
- **Node.js** (v20+)
- **pnpm** or **npm** package manager
- **Git** version control
- **GitHub** account with repository
- **Expo** account (sign up at [expo.dev](https://expo.dev))

### Install Required CLI Tools
```bash
# Install Expo CLI globally
npm install -g @expo/cli

# Install EAS CLI globally
npm install -g eas-cli
```

### Required Package.json Scripts
Ensure your `package.json` includes these scripts:
```json
{
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web",
    "lint": "expo lint"
  }
}
```

## 🚀 Project Setup

### Step 1: Login to Expo
```bash
# Login to your Expo account
eas login
```

### Step 2: Initialize EAS Project
```bash
# Initialize EAS project (creates project ID)
eas project:init
```

### Step 3: Configure EAS Update
```bash
# Configure EAS Update for OTA updates
eas update:configure
```

### Step 4: Configure EAS Build
```bash
# Configure EAS Build
eas build:configure
```

### Step 5: Configure EAS Submit (Optional)
```bash
# Configure EAS Submit for app store submissions
eas submit:configure
```

## ⚙️ EAS Configuration

### Create `eas.json` File
Create an `eas.json` file in your project root:

```json
{
  "cli": {
    "version": ">= 12.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "channel": "development"
    },
    "development-simulator": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "simulator": true
      },
      "channel": "development-simulator"
    },
    "preview": {
      "distribution": "internal",
      "channel": "preview"
    },
    "production": {
      "autoIncrement": true,
      "channel": "production"
    }
  },
  "submit": {
    "production": {}
  }
}
```

### Update `app.json` Configuration
Ensure your `app.json` includes these configurations:

```json
{
  "expo": {
    "name": "your-app-name",
    "slug": "your-app-slug",
    "version": "1.0.0",
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    },
    "owner": "your-expo-username",
    "runtimeVersion": {
      "policy": "appVersion"
    },
    "updates": {
      "url": "https://u.expo.dev/your-project-id"
    }
  }
}
```

## 🔐 Environment Variables

### Step 1: Generate Expo Token
1. Go to [expo.dev/accounts/[username]/settings/access-tokens](https://expo.dev/accounts/[username]/settings/access-tokens)
2. Click "Create Token"
3. Name it "GitHub Actions CI/CD"
4. Copy the generated token

### Step 2: Create GitHub Environment
1. Go to your GitHub repository
2. Navigate to **Settings → Environments**
3. Click "New environment"
4. Name: `CI CD`
5. Click "Configure environment"
6. Add environment secrets:
   - **Name**: `EXPO_TOKEN`
   - **Value**: Your Expo access token

### Step 3: Optional Store Submission Secrets
If you plan to submit to app stores, add these secrets:

**For Google Play Store:**
```
GOOGLE_SERVICE_ACCOUNT_KEY = base64 encoded service account JSON
```

**For Apple App Store:**
```
APPLE_API_KEY_ID = Your API key ID
APPLE_API_ISSUER_ID = Your issuer ID
APPLE_API_KEY = base64 encoded .p8 file content
```

## 🔄 Workflow Files

### Create `.github/workflows/` Directory
```bash
mkdir -p .github/workflows
```

### Main CI/CD Pipeline: `.github/workflows/ci.yml`

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

permissions:
  contents: read
  pull-requests: write
  issues: write

jobs:
  code-quality-check:
    name: "Code Quality Check"
    runs-on: ubuntu-latest
    outputs:
      branch-type: ${{ steps.branch-info.outputs.branch-type }}
      build-version: ${{ steps.version.outputs.build-version }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Determine branch type and version
        id: branch-info
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "branch-type=production" >> $GITHUB_OUTPUT
          elif [[ "${{ github.ref }}" == "refs/heads/develop" ]]; then
            echo "branch-type=staging" >> $GITHUB_OUTPUT
          elif [[ "${{ github.event_name }}" == "pull_request" ]]; then
            echo "branch-type=preview" >> $GITHUB_OUTPUT
          else
            echo "branch-type=feature" >> $GITHUB_OUTPUT
          fi

      - name: Validate app.json
        run: |
          echo "Validating app.json file..."
          if [ ! -f app.json ]; then
            echo "❌ app.json not found in current directory!"
            echo "Current directory: $(pwd)"
            echo "Files in current directory:"
            ls -la
            exit 1
          fi
          
          echo "✅ app.json found, validating JSON syntax..."
          if ! cat app.json | jq empty; then
            echo "❌ app.json contains invalid JSON!"
            echo "Content preview:"
            head -20 app.json
            exit 1
          fi
          echo "✅ app.json is valid JSON"

      - name: Generate build version
        id: version
        run: |
          # Get current version from app.json with error handling
          CURRENT_VERSION=$(cat app.json | jq -er '.expo.version // "1.0.0"' || echo "1.0.0")
          
          if [[ "$CURRENT_VERSION" == "1.0.0" ]]; then
            echo "⚠️  Using fallback version 1.0.0 (expo.version not found in app.json)"
          else
            echo "✅ Found version in app.json: $CURRENT_VERSION"
          fi
          
          # Generate build version based on branch type
          if [[ "${{ steps.branch-info.outputs.branch-type }}" == "production" ]]; then
            # Production: use semantic versioning
            BUILD_VERSION="$CURRENT_VERSION"
          else
            # Non-production: add commit hash and timestamp
            TIMESTAMP=$(date +%Y%m%d-%H%M%S)
            SHORT_SHA=${GITHUB_SHA:0:7}
            BUILD_VERSION="$CURRENT_VERSION-${{ steps.branch-info.outputs.branch-type }}-$TIMESTAMP-$SHORT_SHA"
          fi
          
          echo "build-version=$BUILD_VERSION" >> $GITHUB_OUTPUT
          echo "Generated version: $BUILD_VERSION"

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --no-frozen-lockfile

      - name: Run ESLint
        run: pnpm run lint

      - name: TypeScript check
        run: npx tsc --noEmit

      - name: Check Expo configuration
        run: npx expo-doctor

  production-build:
    name: "Production Build"
    runs-on: ubuntu-latest
    needs: code-quality-check
    if: github.ref == 'refs/heads/main'
    environment: CI CD
    outputs:
      build-url: ${{ steps.build-info.outputs.build-url }}
      download-url: ${{ steps.build-info.outputs.download-url }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --no-frozen-lockfile

      - name: Setup EAS CLI
        uses: expo/expo-github-action@v8
        with:
          expo-version: latest
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - name: Check for existing production build
        id: check-build
        run: |
          echo "Checking for existing production builds..."
          BUILD_LIST_OUTPUT=$(eas build:list --platform android --limit 1 --json --non-interactive 2>/dev/null || echo "[]")
          
          if [[ "$BUILD_LIST_OUTPUT" == "[]" ]] || [[ -z "$BUILD_LIST_OUTPUT" ]]; then
            echo "No builds found or command failed"
            BUILD_COUNT=0
          else
            BUILD_COUNT=$(echo "$BUILD_LIST_OUTPUT" | jq length 2>/dev/null || echo 0)
          fi
          
          echo "Found $BUILD_COUNT existing builds"
          if [ "$BUILD_COUNT" -gt 0 ]; then
            echo "has-builds=true" >> $GITHUB_OUTPUT
          else
            echo "has-builds=false" >> $GITHUB_OUTPUT
          fi

      - name: Create production build
        if: steps.check-build.outputs.has-builds == 'false'
        id: build-info
        run: |
          echo "Creating production build v${{ needs.code-quality-check.outputs.build-version }}"
          
          # Create build with version
          BUILD_OUTPUT=$(eas build --platform android --profile production --non-interactive --wait --json)
          
          if [[ -z "$BUILD_OUTPUT" ]] || [[ "$BUILD_OUTPUT" == "null" ]]; then
            echo "❌ Build failed or returned empty output"
            exit 1
          fi
          
          # Parse build information with error handling
          BUILD_ID=$(echo "$BUILD_OUTPUT" | jq -r '.[0].id // "unknown"' 2>/dev/null || echo "unknown")
          BUILD_URL=$(echo "$BUILD_OUTPUT" | jq -r '.[0].logsUrl // "unknown"' 2>/dev/null || echo "unknown")
          DOWNLOAD_URL=$(echo "$BUILD_OUTPUT" | jq -r '.[0].artifacts.buildUrl // "pending"' 2>/dev/null || echo "pending")
          
          echo "build-id=$BUILD_ID" >> $GITHUB_OUTPUT
          echo "build-url=$BUILD_URL" >> $GITHUB_OUTPUT
          echo "download-url=$DOWNLOAD_URL" >> $GITHUB_OUTPUT
          
          echo "✅ Production build created:"
          echo "Build ID: $BUILD_ID"
          echo "Build Logs: $BUILD_URL"
          echo "Download URL: $DOWNLOAD_URL"

  production-deploy:
    name: "Production Deploy"
    runs-on: ubuntu-latest
    needs: [code-quality-check, production-build]
    if: github.ref == 'refs/heads/main' && !cancelled() && !failure()
    environment: CI CD
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --no-frozen-lockfile

      - name: Setup EAS CLI
        uses: expo/expo-github-action@v8
        with:
          expo-version: latest
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - name: Deploy OTA update
        run: |
          echo "Publishing production OTA update v${{ needs.code-quality-check.outputs.build-version }}"
          eas update \
            --branch production \
            --platform android \
            --message "Production v${{ needs.code-quality-check.outputs.build-version }} - ${{ github.sha }}" \
            --non-interactive

      - name: Deployment summary
        run: |
          echo "✅ Production deployment completed successfully"
          echo "Version: ${{ needs.code-quality-check.outputs.build-version }}"
          echo "Platform: Android"
          echo "Branch: production"
          
          if [[ "${{ needs.production-build.outputs.download-url }}" != "null" && "${{ needs.production-build.outputs.download-url }}" != "" ]]; then
            echo "📱 New build available: ${{ needs.production-build.outputs.download-url }}"
          fi
          echo "🚀 OTA update published - users will receive automatically"

  preview-deploy:
    name: "Preview Deploy"
    runs-on: ubuntu-latest
    needs: code-quality-check
    if: github.ref == 'refs/heads/develop' || github.event_name == 'pull_request'
    environment: CI CD
    outputs:
      preview-url: ${{ steps.preview-info.outputs.preview-url }}
      qr-url: ${{ steps.preview-info.outputs.qr-url }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --no-frozen-lockfile

      - name: Setup EAS CLI
        uses: expo/expo-github-action@v8
        with:
          expo-version: latest
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - name: Get branch name for preview
        id: branch
        run: |
          if [[ "${{ github.event_name }}" == "pull_request" ]]; then
            BRANCH_NAME="${{ github.head_ref }}"
          else
            BRANCH_NAME="${GITHUB_REF#refs/heads/}"
          fi
          CLEAN_BRANCH=$(echo "$BRANCH_NAME" | sed 's/[^a-zA-Z0-9]/-/g')
          echo "clean-branch=$CLEAN_BRANCH" >> $GITHUB_OUTPUT

      - name: Publish preview update
        id: preview-info
        run: |
          BRANCH_NAME="${{ steps.branch.outputs.clean-branch }}"
          PREVIEW_BRANCH="preview-$BRANCH_NAME"
          VERSION="${{ needs.code-quality-check.outputs.build-version }}"
          
          echo "Publishing preview update: $PREVIEW_BRANCH"
          echo "Version: $VERSION"
          
          eas update \
            --branch="$PREVIEW_BRANCH" \
            --message="Preview v$VERSION - ${{ github.sha }}" \
            --non-interactive
          
          # Generate URLs
          PROJECT_INFO=$(eas project:info --json 2>/dev/null || echo '{"id":"unknown"}')
          PROJECT_ID=$(echo "$PROJECT_INFO" | jq -r '.id // "unknown"' 2>/dev/null || echo "unknown")
          PREVIEW_URL="https://expo.dev/accounts/[username]/projects/[project-name]/updates"
          QR_URL="https://qr.expo.dev/eas-update?updateId=$PREVIEW_BRANCH&appScheme=exp"
          
          echo "preview-url=$PREVIEW_URL" >> $GITHUB_OUTPUT
          echo "qr-url=$QR_URL" >> $GITHUB_OUTPUT
          
          echo "✅ Preview published on branch: $PREVIEW_BRANCH"
          echo "🔗 Dashboard: $PREVIEW_URL"

      - name: Comment on PR with preview info
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const branchName = '${{ steps.branch.outputs.clean-branch }}';
            const version = '${{ needs.code-quality-check.outputs.build-version }}';
            const previewUrl = '${{ steps.preview-info.outputs.preview-url }}';
            
            const message = `🚀 **Preview Update Ready!**
            
            **Version:** \`${version}\`  
            **Branch:** \`preview-${branchName}\`
            
            ### 📱 Test this preview:
            1. Open your development build
            2. Navigate to branch: \`preview-${branchName}\`
            3. Or scan QR code on [Expo Dashboard](${previewUrl})
            
            ### 🔗 Links:
            - [View on Expo Dashboard](${previewUrl})
            - [Build Logs](https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }})
            
            ---
            _Generated automatically by CI/CD Pipeline_`;

            await github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: message
            });
```

### Manual Build & Deploy: `.github/workflows/manual-build-submit.yml`

```yaml
name: Manual Build & Deploy

on:
  workflow_dispatch:
    inputs:
      platform:
        description: 'Platform to build'
        required: true
        default: 'android'
        type: choice
        options:
          - android
          # - ios  # Disabled - no Apple Developer account yet
      profile:
        description: 'Build profile'
        required: true
        default: 'production'
        type: choice
        options:
          - development
          - preview
          - production
      submit_to_store:
        description: 'Submit to app store after build'
        required: false
        default: false
        type: boolean
      custom_version:
        description: 'Custom version (optional)'
        required: false
        default: ''
        type: string

jobs:
  validate-and-prepare:
    name: "Validate & Prepare"
    runs-on: ubuntu-latest
    outputs:
      should-build: ${{ steps.validation.outputs.should-build }}
      should-submit: ${{ steps.validation.outputs.should-submit }}
      build-version: ${{ steps.version.outputs.build-version }}
      profile-type: ${{ steps.validation.outputs.profile-type }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Validate inputs and determine profile type
        id: validation
        run: |
          echo "should-build=true" >> $GITHUB_OUTPUT
          
          if [[ "${{ github.event.inputs.submit_to_store }}" == "true" ]]; then
            echo "should-submit=true" >> $GITHUB_OUTPUT
          else
            echo "should-submit=false" >> $GITHUB_OUTPUT
          fi
          
          # Determine profile type for versioning
          case "${{ github.event.inputs.profile }}" in
            "production")
              echo "profile-type=release" >> $GITHUB_OUTPUT
              ;;
            "preview")
              echo "profile-type=beta" >> $GITHUB_OUTPUT
              ;;
            "development")
              echo "profile-type=dev" >> $GITHUB_OUTPUT
              ;;
          esac

      - name: Validate app.json
        run: |
          echo "Validating app.json file..."
          if [ ! -f app.json ]; then
            echo "❌ app.json not found in current directory!"
            echo "Current directory: $(pwd)"
            echo "Files in current directory:"
            ls -la
            exit 1
          fi
          
          echo "✅ app.json found, validating JSON syntax..."
          if ! cat app.json | jq empty; then
            echo "❌ app.json contains invalid JSON!"
            echo "Content preview:"
            head -20 app.json
            exit 1
          fi
          echo "✅ app.json is valid JSON"

      - name: Generate build version
        id: version
        run: |
          if [[ -n "${{ github.event.inputs.custom_version }}" ]]; then
            BUILD_VERSION="${{ github.event.inputs.custom_version }}"
            echo "✅ Using custom version: $BUILD_VERSION"
          else
            # Get current version from app.json with error handling
            CURRENT_VERSION=$(cat app.json | jq -er '.expo.version // "1.0.0"' || echo "1.0.0")
            
            if [[ "$CURRENT_VERSION" == "1.0.0" ]]; then
              echo "⚠️  Using fallback version 1.0.0 (expo.version not found in app.json)"
            else
              echo "✅ Found version in app.json: $CURRENT_VERSION"
            fi
            
            # Generate version based on profile
            case "${{ github.event.inputs.profile }}" in
              "production")
                BUILD_VERSION="$CURRENT_VERSION"
                ;;
              "preview")
                TIMESTAMP=$(date +%Y%m%d-%H%M%S)
                BUILD_VERSION="$CURRENT_VERSION-beta-$TIMESTAMP"
                ;;
              "development")
                TIMESTAMP=$(date +%Y%m%d-%H%M%S)
                SHORT_SHA=${GITHUB_SHA:0:7}
                BUILD_VERSION="$CURRENT_VERSION-dev-$TIMESTAMP-$SHORT_SHA"
                ;;
            esac
          fi
          
          echo "build-version=$BUILD_VERSION" >> $GITHUB_OUTPUT
          echo "Generated version: $BUILD_VERSION for ${{ github.event.inputs.profile }} profile"

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --no-frozen-lockfile

      - name: Run code quality checks
        run: |
          echo "Running code quality checks for ${{ github.event.inputs.profile }} build..."
          pnpm run lint
          npx tsc --noEmit
          npx expo-doctor

  create-build:
    name: "Create ${{ github.event.inputs.profile }} Build"
    runs-on: ubuntu-latest
    needs: validate-and-prepare
    if: needs.validate-and-prepare.outputs.should-build == 'true'
    environment: CI CD
    outputs:
      build-id: ${{ steps.build.outputs.build-id }}
      build-url: ${{ steps.build.outputs.build-url }}
      download-url: ${{ steps.build.outputs.download-url }}
      qr-url: ${{ steps.build.outputs.qr-url }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --no-frozen-lockfile

      - name: Setup EAS CLI
        uses: expo/expo-github-action@v8
        with:
          expo-version: latest
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - name: Build application
        id: build
        run: |
          PLATFORM="${{ github.event.inputs.platform }}"
          PROFILE="${{ github.event.inputs.profile }}"
          VERSION="${{ needs.validate-and-prepare.outputs.build-version }}"
          
          echo "Building $PLATFORM app with $PROFILE profile"
          echo "Version: $VERSION"
          
          # Create build and capture output
          BUILD_OUTPUT=$(eas build \
            --platform="$PLATFORM" \
            --profile="$PROFILE" \
            --non-interactive \
            --wait \
            --json)
          
          if [[ -z "$BUILD_OUTPUT" ]] || [[ "$BUILD_OUTPUT" == "null" ]]; then
            echo "❌ Build failed or returned empty output"
            exit 1
          fi
          
          # Extract build information with error handling
          BUILD_ID=$(echo "$BUILD_OUTPUT" | jq -r '.[0].id // "unknown"' 2>/dev/null || echo "unknown")
          BUILD_URL=$(echo "$BUILD_OUTPUT" | jq -r '.[0].logsUrl // "unknown"' 2>/dev/null || echo "unknown")
          DOWNLOAD_URL=$(echo "$BUILD_OUTPUT" | jq -r '.[0].artifacts.buildUrl // "pending"' 2>/dev/null || echo "pending")
          
          # Generate QR code URL for development builds
          if [[ "$PROFILE" == "development" ]]; then
            PROJECT_INFO=$(eas project:info --json 2>/dev/null || echo '{"id":"unknown"}')
            PROJECT_ID=$(echo "$PROJECT_INFO" | jq -r '.id // "unknown"' 2>/dev/null || echo "unknown")
            QR_URL="https://qr.expo.dev/development-build?url=exp://$PROJECT_ID"
          else
            QR_URL="N/A"
          fi
          
          echo "build-id=$BUILD_ID" >> $GITHUB_OUTPUT
          echo "build-url=$BUILD_URL" >> $GITHUB_OUTPUT
          echo "download-url=$DOWNLOAD_URL" >> $GITHUB_OUTPUT
          echo "qr-url=$QR_URL" >> $GITHUB_OUTPUT
          
          echo "✅ Build completed successfully:"
          echo "Build ID: $BUILD_ID"
          echo "Version: $VERSION"
          echo "Profile: $PROFILE"
          echo "Platform: $PLATFORM"
          echo "Build Logs: $BUILD_URL"
          echo "Download URL: $DOWNLOAD_URL"

  submit-to-store:
    name: "Submit to ${{ github.event.inputs.platform }} Store"
    runs-on: ubuntu-latest
    needs: [validate-and-prepare, create-build]
    if: needs.validate-and-prepare.outputs.should-submit == 'true' && needs.create-build.outputs.build-id != ''
    environment: CI CD
    outputs:
      submission-id: ${{ steps.submit.outputs.submission-id }}
      submission-url: ${{ steps.submit.outputs.submission-url }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --no-frozen-lockfile

      - name: Setup EAS CLI
        uses: expo/expo-github-action@v8
        with:
          expo-version: latest
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - name: Submit to app store
        id: submit
        run: |
          PLATFORM="${{ github.event.inputs.platform }}"
          BUILD_ID="${{ needs.create-build.outputs.build-id }}"
          
          echo "Submitting build $BUILD_ID to $PLATFORM store..."
          
          if [[ "$PLATFORM" == "android" ]]; then
            SUBMIT_OUTPUT=$(eas submit \
              --platform=android \
              --id="$BUILD_ID" \
              --non-interactive \
              --wait \
              --json)
            
            if [[ -z "$SUBMIT_OUTPUT" ]] || [[ "$SUBMIT_OUTPUT" == "null" ]]; then
              echo "❌ Submission failed or returned empty output"
              exit 1
            fi
            
            SUBMISSION_ID=$(echo "$SUBMIT_OUTPUT" | jq -r '.id // "unknown"' 2>/dev/null || echo "unknown")
            SUBMISSION_URL=$(echo "$SUBMIT_OUTPUT" | jq -r '.logsUrl // "unknown"' 2>/dev/null || echo "unknown")
            
            echo "submission-id=$SUBMISSION_ID" >> $GITHUB_OUTPUT
            echo "submission-url=$SUBMISSION_URL" >> $GITHUB_OUTPUT
            
            echo "✅ Successfully submitted to Google Play Store"
            echo "Submission ID: $SUBMISSION_ID"
          else
            echo "❌ iOS submissions not configured - Apple Developer account required"
            exit 1
          fi

  deployment-summary:
    name: "Deployment Summary"
    runs-on: ubuntu-latest
    needs: [validate-and-prepare, create-build, submit-to-store]
    if: always()
    steps:
      - name: Generate deployment report
        run: |
          echo "# 📊 Manual Build & Deploy Summary"
          echo ""
          echo "**Triggered by:** ${{ github.actor }}"
          echo "**Timestamp:** $(date)"
          echo "**Repository:** ${{ github.repository }}"
          echo "**Commit:** ${{ github.sha }}"
          echo ""
          echo "## Configuration"
          echo "- **Platform:** ${{ github.event.inputs.platform }}"
          echo "- **Profile:** ${{ github.event.inputs.profile }}"
          echo "- **Version:** ${{ needs.validate-and-prepare.outputs.build-version }}"
          echo "- **Submit to Store:** ${{ github.event.inputs.submit_to_store }}"
          echo ""
          echo "## Results"
          
          # Validation results
          if [[ "${{ needs.validate-and-prepare.result }}" == "success" ]]; then
            echo "✅ **Validation:** Passed"
          else
            echo "❌ **Validation:** Failed"
          fi
          
          # Build results
          if [[ "${{ needs.create-build.result }}" == "success" ]]; then
            echo "✅ **Build:** Successful"
            echo "   - Build ID: ${{ needs.create-build.outputs.build-id }}"
            echo "   - [Build Logs](${{ needs.create-build.outputs.build-url }})"
            
            if [[ "${{ needs.create-build.outputs.download-url }}" != "null" && "${{ needs.create-build.outputs.download-url }}" != "" ]]; then
              echo "   - [📱 Download APK](${{ needs.create-build.outputs.download-url }})"
            fi
            
            if [[ "${{ needs.create-build.outputs.qr-url }}" != "N/A" ]]; then
              echo "   - [📱 Development QR Code](${{ needs.create-build.outputs.qr-url }})"
            fi
          else
            echo "❌ **Build:** Failed"
          fi
          
          # Submission results
          if [[ "${{ github.event.inputs.submit_to_store }}" == "true" ]]; then
            if [[ "${{ needs.submit-to-store.result }}" == "success" ]]; then
              echo "✅ **Store Submission:** Successful"
              echo "   - Submission ID: ${{ needs.submit-to-store.outputs.submission-id }}"
              echo "   - [Submission Logs](${{ needs.submit-to-store.outputs.submission-url }})"
            else
              echo "❌ **Store Submission:** Failed"
            fi
          else
            echo "⏭️ **Store Submission:** Skipped (not requested)"
          fi
          
          echo ""
          echo "---"
          echo "_Workflow completed at $(date)_"
```

## 🧪 Testing and Validation

### Test Your Setup

1. **Test Code Quality Check:**
   ```bash
   # Create a feature branch
   git checkout -b test/ci-cd-setup
   
   # Make a small change
   echo "// Test change" >> app.json
   
   # Push to trigger workflow
   git add .
   git commit -m "Test CI/CD setup"
   git push origin test/ci-cd-setup
   ```

2. **Create Pull Request:**
   - Go to GitHub and create a PR from your test branch
   - Check that preview update is published
   - Verify PR comment is added with preview links

3. **Test Production Deployment:**
   ```bash
   # Merge to main branch
   git checkout main
   git merge test/ci-cd-setup
   git push origin main
   ```

4. **Test Manual Build:**
   - Go to GitHub → Actions → Manual Build & Deploy
   - Choose your options and run the workflow

### Validate Build Results

1. **Check Expo Dashboard:**
   - Visit [expo.dev](https://expo.dev)
   - Go to your project
   - Check Builds and Updates tabs

2. **Verify Download Links:**
   - APK files should be available in Builds tab
   - Preview updates should be visible in Updates tab
   - QR codes should work with development builds

## 🔍 Troubleshooting

### Common Issues and Solutions

#### 1. Authentication Failed
```bash
# Check token validity
eas whoami

# Re-login if needed
eas login
```

#### 2. JSON Parse Errors
- Validate app.json structure
- Ensure no trailing commas
- Check for proper closing brackets

#### 3. Build Failures
```bash
# Check build logs in Expo dashboard
# Common fixes:
- Clear node_modules and reinstall
- Check eas.json configuration
- Verify platform-specific settings
```

#### 4. Permission Errors
- Ensure `EXPO_TOKEN` is in CI CD environment (not repository secrets)
- Check workflow permissions for PR comments

#### 5. Missing Dependencies
```bash
# Install missing packages
pnpm install

# Update package.json scripts if needed
```

### Debug Commands

```bash
# Check EAS project info
eas project:info

# List builds
eas build:list

# Check update branches
eas update:list

# View build logs
eas build:view [build-id]
```

## 📦 Migration to Other Projects

### Checklist for New Projects

#### 1. **Required Files to Copy:**
- `.github/workflows/ci.yml`
- `.github/workflows/manual-build-submit.yml`
- `eas.json` (update project-specific values)

#### 2. **Update Project-Specific Values:**

**In `app.json`:**
```json
{
  "expo": {
    "name": "your-new-project-name",
    "slug": "your-new-project-slug",
    "extra": {
      "eas": {
        "projectId": "your-new-project-id"
      }
    },
    "owner": "your-expo-username",
    "updates": {
      "url": "https://u.expo.dev/your-new-project-id"
    }
  }
}
```

**In workflow files:**
- Update project URLs in PR comments
- Update dashboard URLs
- Adjust branch names if different

#### 3. **Setup Commands for New Project:**
```bash
# Navigate to new project
cd your-new-project

# Install dependencies
pnpm install

# Login to Expo
eas login

# Initialize EAS project
eas project:init

# Configure EAS Update
eas update:configure

# Configure EAS Build
eas build:configure
```

#### 4. **Environment Setup:**
- Create "CI CD" environment in GitHub
- Add `EXPO_TOKEN` secret
- Add optional store submission secrets

#### 5. **Required Package.json Scripts:**
```json
{
  "scripts": {
    "lint": "expo lint",
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios"
  }
}
```

#### 6. **Test the Setup:**
- Create test branch and push
- Verify workflows run successfully
- Check build outputs and preview updates

### Package Manager Compatibility

**For npm projects:**
- Replace `pnpm` with `npm` in workflows
- Change `pnpm install --no-frozen-lockfile` to `npm ci`
- Update cache strategy to `cache: 'npm'`

**For yarn projects:**
- Replace `pnpm` with `yarn` in workflows
- Change `pnpm install --no-frozen-lockfile` to `yarn install --frozen-lockfile`
- Update cache strategy to `cache: 'yarn'`

## 🏆 Best Practices

### Version Management
- Use semantic versioning for production (1.0.0, 1.1.0, 2.0.0)
- Update version in app.json before major releases
- Tag releases in git
- Maintain changelog

### Branch Strategy
- `main`: Production releases
- `develop`: Staging/testing
- `feature/*`: New features
- `hotfix/*`: Critical fixes

### Security
- Never commit tokens to repository
- Use GitHub environments for secrets
- Regularly rotate access tokens
- Limit permissions on service accounts

### Performance
- Use build caching
- Optimize dependencies
- Monitor build times
- Use parallel jobs when possible

---

## 📞 Support

**Need Help?**
- [EAS Documentation](https://docs.expo.dev/eas/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Expo Discord](https://discord.gg/expo)
- [GitHub Issues](https://github.com/expo/expo/issues)

**Repository Issues:**
- Check build logs in GitHub Actions
- Review EAS dashboard for detailed error messages
- Verify all configuration files are correct
- Test locally before pushing changes

---

*This guide covers the complete setup process we implemented, including all troubleshooting steps and solutions. You can use this as a reference for setting up CI/CD in any other Expo React Native project.* 