# Invoice Generator - Android App Setup Guide

Complete guide to building and running the native Android app for Invoice Generator.

## Prerequisites

- **Android Studio**: Version 2023.1 or later
- **Android SDK**: API 24 (Android 7.0) or higher
- **Java/Kotlin**: JDK 17 or later
- **Gradle**: 8.0 or later
- **Git**: For version control

## Project Structure

```
invoice-generator-android/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── kotlin/com/invoicegenerator/
│   │   │   │   ├── MainActivity.kt
│   │   │   │   ├── data/
│   │   │   │   │   ├── api/
│   │   │   │   │   ├── database/
│   │   │   │   │   └── preferences/
│   │   │   │   ├── domain/
│   │   │   │   │   └── repository/
│   │   │   │   ├── di/
│   │   │   │   └── presentation/
│   │   │   │       ├── navigation/
│   │   │   │       ├── screens/
│   │   │   │       └── theme/
│   │   │   ├── AndroidManifest.xml
│   │   │   └── res/
│   │   └── test/
│   ├── build.gradle.kts
│   └── proguard-rules.pro
├── build.gradle.kts
├── settings.gradle.kts
└── gradle/
```

## Installation Steps

### 1. Clone or Download Project

```bash
# If using Git
git clone <repository-url> invoice-generator-android
cd invoice-generator-android

# Or extract the provided files
cd invoice-generator-android
```

### 2. Open in Android Studio

1. Launch Android Studio
2. Click "Open" or "File" → "Open"
3. Navigate to the `invoice-generator-android` directory
4. Click "OK"

Android Studio will automatically:
- Download Gradle wrapper
- Install required SDK components
- Build the project

### 3. Configure Backend URL

Edit `app/src/main/kotlin/com/invoicegenerator/data/api/RetrofitClient.kt`:

```kotlin
private const val BASE_URL = "https://your-backend-api.com/api/trpc/"
```

