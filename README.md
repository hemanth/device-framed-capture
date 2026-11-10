# device-framed-capture

Composite physical screen masks (rounded corners, camera cutouts, foldable hinges) and vendor-neutral device frames directly into `getViewportMedia()` / `getDisplayMedia()` streams and WebDriver BiDi screencasts.

## Quick start (Web API)

```js
const controller = new CaptureController();
const stream = await navigator.mediaDevices.getViewportMedia({
  video: { frameRate: 60 },
  audio: true,
  controller,
});

const previewIframe = document.querySelector('#mobile-sandbox');
const target = await RestrictionTarget.fromElement(previewIframe);
await stream.getVideoTracks()[0].restrictTo(target);

await controller.setDeviceFrame({
  mode: 'device',
  formFactor: 'phone',
  cornerRadius: 39,
  cutouts: [
    { shape: 'pill', x: 134, y: 11, width: 125, height: 37, borderRadius: 19 },
  ],
  rotationBehavior: 'contain',
});
```

`setDeviceFrame()` composites corner clipping, display cutouts, foldable hinges, and a vendor-neutral chassis on the browser capture pipeline at full frame rate without main-thread `<canvas>` copies or dropped audio sync. Pass `null` to return to raw rectangular capture.

## Occlusion-only mode

```js
await controller.setDeviceFrame({
  mode: 'occlusions',
  cornerRadius: 39,
  cutouts: [{ shape: 'circle', x: 188, y: 0, width: 37, height: 58 }],
  hinge: { orientation: 'vertical', offset: 372, maskLength: 0 },
});
```

`'occlusions'` keeps the exact viewport dimensions while masking rounded corners, punch-hole/pill cutouts, and foldable hinges so visual tests and in-app bug recorders capture what is actually visible on hardware.

## WebIDL

```webidl
enum CaptureFramingMode {
  "none",
  "occlusions",
  "device"
};

enum DeviceFormFactor {
  "phone",
  "foldable",
  "feature-phone",
  "tablet",
  "notebook",
  "smart-display",
  "watch",
  "desktop"
};

enum DisplayCutoutShape {
  "pill",
  "circle",
  "notch",
  "rectangle"
};

enum CaptureRotationBehavior {
  "resize",
  "contain"
};

dictionary DisplayCutoutInit {
  required DisplayCutoutShape shape;
  required unsigned long x;
  required unsigned long y;
  required unsigned long width;
  required unsigned long height;
  unsigned long borderRadius = 0;
};

dictionary DisplayHingeInit {
  required DOMString orientation; // "vertical" | "horizontal"
  required unsigned long offset;
  required unsigned long maskLength;
};

dictionary DeviceFrameConfiguration {
  CaptureFramingMode mode = "device";
  DeviceFormFactor formFactor = "phone";
  (unsigned long or sequence<unsigned long>) cornerRadius = 0;
  sequence<DisplayCutoutInit> cutouts = [];
  DisplayHingeInit? hinge = null;
  CaptureRotationBehavior rotationBehavior = "contain";
  DOMString stageBackground = "#f1f3f4";
};

partial interface CaptureController {
  Promise<undefined> setDeviceFrame(optional DeviceFrameConfiguration? configuration = null);
};
```

## WebDriver BiDi hook

```json
{
  "id": 1,
  "method": "emulation.setDisplayFeaturesOverride",
  "params": {
    "contexts": ["ctx-1"],
    "features": {
      "formFactor": "phone",
      "cornerRadius": 39,
      "cutouts": [
        { "shape": "pill", "x": 134, "y": 11, "width": 125, "height": 37, "borderRadius": 19 }
      ]
    }
  }
}
```

```json
{
  "id": 2,
  "method": "browsingContext.startScreencast",
  "params": {
    "context": "ctx-1",
    "framing": "device",
    "video": { "frameRate": 30 }
  }
}
```

`emulation.setDisplayFeaturesOverride` defines physical screen masks alongside `emulation.setSafeAreaInsetsOverride`, and `framing: "none" | "occlusions" | "device"` exposes the same capture pipeline to DevTools, Playwright/Puppeteer/Selenium, and AI browser agents.

## Proposal documents

- [WICG/proposals#318](https://github.com/WICG/proposals/issues/318) - WICG Web Platform API incubation thread
- [w3c/webdriver-bidi#1211](https://github.com/w3c/webdriver-bidi/issues/1211) - W3C WebDriver BiDi protocol proposal thread
- [EXPLAINER.md](./EXPLAINER.md) - Problem motivation, Web API design, WebDriver BiDi integration, and alternatives considered
- [spec.bs](./spec.bs) - Draft Bikeshed specification (WebIDL + CDDL + normative algorithms)
- [security-privacy-questionnaire.md](./security-privacy-questionnaire.md) - W3C TAG Security & Privacy Self-Review
- [Chromium CL 8779313](https://chromium-review.googlesource.com/c/devtools/devtools-frontend/+/8779313) - Working reference implementation in Chromium DevTools ([crbug.com/40521790](https://issues.chromium.org/issues/40521790))

## License

[W3C Software and Document Notice and License](./LICENSE) © [Hemanth.HM](https://h3manth.com)
