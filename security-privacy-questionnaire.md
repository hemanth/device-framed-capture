# W3C TAG Security and Privacy Self-Review Questionnaire

> Reference: [https://www.w3.org/TR/security-privacy-questionnaire/](https://www.w3.org/TR/security-privacy-questionnaire/)

### 2.1 What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?
**None.** `CaptureController.prototype.setDeviceFrame()` is a write-only presentation/compositing control on an already-authorized capture stream. It accepts caller-provided synthetic geometry (`formFactor`, `cornerRadius`, `cutouts`, `hinge`, `stageBackground`) and exposes zero host hardware information.

### 2.2 Do features in your specification expose the minimum amount of information necessary to enable their intended uses?
**Yes.** No new information is exposed to scripts.

### 2.3 How do the features in your specification deal with personal information, personally-identifiable information (PII), or information derived from them?
The feature does not access or introduce any PII. Frame pixels originate exclusively from a `getViewportMedia()` / `getDisplayMedia()` session explicitly approved by the user (and further restricted if `RestrictionTarget` / `CropTarget` is active).

### 2.4 How do the features in your specification deal with sensitive information?
All existing Screen Capture / Viewport Capture / Element Capture redaction and cross-origin pixel isolation rules apply *before* `setDeviceFrame()` composites corner clipping, cutouts, and bezels onto the captured surface.

### 2.5 Do the features in your specification introduce new state for an origin that persists across browsing sessions?
**No.** Configuration lives solely in memory for the lifetime of the `CaptureController` instance.

### 2.6 Do the features in your specification expose information about the underlying platform to origins?
**No.** There is no passive or active fingerprinting surface: the browser renders vendor-neutral procedural vector geometry from caller-supplied parameters and never queries the host OS/display hardware for physical notch or bezel shapes.

### 2.7 Does this specification allow an origin to send data to the underlying platform?
Only bounded geometric primitives (unsigned integers, enum strings, and a CSS color string parsed in a safe context) sent to the browser's capture compositor.

### 2.8 Do features in this specification allow an origin access to sensors on a user's device?
**No.**

### 2.9 What data do the features in this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.
Only the resulting video frames on the `MediaStreamTrack` that the origin already requested and received user consent to capture.

### 2.10 Do features in this specification enable new script execution/loading mechanisms?
**No.**

### 2.11 Do features in this specification allow an origin to access other devices?
**No.**

### 2.12 Do features in this specification allow an origin some measure of control over a user agent's native UI?
**No.** Device frames are composited strictly inside the captured `MediaStreamTrack` / BiDi screencast output, never altering the browser's trusted top-level chrome or permission indicators.

### 2.13 What temporary identifiers do the features in this specification create or expose to the web?
**None.**

### 2.14 How does this specification distinguish between behavior in first-party and third-party contexts?
Inherits `Permissions-Policy: display-capture` and Secure Context (`[SecureContext]`) requirements from Screen Capture / Viewport Capture.

### 2.15 How do the features in this specification work in the context of a browser's Private Browsing or Incognito mode?
Identically to `getViewportMedia()` / `getDisplayMedia()`, retaining zero persistent state.