Replace with your actual backend URL (e.g., the web app's backend).

### 4. Configure OAuth

For OAuth integration with Manus:

1. Update `AndroidManifest.xml` with your OAuth redirect scheme:
```xml
<data
    android:host="oauth-callback"
    android:scheme="invoicegenerator" />
```

2. Configure OAuth credentials in your backend

### 5. Build the Project

```bash
# Using Gradle
./gradlew build

# Or in Android Studio:
# Build → Make Project
```

### 6. Run on Emulator

1. **Create Virtual Device** (if needed):
   - Tools → Device Manager
   - Create Device
   - Select API 24 or higher
   - Finish

2. **Run App**:
   - Select device from dropdown
   - Click "Run" (Shift+F10)
   - Or: Run → Run 'app'

### 7. Run on Physical Device

1. **Enable Developer Mode**:
   - Settings → About Phone
   - Tap "Build Number" 7 times
   - Settings → Developer Options → USB Debugging (ON)

2. **Connect Device**:
   - Connect via USB cable
   - Allow USB debugging when prompted

3. **Run App**:
   - Device should appear in Android Studio
   - Click "Run"

## Key Features Implemented

### Data Layer
- **Room Database**: Local invoice, line items, and branding template storage
- **Retrofit API**: Type-safe HTTP communication with backend
- **DataStore**: Secure token and user preference storage
- **Offline-First**: All data saved locally before syncing

### Domain Layer
- **Repository Pattern**: Clean separation of data sources
- **Offline Sync**: Automatic sync when online
- **Conflict Resolution**: Handle offline changes

### Presentation Layer
- **Jetpack Compose**: Modern UI framework
- **Navigation**: Type-safe navigation between screens
- **Material Design 3**: Professional UI components
- **Dark Mode**: System-aware theme switching

### Architecture
- **MVVM**: Model-View-ViewModel pattern
- **Hilt**: Dependency injection
- **Coroutines**: Asynchronous operations
- **Flow**: Reactive data streams

## Gradle Dependencies

Key dependencies included:

```kotlin
// Jetpack Compose
androidx.compose.ui:ui:1.5.4
androidx.compose.material3:material3:1.1.2

// Room (Database)
androidx.room:room-runtime:2.6.1

// Retrofit (HTTP)
com.squareup.retrofit2:retrofit:2.10.0

// Hilt (DI)
com.google.dagger:hilt-android:2.48

// PDF Generation
com.itextpdf:itextg:5.5.10

// QR Codes
com.google.zxing:core:3.5.2

// And many more...
```

## Configuration Files

### build.gradle.kts (App Level)

Configures:
- SDK versions (min: 24, target: 34)
- Dependencies
- Build types (debug/release)
- Compose options

### AndroidManifest.xml

Declares:
- Required permissions (internet, storage, camera)
- Activities
- OAuth callback intent filter
- WhatsApp package queries

### proguard-rules.pro

Protects:
- API models from obfuscation
- Third-party libraries
- Kotlin/Coroutines classes

## Development Workflow

### 1. Create New Screen

1. Create file in `presentation/screens/`
2. Build Composable function
3. Add route to `Screen` sealed class
4. Add navigation in `MainActivity.kt`

### 2. Add Database Entity

1. Create entity in `data/database/entity/`
2. Create DAO in `data/database/dao/`
3. Add to `InvoiceDatabase`
4. Run migrations

### 3. Add API Endpoint

1. Add method to `InvoiceApiService`
2. Create request/response models
3. Add repository method
4. Use in ViewModel

### 4. Implement Offline Sync

1. Mark changes with `syncStatus`
2. Implement sync logic in repository
3. Use `getPendingSyncInvoices()`
4. Sync when online

## Testing

### Unit Tests

```bash
./gradlew test
```

### Instrumented Tests

```bash
./gradlew connectedAndroidTest
```

### Manual Testing Checklist

- [ ] Create invoice offline
- [ ] Verify local storage
- [ ] Go online and sync
- [ ] Search invoices
- [ ] Filter by status
- [ ] Update status
- [ ] Export PDF
- [ ] Share via WhatsApp
- [ ] Dark mode toggle
- [ ] Handle network errors

## Debugging

### Logcat

```bash
# View all logs
adb logcat

# Filter by tag
adb logcat | grep "InvoiceGenerator"

# Or use Android Studio Logcat tab
```

### Database Inspector

In Android Studio:
1. App Inspector
2. Database Inspector
3. Select database
4. Browse tables

### Network Inspector

Monitor HTTP requests:
1. Profiler
2. Network
3. View requests/responses

## Building Release APK

### 1. Create Keystore

```bash
keytool -genkey -v -keystore release.keystore \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias release
```

### 2. Configure Signing

Edit `app/build.gradle.kts`:

```kotlin
signingConfigs {
    create("release") {
        storeFile = file("../release.keystore")
        storePassword = "your-password"
        keyAlias = "release"
        keyPassword = "your-password"
    }
}

buildTypes {
    release {
        signingConfig = signingConfigs.getByName("release")
    }
}
```

### 3. Build Release APK

```bash
./gradlew assembleRelease
```

APK location: `app/build/outputs/apk/release/app-release.apk`

### 4. Build App Bundle (for Play Store)

```bash
./gradlew bundleRelease
```

Bundle location: `app/build/outputs/bundle/release/app-release.aab`

## Publishing to Google Play Store

1. Create Google Play Developer account
2. Create app in Play Console
3. Upload app bundle
4. Fill app details
5. Submit for review

## Troubleshooting

### Build Errors

**Error**: `Gradle sync failed`
- Solution: File → Sync Now
- Check SDK versions in `build.gradle.kts`

**Error**: `Cannot find symbol`
- Solution: Build → Clean Project
- Then Build → Rebuild Project

**Error**: `Compilation failed`
- Solution: Check Kotlin version compatibility
- Update Gradle to latest version

### Runtime Errors

**Error**: `Database not found`
- Solution: Database is created on first run
- Check app permissions

**Error**: `Network timeout`
- Solution: Check backend URL in `RetrofitClient.kt`
- Verify backend is running
- Check device internet connection

**Error**: `OAuth callback not working`
- Solution: Verify OAuth redirect scheme in manifest
- Check backend OAuth configuration

### Performance Issues

**Slow startup**
- Enable ProGuard/R8 minification
- Lazy load screens
- Use background threads

**Memory leaks**
- Use Android Studio Profiler
- Check for context leaks
- Verify coroutine cancellation

## Offline Functionality

### How It Works

1. **Create Offline**: Invoice saved to local database immediately
2. **Mark Pending**: `syncStatus = "PENDING"`
3. **Detect Online**: Monitor network connectivity
4. **Auto Sync**: Upload pending changes when online
5. **Conflict Resolution**: Handle conflicts if offline changes conflict

### Sync Status Values

- `PENDING`: Waiting to sync
- `SYNCED`: Successfully synced
- `FAILED`: Sync failed (will retry)

### Testing Offline

1. Enable Airplane Mode
2. Create invoice
3. Verify saved locally
4. Disable Airplane Mode
5. Verify auto-sync

## Performance Optimization

### Database
- Use indexes on frequently queried fields
- Implement pagination for large lists
- Use `Flow` for reactive updates

### Network
- Implement request caching
- Use compression
- Batch requests when possible

### UI
- Use `remember` to avoid recomposition
- Lazy load images
- Implement pagination

## Security Best Practices

1. **Token Storage**: Use DataStore (encrypted)
2. **HTTPS Only**: All API calls use HTTPS
3. **Input Validation**: Validate all user input
4. **Permissions**: Request only needed permissions
5. **Proguard**: Enable code obfuscation

## Deployment Checklist

- [ ] Update backend URL
- [ ] Configure OAuth
- [ ] Test all features
- [ ] Test offline mode
- [ ] Test on multiple devices
- [ ] Create signing keystore
- [ ] Build release APK
- [ ] Test release build
- [ ] Upload to Play Store
- [ ] Monitor crash reports

## Support & Resources

### Documentation
- [Android Developer Docs](https://developer.android.com)
- [Jetpack Compose](https://developer.android.com/jetpack/compose)
- [Room Database](https://developer.android.com/training/data-storage/room)
- [Retrofit](https://square.github.io/retrofit/)

### Tools
- Android Studio
- Android Emulator
- Android Device Monitor
- Logcat

### Community
- Stack Overflow
- Android Developers Subreddit
- Google Android Developers YouTube

## Version History

### v1.0.0 (Current)
- Initial Android app
- Offline-first architecture
- Jetpack Compose UI
- Room database
- Retrofit API integration
- OAuth authentication

### Future Versions
- Push notifications
- Widget support
- Biometric authentication
- Advanced offline sync
- Payment processing

---

**Last Updated**: April 11, 2026  
**Status**: Production Ready  
**Minimum SDK**: 24 (Android 7.0)  
**Target SDK**: 34 (Android 14)
