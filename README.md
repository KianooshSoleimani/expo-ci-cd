# CI/CD Project

A modern React Native app with Expo and comprehensive CI/CD pipeline using GitHub Actions and EAS.

## 🚀 Features

- **React Native** with Expo SDK
- **TypeScript** for type safety
- **Automatic CI/CD** with GitHub Actions
- **EAS Build** for native builds
- **EAS Update** for over-the-air updates
- **Branch-based deployments**
- **Automatic versioning**
- **Preview updates** for testing

## 📁 Project Structure

```
ci-cd/
├── app/                    # Expo Router app directory
│   ├── (tabs)/            # Tab-based navigation
│   ├── _layout.tsx        # Root layout
│   └── +not-found.tsx     # 404 page
├── components/            # Reusable components
├── constants/             # App constants
├── hooks/                 # Custom hooks
├── assets/                # Images and fonts
├── .github/workflows/     # CI/CD workflows
├── eas.json              # EAS configuration
├── app.json              # Expo configuration
└── package.json          # Dependencies
```

## 🔄 CI/CD Pipeline

### Automatic Workflows

#### **Main Branch** (`main`)
```
Push → Code Quality Check → Production Build → Production Deploy
```
- Runs full production build process
- Uses clean version number (e.g., `1.0.0`)
- Publishes OTA update to production users
- Only builds if no existing production build exists

#### **Develop Branch** (`develop`)
```
Push → Code Quality Check → Preview Deploy
```
- Skips production build
- Uses staging version format (e.g., `1.0.0-staging-20241201-143022`)
- Publishes preview update for testing

#### **Feature Branches** & **Pull Requests**
```
Push/PR → Code Quality Check → Preview Deploy → PR Comment
```
- Creates preview updates for testing
- Automatically comments on PR with preview links
- Uses development version format with commit hash

### Manual Workflow

**Manual Build & Deploy** - Triggered manually with options:
```
Validate & Prepare → Create Build → Submit to Store → Summary
```
- Choose platform, profile, and submission options
- Custom version support
- Comprehensive deployment summary

## 📱 Version Strategy

### Production Builds (main branch)
- **Format**: `{version}` (e.g., `1.0.0`)
- **Source**: Uses version from `app.json`

### Preview Builds (develop branch)
- **Format**: `{version}-staging-{timestamp}`
- **Example**: `1.0.0-staging-20241201-143022`

### Development Builds (feature branches)
- **Format**: `{version}-dev-{timestamp}-{commit-hash}`
- **Example**: `1.0.0-dev-20241201-143022-a1b2c3d`

### Pull Request Builds
- **Format**: `{version}-preview-{timestamp}-{commit-hash}`
- **Example**: `1.0.0-preview-20241201-143022-a1b2c3d`

## 🛠️ Development Setup

### Prerequisites

1. **Node.js** (v20+)
2. **pnpm** package manager
3. **Expo CLI** (`npm install -g @expo/cli`)
4. **EAS CLI** (`npm install -g eas-cli`)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/ci-cd.git
cd ci-cd

# Install dependencies
pnpm install

# Start development server
pnpm start
```

### EAS Setup

```bash
# Login to Expo
eas login

# Configure EAS (if needed)
eas project:init
eas update:configure
eas build:configure
```

## 🎯 Build Profiles

### Development Profile
```json
{
  "developmentClient": true,
  "distribution": "internal",
  "android": {
    "buildType": "developmentClient"
  }
}
```

### Preview Profile
```json
{
  "distribution": "internal",
  "android": {
    "buildType": "apk"
  }
}
```

### Production Profile
```json
{
  "distribution": "store",
  "android": {
    "buildType": "app-bundle"
  }
}
```

## 🔗 Download Links

### Production Builds
- **APK**: Available on Expo Dashboard
- **Build Logs**: GitHub Actions logs
- **Store**: Google Play Store (when submitted)

### Preview Updates
- **Development Builds**: Access via branch selection
- **QR Codes**: Available on Expo Dashboard
- **PR Comments**: Automatic links in pull requests

## 📋 Available Scripts

```bash
# Development
pnpm start          # Start Expo development server
pnpm android        # Run on Android device/emulator
pnpm ios            # Run on iOS device/simulator
pnpm web            # Run on web browser

# Code Quality
pnpm lint           # Run ESLint
pnpm lint:fix       # Fix ESLint issues
pnpm type-check     # Run TypeScript checks

# Build & Deploy
pnpm build          # Create production build
pnpm export         # Export for web deployment
```

## 🌐 Deployment

### Automatic Deployment
- **Production**: Push to `main` branch
- **Preview**: Push to `develop` branch or create PR
- **Feature Testing**: Push to feature branches

### Manual Deployment
1. Go to **Actions** tab in GitHub repository
2. Select **"Manual Build & Deploy"** workflow
3. Click **"Run workflow"**
4. Choose your options:
   - **Platform**: `android` (iOS disabled - no Apple Developer account)
   - **Profile**: `development`/`preview`/`production`
   - **Submit to Store**: `true`/`false`
   - **Custom Version**: Optional custom version

## 🔐 Environment Setup

### Required Secrets
Add these to your GitHub repository environment (`CI CD`):

- `EXPO_TOKEN`: Your Expo access token

### Optional Secrets (for store submissions)
- `GOOGLE_SERVICE_ACCOUNT_KEY`: Google Play Console service account
- `APPLE_API_KEY_ID`: Apple App Store Connect API key ID
- `APPLE_API_ISSUER_ID`: Apple App Store Connect issuer ID
- `APPLE_API_KEY`: Apple App Store Connect API key

## 📚 Documentation

- **[CI/CD Setup Guide](CI_CD_SETUP.md)** - Complete setup instructions
- **[Versioning & Downloads](VERSIONING_AND_DOWNLOADS.md)** - Version strategy and download links
- **[EAS Documentation](https://docs.expo.dev/eas/)** - Official EAS documentation
- **[Expo Router](https://expo.github.io/router/)** - Navigation documentation

## 🔍 Troubleshooting

### Common Issues

1. **Build Failures**
   - Check GitHub Actions logs
   - Verify EAS configuration
   - Ensure all dependencies are installed

2. **Authentication Errors**
   - Verify `EXPO_TOKEN` is set in environment
   - Check token hasn't expired
   - Ensure token has proper permissions

3. **Preview Updates Not Showing**
   - Check EAS Update branch name
   - Verify development build is on correct branch
   - Ensure update was published successfully

### Getting Help

- Check the [Troubleshooting Guide](CI_CD_SETUP.md#troubleshooting)
- Review [EAS Documentation](https://docs.expo.dev/eas/)
- Visit [Expo Discord](https://discord.gg/expo) for community support

## 📈 Project Status

- ✅ **CI/CD Pipeline**: Fully configured
- ✅ **Android Builds**: Working
- ⏳ **iOS Builds**: Disabled (no Apple Developer account)
- ✅ **OTA Updates**: Working
- ✅ **Preview Updates**: Working
- ✅ **Store Submission**: Android ready

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

The CI/CD pipeline will automatically:
- Run code quality checks
- Create preview updates
- Comment on the PR with testing instructions

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Built with ❤️ using Expo and EAS**
