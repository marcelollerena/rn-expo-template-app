# React Native Expo Template

A template for building React Native apps with [Expo](https://expo.dev), organized around a feature-based architecture.

## Project Structure

```text
src/
├── app/          # Expo Router route files (thin wrappers)
├── common/       # Shared components, hooks, theme, and styles
├── core/         # App-wide infrastructure (API client, query keys)
└── features/     # Feature modules (screens, components per feature)
```

For full architecture details, conventions, and guidelines see [docs/architecture/architecture.md](docs/architecture/architecture.md).

## Getting Started

1. Install dependencies

   ```bash
   pnpm install
   ```

2. Start the app

   ```bash
   npx expo start
   ```

3. Open in your preferred environment:
   - [Development build](https://docs.expo.dev/develop/development-builds/introduction/)
   - [Android emulator](https://docs.expo.dev/workflow/android-studio-emulator/)
   - [iOS simulator](https://docs.expo.dev/workflow/ios-simulator/)
   - [Expo Go](https://expo.dev/go)

## Learn More

- [Expo documentation](https://docs.expo.dev/)
- [Expo Router (file-based routing)](https://docs.expo.dev/router/introduction)
