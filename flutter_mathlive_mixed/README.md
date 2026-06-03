# mathlive_studio

Mixed **plain text + inline LaTeX** `\(...\)` preview and **MathLive** math editor for Flutter.

- **Viewer** — `MathLiveMixedPreview` (WebView on mobile/desktop, iframe on web).
- **Editor** — `MathLiveEmbeddedEditor`, `MathLiveEditorPage`.
- **Utilities** — `textUsesMathLivePreview`, `normalizeInlineMath`, `parsePreviewParts`.
- **Fallback** — `InlineTexMixedText` / `buildSimpleLatexInline` (no WebView, no `flutter_math_fork`).

MathLive **0.101.2** is loaded from [jsDelivr](https://cdn.jsdelivr.net/npm/mathlive@0.101.2/) at runtime; `assets/mathlive/mathlive_editor.html` is bundled for the editor shell.

## Add to your app

### Path dependency (local / monorepo)

```yaml
dependencies:
  mathlive_studio:
    path: ../flutter_mathlive_mixed
```

### Git dependency

```yaml
dependencies:
  mathlive_studio:
    git:
      url: https://github.com/your-org/mathlive_studio.git
      ref: main
```

### pub.dev (after publish)

```yaml
dependencies:
  mathlive_studio: ^0.1.0
```

No extra asset declaration is required in the host app — assets ship inside the package.

## Quick start

```dart
import 'package:mathlive_studio/mathlive_studio.dart';
import 'package:mathlive_studio/utils.dart';

// Preview (chat bubble style)
MathLiveMixedPreview(
  previewText: r'Solve \(x^2 + 1 = 0\).',
  isDark: false,
  backgroundColor: Colors.transparent,
  baseStyle: TextStyle(fontSize: 15, color: Colors.black87),
  expandToContent: true,
  displayOnly: true,
)

// Gate: WebView vs lightweight Text
if (textUsesMathLivePreview(body)) {
  // use MathLiveMixedPreview
} else {
  InlineTexMixedText(source: body, style: textStyle);
}

// Embedded editor
MathLiveEmbeddedEditor(
  isDark: false,
  initialLatex: r'\frac{1}{2}',
  onLatexChanged: (latex) => print(latex),
)

// Full-screen editor
final latex = await MathLiveEditorPage.open(context, isDark: false);
```

## Platform notes

### Android

- Uses `webview_flutter` / `webview_flutter_android`.
- MathLive script/CSS load over **HTTPS** (jsDelivr). Internet permission is required (default in Flutter apps).
- If you load non-HTTPS URLs, configure cleartext in `AndroidManifest.xml` (not needed for the default CDN setup).

### iOS

- Uses `WKWebView` via `webview_flutter`.
- **App Transport Security (ATS)** allows HTTPS to jsDelivr by default. For CDN-only usage, no ATS exceptions are required.
- For offline-only MathLive (no CDN), bundle all JS/CSS locally and adjust `baseUrl` / HTML links (future enhancement).

### Web

- Preview: blob iframe + `postMessage` (`MathLiveMixedPreview` channel).
- Editor: MathLive injected into a `HtmlElementView` div (virtual keyboard works in the host document).
- Pin `webview_flutter_web: 0.2.3+2` if your Flutter SDK is older than 3.22.

## Example app

```bash
cd example
flutter pub get
flutter run -d chrome    # web
flutter run              # device / simulator
```

## Publishing to pub.dev

Follow the [Dart publishing guide](https://dart.dev/tools/pub/publishing).

### 1. Prepare the package

- [ ] Set real `homepage`, `repository`, and `issue_tracker` URLs in `pubspec.yaml`.
- [ ] Ensure `LICENSE` is present (MIT for this package; MathLive has its own license).
- [ ] Write a clear `CHANGELOG.md` for each release.
- [ ] Bump version in `pubspec.yaml` (semver: `0.1.0` → `0.1.1` patch, `0.2.0` minor).
- [ ] Run checks:

```bash
cd flutter_mathlive_mixed
dart pub publish --dry-run
dart analyze
flutter test
cd example && dart analyze
```

Fix anything reported by `--dry-run` (missing fields, invalid licenses, large files, etc.).

### 2. Create a pub.dev account

1. Sign in at [pub.dev](https://pub.dev/) with your Google account.
2. Run locally:

```bash
dart pub login
```

### 3. Verify publisher (recommended)

For a verified publisher (e.g. `publisher:yourcompany.com`):

1. Create organization on pub.dev → **Create publisher**.
2. Add the DNS `TXT` record pub.dev provides.
3. After verification, add to `pubspec.yaml`:

```yaml
publisher: yourcompany.com
```

### 4. Publish

```bash
cd flutter_mathlive_mixed
dart pub publish
```

Confirm when prompted. First upload may take a few minutes to appear on pub.dev.

### 5. Tag releases (optional)

```bash
git tag v0.1.0
git push origin v0.1.0
```

### 6. CI for pub.dev (optional)

- GitHub Action: run `dart analyze` and `flutter test` on PRs.
- On release tag, run `dart pub publish` with `PUB_CREDENTIALS` or `dart pub token add` in CI (see [pub.dev publishing automation](https://dart.dev/tools/pub/automation)).

### Common publish failures

| Issue | Fix |
|--------|-----|
| `publish_to: none` | Remove that line from **package** `pubspec.yaml` (keep it in `example/`). |
| Missing description | `description:` must be a non-empty sentence in `pubspec.yaml`. |
| Large assets | pub.dev limits package size; vendored `mathlive.min.js` in assets is OK if under limits. |
| Score / pana warnings | Run `dart pub publish --dry-run`; address `dartdoc` and platform warnings. |

## Rodha / host app migration

This package was generalized from an internal mentor editor stack. Replace:

| Rodha | Package |
|--------|---------|
| `MentorMathLiveMixedPreview` | `MathLiveMixedPreview` |
| `MentorMathLiveEmbeddedEditor` | `MathLiveEmbeddedEditor` |
| `MentorMathLiveEditorPage` | `MathLiveEditorPage` |
| `threadTextUsesMixedMathPreview` | `textUsesMathLivePreview` |
| `RodhaMathLive` / `RodhaMathLiveHost` | `MathLiveMixed` / `MathLiveMixedHost` |

## License

MIT for Dart/package code. MathLive is subject to its own license when loaded from CDN or bundled assets.
