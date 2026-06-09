---
name: gsap-timeline
description: GSAP Timeline — sequencing, position parameter, labels, nesting, playback control
---

Skill gốc: `gsap-skills/skills/gsap-timeline/SKILL.md`

## Mô tả
`gsap.timeline()` để sắp xếp nhiều animation theo trình tự, với position parameter, labels, nesting.

## Pattern chính
```js
const tl = gsap.timeline({ defaults: { duration: 0.5, ease: "power2" } });
tl.to(".a", { x: 100 })
  .to(".b", { y: 50 }, "+=0.2")  // sau .a xong 0.2s
  .to(".c", { opacity: 0 }, "-=0.1"); // trước khi .b kết thúc 0.1s

tl.play(), tl.pause(), tl.reverse(), tl.seek(1), tl.progress(0.5);
tl.addLabel("mid", 1.5);
```

## Lưu ý
- Position parameter: `0.5` (tuyệt đối), `+=0.5` (relative), `-=0.5` (overlap), `"label"` (tại label)
- Timeline ưu tiên hơn chained `delay` — dễ kiểm soát hơn
