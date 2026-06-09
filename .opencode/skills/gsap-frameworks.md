---
name: gsap-frameworks
description: GSAP + Vue/Svelte — lifecycle, scoping selectors, cleanup on unmount
---

Skill gốc: `gsap-skills/skills/gsap-frameworks/SKILL.md`

## Mô tả
Tích hợp GSAP với Vue, Svelte, và các framework khác: lifecycle hooks, scoped selectors, cleanup.

## Pattern chính
```vue
<script setup>
import { onMounted, onUnmounted, ref } from 'vue';
const el = ref(null);
let ctx;
onMounted(() => { ctx = gsap.context(() => { gsap.to(el.value, { x: 100 }); }); });
onUnmounted(() => ctx?.revert());
</script>
```
