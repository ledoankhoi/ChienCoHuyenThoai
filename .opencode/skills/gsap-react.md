---
name: gsap-react
description: GSAP + React — useGSAP hook, gsap.context(), cleanup, SSR
---

Skill gốc: `gsap-skills/skills/gsap-react/SKILL.md`

## Mô tả
Tích hợp GSAP với React: `useGSAP()` hook, `gsap.context()` cho scoped selector, cleanup khi unmount, SSR.

## Pattern chính
```js
import { useGSAP } from "@gsap/react";
gsap.registerPlugin(useGSAP);

useGSAP(() => {
  gsap.to(ref.current, { x: 100, duration: 1 });
}, { scope: containerRef });

// Hoặc dùng useEffect:
useEffect(() => {
  const ctx = gsap.context(() => { gsap.to(".el", { x: 100 }); }, containerRef);
  return () => ctx.revert();
}, []);
```

## Lưu ý
- Không dùng selector string không có scope — dùng refs hoặc scope
- Luôn cleanup: `ctx.revert()` trong useEffect return
- Dùng `useGSAP` thay vì `useEffect` cho GSAP-specific code
