---
description: Avoid hydration mismatches in React client components under Next.js static export by reading browser-only state at user-action time instead of useEffect+setState.
globs: src/**/*.tsx
alwaysApply: false
---

Reading browser-only state:
- Read `sessionStorage`/`localStorage`/`window` at the user action.
- Never `useEffect(() => setState(read()), [])` to display it.
- Merge the read value into the action payload.
- ESLint `react-hooks/set-state-in-effect` forbids it.
- SSR renders empty, client renders value → hydration mismatch.

Example:
- ❌ `useEffect(() => setAttr(getAttribution()), [])` then render hidden inputs.
- ✅ read `getAttribution()` inside the submit handler, spread into the payload.
