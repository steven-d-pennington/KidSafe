# Development environment and first-build plan

Status: proposed development plan, inspected 2026-10-27. This document does not establish a successful build, working current-Android support, or a provisioned development environment.

## Recommendation

Use a personal Pop!_OS machine for Android Studio and interactive Android emulator testing. Use GitHub-hosted Actions runners for reproducible builds, static analysis and automated tests. A paid, always-on cloud server is not a prerequisite. Keep the project, Android SDK, test-device data and signing material off any work laptop.

GitHub Codespaces is an optional browser-based Linux editor/build environment, not an assumed replacement for an accelerated interactive Android emulator. Verify available virtualization, graphics and remote-display capabilities before choosing a fully remote test environment. Keep development machines separate from the eventual production backend.

## Verified repository observations

The inspected default branch is `master`.

- The Android Gradle project lives in `Application/`.
- `README.md` describes a Java/Android app using Firebase Auth, Database and Storage and marks the upstream project as unmaintained.
- `Application/app/build.gradle` declares `compileSdkVersion 28`, `targetSdkVersion 28`, `minSdkVersion 19`, and legacy AndroidX/Firebase dependencies.
- `Application/build.gradle` uses Android Gradle Plugin `3.4.1`, Google Services plugin `4.2.0`, and `jcenter()`.
- These are source observations, not evidence that the project currently compiles or that every advertised feature works.
- The inspected project is native Android. An Android APK is not an iPhone parent app. Reusing the existing parent interface depends on an Android parent device; an iPhone-compatible interface is a separate scope decision.

## 1. Verify the personal Linux host

Before installing or changing system tools, inspect CPU architecture, available RAM, disk space, graphics and virtualization support. Current Android Studio documentation lists 16 GB RAM minimum for Studio plus Emulator on Linux and recommends 32 GB or more. Linux emulator acceleration uses KVM. Actual suitability remains unverified until the host is checked.

Suggested read-only preflight:

```bash
uname -m
lscpu
free -h
df -h "$HOME"
ls -l /dev/kvm 2>/dev/null || true
```

An existing `/dev/kvm` path alone does not prove that the current user can run an accelerated emulator. After SDK installation, also run the emulator's `-accel-check` and boot a test virtual device.

Install Android Studio and run the interactive emulator on the Linux host. Build dependencies may be isolated in a project-specific container or toolchain. Do not replace system-wide Java defaults merely to accommodate this legacy project.

## 2. Establish a reproducible build before feature development

1. Inspect the Gradle wrapper, manifests, services, dependency repositories, Firebase configuration and repository history for security-sensitive material.
2. Select and pin a mutually compatible JDK, Gradle, Android Gradle Plugin and SDK combination. Decide explicitly between reproducing the legacy build in isolation and a minimal build-system migration; do not treat an SDK-number bump as compatibility validation.
3. Use synthetic accounts/data and an isolated test backend. Do not run the inherited app against an upstream author's backend or put real family information into development builds.
4. Update dependencies and add explicit local/test backend configuration. Firebase Local Emulator Suite is a candidate for Auth/Database/Storage testing after client compatibility is checked.
5. Add a GitHub-hosted CI workflow to build a debug APK, run unit tests and lint, and retain sanitized test reports. Add instrumented emulator tests only on a verified compatible runner.

Gate: a clean checkout builds under a documented pinned toolchain, with exact commands and test results recorded. No successful build is claimed by this planning change.

## 3. Test both roles, then the intended physical device

Create separate parent-role and child-role Android virtual devices with synthetic accounts. Test pairing, authorization boundaries, supported controls, permission-denial paths, reconnection, restarts and account/device removal. Simulated contacts, location and communication events are fixtures, not evidence of real-world collection reliability.

Treat physical-device acceptance as a separate release gate: fresh installation, permission onboarding, screen-off behavior, reboot, battery restrictions, offline recovery and update behavior. Record the actual target phone model and Android version before finalizing this matrix. A passing emulator build does not certify the intended phone.

Do not claim third-party messenger access or all advertised upstream monitoring features work without feature-specific evidence. Scope the product to transparent, parent-authorized controls using supported Android permissions. No hidden installation, permission bypass, or disabling platform security checks.

## 4. Keep costs, public artifacts and credentials controlled

- Prefer standard GitHub-hosted runners; this repository is public. GitHub currently documents free standard runner usage for public repositories. Artifact/cache retention and any other services still need explicit limits.
- Do not attach a privileged personal-machine self-hosted runner to untrusted public pull-request code.
- Codespaces has separate compute/storage quotas and billing. Check the personal account's remaining quota and set a blocking budget before provisioning paid capacity. Stopping a codespace stops compute, not retained storage usage.
- Never commit release signing keys, passwords, service-account credentials, access tokens, real device records or personal test logs. Keep signing backups private and use protected secrets for any later release workflow.
- Public source code does not imply public backend data. Require tested per-family authorization, limited retention and deletion controls before real use.

## First milestone and explicit non-actions

The first implementation milestone is a reproducible debug build and a synthetic parent/child test workflow, not a production deployment or a promise of complete current-Android feature support.

This planning change does not create a cloud VM, Codespace, Firebase project, dashboard, APK, signing key or CI workflow. It does not install software on a local computer or alter application behavior. Hosting and release distribution follow only after the build, privacy review and device tests.

## Sources checked

Repository sources: `README.md`, `Application/build.gradle`, `Application/app/build.gradle` on the inspected default branch.

- Android Studio installation and Linux requirements: https://developer.android.com/studio/install
- Android Emulator acceleration and KVM: https://developer.android.com/studio/run/emulator-acceleration
- Android Emulator capabilities: https://developer.android.com/studio/run/emulator
- GitHub Codespaces architecture: https://docs.github.com/en/codespaces/about-codespaces/what-are-codespaces
- GitHub Codespaces billing: https://docs.github.com/en/billing/concepts/product-billing/github-codespaces
- GitHub-hosted runners: https://docs.github.com/en/actions/reference/runners/github-hosted-runners
- Firebase Local Emulator Suite: https://firebase.google.com/docs/emulator-suite
