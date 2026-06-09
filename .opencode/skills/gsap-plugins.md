---
name: gsap-plugins
description: GSAP Plugins — ScrollTrigger, Flip, Draggable, SplitText, MorphSVG, CustomEase, ScrollSmoother...
---

Skill gốc: `gsap-skills/skills/gsap-plugins/SKILL.md`

## Mô tả
Tất cả plugin GSAP đều **miễn phí** (kể từ Webflow acquisition). Gồm: ScrollToPlugin, ScrollSmoother, Flip, Draggable, Inertia, Observer, SplitText, ScrambleText, MotionPath, MorphSVG, DrawSVG, physics plugins, CustomEase, EasePack, GSDevTools.

## Pattern chính
```js
import { Flip } from "gsap/Flip";
gsap.registerPlugin(Flip);
const state = Flip.getState(".cards");
// thay đổi DOM...
Flip.from(state, { duration: 0.6, ease: "power2" });
```

## Lưu ý
- Cài từ `npm install gsap` — không cần auth token hay Club GSAP membership
- Luôn `gsap.registerPlugin(...)` trước khi dùng
