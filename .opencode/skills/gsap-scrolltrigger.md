---
name: gsap-scrolltrigger
description: GSAP ScrollTrigger — scroll-linked animation, pinning, scrub, refresh & cleanup
---

Skill gốc: `gsap-skills/skills/gsap-scrolltrigger/SKILL.md`

## Mô tả
ScrollTrigger plugin: animation gắn với scroll, pin section, scrub (animation theo scroll), trigger refresh.

## Pattern chính
```js
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

gsap.to(".panel", {
  scrollTrigger: { trigger: ".section", start: "top center", end: "bottom center", scrub: true },
  x: 100, rotation: 5
});

// Sau DOM/layout change: ScrollTrigger.refresh();
```

## Lưu ý
- Luôn gọi `ScrollTrigger.refresh()` sau khi DOM thay đổi
- Cleanup trong React/Vue: `ctx.add(() => ScrollTrigger.getAll().forEach(st => st.kill()));`
