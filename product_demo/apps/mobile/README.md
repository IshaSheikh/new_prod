# School SaaS Mobile App (Flutter)

This is the mobile application for the School SaaS Platform, built with Flutter. It serves parents, students, and teachers.

## Prerequisites

- Flutter SDK (latest stable)
- Android Studio / Xcode

## Local Development Setup

1. Copy the environment configuration:
   ```bash
   cp .env.example .env
   ```
2. Install dependencies:
   ```bash
   flutter pub get
   ```
3. Run code generation (for Riverpod, Freezed, etc.):
   ```bash
   flutter pub run build_runner build --delete-conflicting-outputs
   ```
4. Start the application:
   ```bash
   flutter run
   ```

## Architecture & Standards

- **State Management**: [Riverpod](https://riverpod.dev/).
- **Networking**: Dio with interceptors for token refresh and tenant context injection.
- **Routing**: GoRouter for declarative routing.
- **UI Architecture**: Feature-based folder structure (`lib/features/`).

## Testing

```bash
flutter test
```
