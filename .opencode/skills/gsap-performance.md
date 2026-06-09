---
name: gsap-performance
description: GSAP Performance — transforms, will-change, batching, ScrollTrigger optimization
---

Skill gốc: `gsap-skills/skills/gsap-performance/SKILL.md`

## Mô tả
Tối ưu performance cho GSAP animation: dùng transforms thay vì layout properties, will-change, batching, ScrollTrigger tips.

## Nguyên tắc
- Dùng `x`/`y`/`scale`/`rotation` thay vì `left`/`top`/`width`/`height` (GPU composite)
- Dùng `autoAlpha` thay vì `opacity` + `visibility`
- Tránh animate `box-shadow`, `filter` — nặng paint
- ScrollTrigger: `fastScrollEnd` cho mobile, `debounce` refresh
