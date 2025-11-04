# Technical Context

**Language/Version**: Typescript
**Primary Dependencies**: NextJS 16
**Storage**: S3
**Testing**: Vitest
**Target Platform**: S3 + Cloud Front (SSG)
**Project Type**: web

## Remarks

### The next lint command has been removed

The next lint command has been removed. Use Biome or ESLint directly. next build no longer runs linting.

ref: https://nextjs.org/docs/app/guides/upgrading/version-16

```bash
npx @next/codemod@canary next-lint-to-eslint-cli .
```
