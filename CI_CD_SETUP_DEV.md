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

### 1. **CI/CD Pipeline** (`.github/workflows/ci.yml`)

**مراحل Sequential:**
- 🔍 **Test & Validate**: Linting, TypeScript check, Expo validation
- 🏗️ **Build Application**: ساخت build (فقط main branch)
- 🚀 **Deploy Update**: انتشار OTA update (فقط main branch)  
- 📱 **Preview Update**: Preview برای PR/develop branch

**ترتیب اجرا:**
```
Test & Validate → Build Application → Deploy Update
                ↘ Preview Update (for PR/develop)
```

### 2. **Manual Build & Submit** (`.github/workflows/manual-build-submit.yml`)

**مراحل Sequential:**
- ✅ **Validate & Prepare**: اعتبارسنجی و آماده‌سازی
- 🏗️ **Build Application**: ساخت manual build
- 📤 **Submit to Store**: آپلود به استور (اختیاری)
- 📊 **Summary**: خلاصه نتایج

**ترتیب اجرا:**
```
Validate & Prepare → Build Application → Submit to Store → Summary
```

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

### 1. **CI/CD Pipeline** (خودکار)

**مراحل:**
- **Push/PR** → Test & Validate اجرا میشه
- **Main branch** → Build + Deploy اجرا میشه
- **Develop/PR** → Preview Update اجرا میشه

### 2. **Manual Build & Submit**

1. GitHub → **Actions** → **"Manual Build & Submit"**
2. **"Run workflow"**
3. انتخاب کن:
   - **Platform**: `android`
   - **Profile**: `development`/`preview`/`production`
   - **Submit to Store**: `true`/`false`

### 3. **Preview Testing**

1. Branch جدید: `git checkout -b feature/my-feature`
2. تغییرات رو push کن
3. **خودکار** preview update منتشر میشه
4. اگر PR باشه، comment میذاره با link

### 4. **Production Deployment**

1. به main branch push کن
2. **خودکار** مراحل زیر اجرا میشه:
   - Test & Validate
   - Build Application (if needed)
   - Deploy Update

## 🎯 Workflow جدید Sequential

```
Push/PR → 1️⃣ Test & Validate
              ↓
Main → 2️⃣ Build Application → 3️⃣ Deploy Update
              ↓
PR/Develop → 4️⃣ Preview Update → PR Comment
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

### خطای "Resource not accessible by integration"

- این خطا وقتی workflow نتونه با PR/Issue کار کنه
- مطمئن شو که workflow فایل‌هایی که با PR کار میکنن، `permissions` داشته باشن:
  ```yaml
  permissions:
    contents: read
    pull-requests: write
    issues: write
  ```

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
