---
name: gsap-core
description: GSAP Core API — gsap.to(), from(), fromTo(), easing, stagger, transforms, autoAlpha, matchMedia
---

Skill gốc: `gsap-skills/skills/gsap-core/SKILL.md`

## Mô tả
Core API của GSAP: tween cơ bản, easing, stagger, transforms, `gsap.matchMedia()` cho responsive.

## Khi nào dùng
- User cần animation JavaScript
- Tween đơn giản (di chuyển, xoay, opacity)
- Cần responsive animation / prefers-reduced-motion

## Pattern chính
```js
gsap.to(".box", { x: 100, autoAlpha: 1, duration: 0.6, ease: "power2.inOut" });
gsap.fromTo(".el", { opacity: 0 }, { opacity: 1, duration: 1 });
gsap.set(".el", { x: 50 }); // set trạng thái ngay lập tức
```

## Lưu ý
- Dùng `autoAlpha` thay vì `opacity` + `visibility` riêng
- Dùng `x`/`y` thay vì `left`/`top` (GPU加速)
- `gsap.matchMedia()` để responsive animation
