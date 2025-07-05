# CI/CD Setup Guide - Development Mode

This guide will help you set up continuous integration and deployment for your Expo React Native project using GitHub Actions and EAS (Expo Application Services).

**🚀 This setup is optimized for development without Apple Developer Account - focusing on Android and preview updates.**

## Prerequisites

1. **Expo Account**: Sign up at [expo.dev](https://expo.dev)
2. **EAS CLI**: Install globally with `npm install -g eas-cli`
3. **GitHub Repository**: Your project should be in a GitHub repository

## Required Tokens and Secrets

### 1. Expo Token (Required)

You need to generate an Expo access token for GitHub Actions to authenticate with EAS services.

**Steps:**

1. Go to [expo.dev/accounts/[username]/settings/access-tokens](https://expo.dev/accounts/[username]/settings/access-tokens)
2. Click "Create Token"
3. Name it something like "GitHub Actions CI/CD"
4. Copy the generated token

**Add to GitHub Environment:**

1. Go to your GitHub repository
2. Navigate to Settings → Environments
3. Click "New environment"
4. Name: `CI CD`
5. Click "Configure environment"
6. In the **Environment secrets** section:
   - Click "Add secret"
   - Name: `EXPO_TOKEN`
   - Value: Your Expo access token

### 2. App Store Connect API Key (Optional - for iOS submissions)

If you plan to submit to the Apple App Store automatically:

**Steps:**

1. Generate an App Store Connect API key from [App Store Connect](https://appstoreconnect.apple.com/access/api)
2. Add these secrets to GitHub:
   - `APPLE_API_KEY_ID`: Your API key ID
   - `APPLE_API_ISSUER_ID`: Your issuer ID
   - `APPLE_API_KEY`: The .p8 file content (base64 encoded)

### 3. Google Play Console Service Account (Optional - for Android submissions)

If you plan to submit to Google Play automatically:

**Steps:**

1. Create a service account in Google Cloud Console
2. Download the JSON key file
3. Add this secret to GitHub:
   - `GOOGLE_SERVICE_ACCOUNT_KEY`: The JSON file content (base64 encoded)

## Initial Setup Commands

Run these commands in your project directory:

```bash
# Login to Expo
eas login

# Configure EAS for your project
eas project:init

# Configure EAS Update (for OTA updates)
eas update:configure

# Configure EAS Build
eas build:configure

# Configure EAS Submit (for app store submissions)
eas submit:configure
```

## Workflow Files Created

### 1. **CI/CD Pipeline** (`.github/workflows/ci.yml`)

**Sequential Steps:**
- 🔍 **Test & Validate**: Linting, TypeScript check, Expo validation
- 🏗️ **Build Application**: Create build (main branch only)
- 🚀 **Deploy Update**: Publish OTA update (main branch only)
- 📱 **Preview Update**: Preview for PR/develop branch

**Execution Order:**
```
Test & Validate → Build Application → Deploy Update
                ↘ Preview Update (for PR/develop)
```

### 2. **Manual Build & Submit** (`.github/workflows/manual-build-submit.yml`)

**Sequential Steps:**
- ✅ **Validate & Prepare**: Validation and preparation
- 🏗️ **Build Application**: Create manual build
- 📤 **Submit to Store**: Upload to store (optional)
- 📊 **Summary**: Results summary

**Execution Order:**
```
Validate & Prepare → Build Application → Submit to Store → Summary
```

## EAS Configuration

The `eas.json` file has been created with the following profiles:

- **development**: For development builds with dev client
- **development-simulator**: For iOS simulator builds
- **preview**: For internal testing builds
- **production**: For production builds with auto-increment

## Usage

### 1. **CI/CD Pipeline** (Automatic)

**Steps:**
- **Push/PR** → Test & Validate runs automatically
- **Main branch** → Build + Deploy runs automatically
- **Develop/PR** → Preview Update runs automatically

### 2. **Manual Build & Submit**

1. Go to Actions tab in your GitHub repository
2. Select "Manual Build & Submit" workflow
3. Click "Run workflow"
4. Choose:
   - **Platform**: `android`
   - **Profile**: `development`/`preview`/`production`
   - **Submit to Store**: `true`/`false`

### 3. **Preview Testing**

1. Create new branch: `git checkout -b feature/my-feature`
2. Push changes
3. **Automatic** preview update is published
4. If PR exists, comment is added with preview link

### 4. **Production Deployment**

1. Push to main branch
2. **Automatic** sequential execution:
   - Test & Validate
   - Build Application (if needed)
   - Deploy Update

## 🎯 New Sequential Workflow

```
Push/PR → 1️⃣ Test & Validate
              ↓
Main → 2️⃣ Build Application → 3️⃣ Deploy Update
              ↓
PR/Develop → 4️⃣ Preview Update → PR Comment
```

## Additional Configuration

### Environment Variables

You can add environment variables to your builds by:

1. Adding them to your `eas.json` file
2. Or storing them as GitHub repository secrets

### Branch-based Deployments

The workflows support different deployment strategies:

- **main branch**: Production deployments
- **develop branch**: Preview builds
- **feature branches**: Development builds

### Customization

You can customize the workflows by modifying the YAML files in `.github/workflows/` to match your specific needs.

## Troubleshooting

### Common Issues:

1. **"Authentication failed"**

   - Check that your `EXPO_TOKEN` is correctly set in the "CI CD" environment
   - Verify the token hasn't expired
   - Make sure the token is in the environment, not in repository secrets

2. **"Resource not accessible by integration"**

   - This error occurs when workflows can't access pull requests or issues
   - Ensure workflows that interact with PRs have proper permissions:
     ```yaml
     permissions:
       contents: read
       pull-requests: write
       issues: write
     ```

3. **"Project not found"**

   - Run `eas project:init` to initialize your project
   - Make sure you're logged in with the correct Expo account

4. **Build failures**
   - Check that your `eas.json` configuration is correct
   - Verify all dependencies are properly installed

### Getting Help

- [EAS Documentation](https://docs.expo.dev/eas/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Expo Discord](https://discord.gg/expo)

## Security Notes

- Never commit tokens or secrets to your repository
- Use GitHub's encrypted secrets for all sensitive information
- Regularly rotate your access tokens
- Review and limit permissions on service accounts

---

**Next Steps:**

1. Set up the `EXPO_TOKEN` in your GitHub repository secrets
2. Run the initial setup commands
3. Test the CI workflow by creating a pull request
4. Create your first development build using the GitHub Actions workflow
