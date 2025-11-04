# Frontend Test Design Policy

## Summary

Defines test design policies specific to the frontend.  
For general test principles, refer to `documents/test-principles.md`.

## Prompt

| Type   | Target                                        | Tool                               | Location                         |
|--------|-----------------------------------------------|------------------------------------|----------------------------------|
| Unit   | Functions, components, hooks                  | Vitest + React Testing Library | `__tests__/` or `*.test.{ts,tsx}` |
| Integration | Interaction between multiple components, API communication | Vitest | Same as above |
| E2E    | User-level screen transitions and interactions | Playwright                          | `tests/e2e/`                     |

Implementation rules:

- E2E browser: Chromium only.
- E2E viewport: Support mobile (375x667) and tablet (768x1024)

## Criteria

- Coverage: 80%+ for business logic, 80%+ for UI components
