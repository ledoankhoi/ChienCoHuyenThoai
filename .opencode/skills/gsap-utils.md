---
name: gsap-utils
description: GSAP Utilities — gsap.utils.clamp, mapRange, normalize, interpolate, random, snap, wrap, pipe
---

Skill gốc: `gsap-skills/skills/gsap-utils/SKILL.md`

## Mô tả
`gsap.utils.*` helpers: clamp, mapRange, normalize, interpolate, random, snap, toArray, wrap, pipe.

## Pattern chính
```js
const clamp = gsap.utils.clamp(0, 100); // clamp(150) → 100
const map = gsap.utils.mapRange(0, 100, 0, 1); // map(50) → 0.5
const rand = gsap.utils.random(["a","b","c"]); // random pick
const snap = gsap.utils.snap(10); // snap(14) → 10
```
