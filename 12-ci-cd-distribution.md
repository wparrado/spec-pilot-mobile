# 12 — CI / CD & Distribution

## GitHub Actions (recommended)

The `.github/workflows/ci.yml` in this repository is the canonical template. It runs:

1. **Format check** — `cargo fmt --check` (or language equivalent)
2. **Lint** — `cargo clippy -- -D warnings`
3. **Tests** — `cargo test` with `INSTA_REQUIRE_SNAPSHOTS=1`
4. **Build** — `cargo build --release`
5. **Security audit** — `cargo audit`

### Mobile app CI additions

Add these jobs to the mobile project's own CI:

```yaml
  mobile-test:
    name: Mobile Tests
    runs-on: macos-latest  # required for iOS

    steps:
      - uses: actions/checkout@v4

      - name: Flutter test
        run: flutter test --coverage

      - name: Check coverage threshold
        run: |
          COVERAGE=$(lcov --summary coverage/lcov.info 2>&1 | grep "lines" | awk '{print $2}' | tr -d '%')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80%"
            exit 1
          fi

      - name: Architecture tests
        run: flutter test test/architecture/

  distribute-staging:
    name: Distribute to Staging
    needs: [mobile-test]
    if: github.ref == 'refs/heads/main'
    runs-on: macos-latest

    steps:
      - uses: actions/checkout@v4

      - name: Build iOS (staging)
        run: |
          flutter build ipa --flavor staging --export-options-plist ios/ExportOptions-staging.plist

      - name: Upload to TestFlight
        env:
          APP_STORE_CONNECT_API_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY }}
        run: xcrun altool --upload-app -f build/ios/ipa/*.ipa --apiKey $APP_STORE_CONNECT_API_KEY
```

---

## Fastlane (alternative)

```ruby
# Fastfile
lane :staging do
  sync_code_signing(type: "adhoc")
  build_app(scheme: "MyApp-Staging")
  firebase_app_distribution(
    app: ENV["FIREBASE_APP_ID"],
    groups: "internal-testers",
    release_notes: "Build #{ENV['BUILD_NUMBER']}"
  )
end

lane :release do
  sync_code_signing(type: "appstore")
  build_app(scheme: "MyApp-Production")
  upload_to_testflight(skip_waiting_for_build_processing: true)
end
```

---

## Environment Configuration

All environment-specific values go through a build-time injection system — never hardcoded.

```
.env.example   ← committed, documents all required keys
.env.local     ← gitignored, developer overrides
.env.staging   ← CI secret, not committed
.env.production ← CI secret, not committed
```

Example `.env.example`:
```
API_BASE_URL=https://api.example.com
FIREBASE_PROJECT_ID=your-project-id
FEATURE_FLAG_PROVIDER=remote_config
SENTRY_DSN=
```

---

## Version Bumping

Use semantic versioning. Automate via CI on tag push:

```yaml
  - name: Bump version
    if: startsWith(github.ref, 'refs/tags/v')
    run: |
      VERSION=${GITHUB_REF#refs/tags/v}
      sed -i "s/version: .*/version: $VERSION/" pubspec.yaml
```

---

## Distribution Channels

| Channel | When | Tool |
|---------|------|------|
| Firebase App Distribution | Every `main` build | Fastlane / GitHub Actions |
| TestFlight | Every `main` build | Fastlane / `xcrun altool` |
| Play Internal Track | Every `main` build | Fastlane Supply |
| App Store | Tag `v*.*.*` | Fastlane / App Store Connect API |
| Play Production | Tag `v*.*.*` | Fastlane Supply |
