# CI/CD Setup Guide - Development Mode

این راهنما برای راه‌اندازی CI/CD برای پروژه Expo React Native شما طراحی شده، **بدون نیاز به Apple Developer Account**.

## 🚀 ویژگی‌های این Setup

- **فقط Android** - هیچ iOS dependency نیست
- **Preview Updates** - تست تغییرات بدون build جدید
- **Development Builds** - برای تست local
- **Production Updates** - OTA updates برای Android
- **Team Collaboration** - PR comments و preview links

## 🔧 Prerequisites

1. **Expo Account**: [expo.dev](https://expo.dev)
2. **EAS CLI**: `npm install -g eas-cli`
3. **GitHub Repository**

## 🔑 تنها توکن مورد نیاز: EXPO_TOKEN

### قدم 1: ایجاد Expo Token

1. برو به [expo.dev/accounts/[username]/settings/access-tokens](https://expo.dev/accounts/[username]/settings/access-tokens)
2. **"Create Token"** کلیک کن
3. نام: `"GitHub Actions CI/CD"`
4. توکن رو کپی کن

### قدم 2: اضافه کردن به GitHub Environment

1. برو به repository → **Settings** → **Environments**
2. **"New environment"** کلیک کن
3. Name: `CI CD`
4. **"Configure environment"**
5. در قسمت **Environment secrets**:
   - **"Add secret"**
   - Name: `EXPO_TOKEN`
   - Value: توکن کپی شده
   - **"Add secret"**

## 📋 Workflows ایجاد شده

### 1. **CI** (`.github/workflows/ci.yml`)

- **تست خودکار** روی هر push/PR
- Linting, TypeScript check, Expo validation
- **هیچ توکنی نیاز نداره**

### 2. **EAS Build** (`.github/workflows/eas-build.yml`)

- **Manual trigger** برای ساخت build
- Development, Preview, Production builds
- **فقط Android** (فعلا)

### 3. **EAS Update** (`.github/workflows/eas-update.yml`)

- **خودکار** روی main branch push
- OTA updates بدون نیاز به build جدید

### 4. **Preview Update** (`.github/workflows/preview-update.yml`)

- **خودکار** روی feature branches
- **PR comments** با preview links
- عالی برای team collaboration

### 5. **Production Deploy** (`.github/workflows/deploy-production.yml`)

- **هوشمند**: اگر build موجود باشه → OTA update
- اگر build نباشه → build جدید
- **فقط Android**

## 🛠️ دستورات اولیه Setup

```bash
# Login to Expo
eas login

# Initialize project
eas project:init

# Configure EAS Update (for OTA updates)
eas update:configure

# Configure EAS Build
eas build:configure
```

## 📱 نحوه استفاده

### 1. **CI Testing**

- هر push/PR خودکار test میشه
- نیازی به کاری نیست

### 2. **Development Build**

1. GitHub → **Actions** → **"EAS Build"**
2. **"Run workflow"**
3. Platform: `android`, Profile: `development`

### 3. **Preview Testing**

1. Branch جدید: `git checkout -b feature/my-feature`
2. تغییرات رو push کن
3. **خودکار** preview update منتشر میشه
4. اگر PR باشه، comment میذاره با link

### 4. **Production Deployment**

1. به main branch push کن
2. **خودکار** production update منتشر میشه

## 🎯 Workflow مخصوص Development

```
Feature Branch → Preview Update (خودکار)
      ↓
Pull Request → Preview Comment (خودکار)
      ↓
Merge to Main → Production Update (خودکار)
```

## 🔄 آینده: وقتی Apple Developer Account گرفتی

فقط کافیه comment های iOS رو از workflows حذف کنی و این secrets رو اضافه کنی:

- `APPLE_API_KEY_ID`
- `APPLE_API_ISSUER_ID`
- `APPLE_API_KEY`

## 🆘 Troubleshooting

### خطای "Authentication failed"

- `EXPO_TOKEN` رو چک کن
- Token expire نشده باشه
- مطمئن شو که توکن در environment "CI CD" هست، نه در repository secrets

### خطای "Project not found"

- `eas project:init` رو اجرا کن
- با اکانت درست login کرده باشی

### Build failure

- `eas.json` رو چک کن
- Dependencies نصب باشن

## 🚀 Next Steps

1. ✅ `EXPO_TOKEN` رو set کن
2. ✅ دستورات setup رو اجرا کن
3. ✅ یک PR بساز (تست CI)
4. ✅ اولین development build رو بساز
5. ✅ یک feature branch بساز (تست preview)

---

**🎉 تبریک! حالا یک CI/CD کامل برای development داری!**

برای سوال یا مشکل: [Expo Discord](https://discord.gg/expo)
