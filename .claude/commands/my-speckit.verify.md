---
description: Verify that the implementation code matches the specification (spec.md) and plan (plan.md)
---

## Input

```text
$ARGUMENTS
```

The argument should specify spec.md.
If not specified, search for spec.md in the current directory. If still not found, stop execution.
Never work without spec.md.

## Goal

Verify that the actual implementation code matches the requirements defined in `spec.md` and the architecture/design defined in `plan.md`. This command identifies gaps, mismatches, and missing implementations before deployment or release.

## Operating Constraints

**STRICTLY READ-ONLY**: Do **not** modify any files. Output a structured verification report. If issues are found, suggest remediation actions (user must explicitly approve before any fixes).

**Constitution Authority**: The project constitution (`documents/constitution.md`) is **non-negotiable**. Any implementation violating constitution principles is automatically CRITICAL.

**Scope**: This command verifies `spec.md` and `plan.md` against actual implementation code. `tasks.md` is intentionally excluded as it is a disposable execution artifact.

## Execution Steps

### 1. Locate Specification Files

- If argument is provided, use it as the path to spec.md
- If not provided, search for spec.md in the current directory
- Abort with error if spec.md is not found: "ERROR: spec.md not found. Please run /frontend.specify or /infrastructure.specify first."

Derive paths (all in the same directory as spec.md):
- SPEC_DIR = directory containing spec.md
- SPEC = SPEC_DIR/spec.md
- PLAN = SPEC_DIR/plan.md
- DATA_MODEL = SPEC_DIR/data-model.md (if exists)
- CONTRACTS_DIR = SPEC_DIR/contracts/ (if exists)

Abort if plan.md is missing: "ERROR: plan.md not found. Please run /frontend.plan or /infrastructure.plan first."

### 2. Infer Domain and Implementation Directory

Detect the domain based on SPEC_DIR path:
- If path contains `frontend/`: Domain = "frontend", Implementation root = SPEC_DIR
- If path contains `infrastructure/`: Domain = "infrastructure", Implementation root = SPEC_DIR
- Otherwise: Abort with "ERROR: Cannot infer domain from spec location. Supported domains: frontend, infrastructure"

### 3. Load Constitution and Design Principles

- Load `documents/constitution.md` (from repository root)
- Load `documents/design-principles.md` (from repository root)
- Extract MUST/SHOULD requirements for validation

### 4. Load Specification Artifacts

Load minimal necessary context from each artifact:

**From spec.md:**
- Overview/Context
- Functional Requirements
- Non-Functional Requirements
- User Stories
- Edge Cases (if present)
- Out of Scope (to avoid false positives)

**From plan.md:**
- Technology stack choices
- File structure/architecture
- Component responsibilities
- API specifications
- Data flow
- Security/performance considerations

**From data-model.md (if exists):**
- Entity definitions
- Relationships
- Schema specifications

**From contracts/ (if exists):**
- API endpoint definitions
- Request/response schemas
- Integration contracts

**From constitution:**
- Already loaded in step 3

### 5. Build Requirement Inventory

Create internal requirement mappings:

- **Functional requirements list**: Each requirement with stable key (slug from imperative phrase)
- **Technical constraints**: Stack choices, architectural patterns
- **File structure map**: Expected files/directories from plan.md
- **API endpoint map**: Expected routes, methods, schemas
- **Data entity map**: Expected models, tables, relationships
- **Non-functional requirements**: Performance, security, scalability requirements

### 6. Scan Implementation Code

**Domain-specific scanning strategy:**

**For Frontend (Next.js):**
- Source directories: `app/`, `src/`, `components/`, `lib/`
- Configuration: `next.config.js`, `package.json`, `tsconfig.json`
- API routes: `app/api/`, `pages/api/`
- Components: Server/Client component files
- Tests: `*.test.ts`, `*.spec.ts`, `__tests__/`

**For Infrastructure (Terraform):**
- Module files: `*.tf` (main.tf, variables.tf, outputs.tf)
- Configuration: `terraform.tf`, `provider.tf`
- Documentation: `README.md`
- Tests: `*.tftest.hcl`, test files
- Examples: `examples/`

Identify actual implementation files based on plan.md's project structure:

- **Source code directories**: Scan directories specified in plan.md (e.g., `src/`, `lib/`, `app/`)
- **Configuration files**: Check for required config files
- **Database/model files**: Verify entity implementations
- **API/route files**: Verify endpoint implementations
- **Test files**: Check test coverage (optional, note if missing)

Use `Glob` and `Grep` tools efficiently to search for:
- File existence checks
- Function/class definitions
- API route definitions
- Database schema/model definitions
- Configuration patterns

### 7. Verification Passes

Focus on high-signal findings. Limit to 50 findings total.

#### A. Functional Requirements Coverage

For each functional requirement in spec.md:
- Search implementation code for related functions/components
- Check if requirement is implemented or missing
- Flag if implementation exists but appears incomplete

**Detection method:**
- Extract key terms from requirement (nouns, verbs)
- Search codebase for matching function names, comments, or file names
- Verify logic matches expected behavior (manual inspection of key files)

#### B. File Structure Alignment

Compare plan.md's expected file structure with actual files:
- Check if expected directories exist
- Check if expected files exist
- Flag unexpected files that may indicate scope creep or technical debt

#### C. Technology Stack Compliance

Verify that chosen tech stack from plan.md is actually used:
- Check package.json/requirements.txt/go.mod for declared dependencies
- Search for import statements matching planned libraries
- Flag if alternative libraries are used without plan updates

#### D. API Implementation Verification

If plan.md or contracts/ define API specifications:
- Check if endpoints are implemented
- Verify HTTP methods match specification
- Check request/response schemas (basic structure check)
- Flag missing or mismatched endpoints

#### E. Data Model Implementation

If data-model.md exists:
- Check if entities are defined in code (classes, models, schemas)
- Verify relationships are implemented (foreign keys, references)
- Check field types match specification
- Flag missing entities or fields

#### F. Non-Functional Requirements

For each non-functional requirement:
- **Performance**: Check for pagination, caching, indexing patterns
- **Security**: Check for authentication, authorization, input validation
- **Scalability**: Check for async patterns, connection pooling
- **Logging/Monitoring**: Check for logging statements, error tracking
- Flag if required patterns are missing

#### G. Constitution Compliance

If constitution exists:
- Verify implementation follows mandated principles
- Check for security/quality gates
- Flag CRITICAL violations

### 8. Severity Assignment

Use this heuristic:

- **CRITICAL**: Core functional requirement not implemented, constitution MUST violation, security requirement missing
- **HIGH**: API endpoint missing, data entity missing, non-functional requirement not addressed
- **MEDIUM**: File structure mismatch, incomplete implementation, tech stack deviation
- **LOW**: Missing optional feature, minor documentation gap

### 9. Produce Verification Report

Output a Markdown report (no file writes) with the following structure:

## Implementation Verification Report

**Summary:**
- Total Functional Requirements: X
- Implemented: Y
- Missing: Z
- Implementation Coverage: Y/X (XX%)

### Verification Results

| ID | Category | Severity | Requirement | Implementation Status | Location | Recommendation |
|----|----------|----------|-------------|----------------------|----------|----------------|
| F1 | Functional | HIGH | User authentication | MISSING | Expected in src/auth/ | Implement auth module per plan.md |
| F2 | Functional | MEDIUM | User profile edit | PARTIAL | src/user/profile.ts:45 | Complete validation logic |
| A1 | API | HIGH | POST /api/users | MISSING | Expected in app/api/users/ | Create endpoint per contracts/users.yaml |
| D1 | Data Model | CRITICAL | User entity | MISSING | Expected in src/models/ | Create User model with fields from data-model.md |

### Missing Files

| Expected File (from plan.md) | Status | Severity |
|------------------------------|--------|----------|
| src/auth/login.ts | MISSING | HIGH |
| src/db/migrations/001_init.sql | MISSING | MEDIUM |

### Tech Stack Deviations

| Planned Technology | Actual Usage | Notes |
|-------------------|-------------|-------|
| PostgreSQL | SQLite found | package.json shows sqlite3 instead of pg |

### Non-Functional Requirements Status

| Requirement | Status | Evidence | Recommendation |
|-------------|--------|----------|----------------|
| Input validation | PARTIAL | Found in 3/7 endpoints | Add validation to remaining endpoints |
| Authentication | MISSING | No auth middleware found | Implement per spec.md security requirements |

### Constitution Violations (if any)

| Principle | Violation | Location | Severity |
|-----------|-----------|----------|----------|

### Metrics

- Total Functional Requirements: X
- Implemented Requirements: Y (XX%)
- Missing Requirements: Z
- Total Files Expected: A
- Files Present: B (XX%)
- API Endpoints Expected: C
- Endpoints Implemented: D (XX%)
- Critical Issues: N
- High Issues: M

### 10. Provide Next Actions

At end of report, output a concise Next Actions block:

- If CRITICAL issues exist: **BLOCK deployment** - resolve before proceeding
- If HIGH issues exist: Recommend implementing missing core features
- If only MEDIUM/LOW: Note issues for future iterations

Provide explicit remediation suggestions:
- "Implement missing User entity in src/models/user.ts per data-model.md"
- "Create POST /api/users endpoint per contracts/users.yaml"
- "Add input validation to existing endpoints per security requirements"

### 11. Post to GitHub Issue

Report verification completion to the related GitHub Issue:
- Summary of verification results
- Total issues found by severity
- Link to the spec.md and plan.md
- Recommendation (BLOCK/PROCEED)

### 12. Offer Detailed Inspection

Ask the user: "Would you like me to inspect specific requirements or files in detail?" (Do NOT perform deep inspection automatically.)

## Operating Principles

### Verification Guidelines

- **NEVER modify files** (this is read-only verification)
- **NEVER hallucinate implementations** (if file not found, report as MISSING)
- **Use code search efficiently** (Glob for files, Grep for patterns)
- **Prioritize critical gaps** (core features, security, data integrity)
- **Report evidence** (cite file paths and line numbers when found)
- **Handle absence gracefully** (missing optional features are LOW severity)

### Search Strategy

- **File existence**: Use `Glob` with patterns from plan.md
- **Implementation patterns**: Use `Grep` for function names, class definitions, API routes
- **Dependency verification**: Read package.json, requirements.txt, go.mod
- **Progressive disclosure**: Don't read entire codebase, target files based on plan.md

### False Positive Mitigation

- Check "Out of Scope" section in spec.md to avoid flagging intentionally excluded features
- Consider implementation patterns may differ from plan.md naming (use fuzzy matching)
- Don't flag missing items if plan.md explicitly marks them as "future work" or "phase 2"

## Notes

- Strictly comply with CLAUDE.md and documents/design-principles.md
- For frontend verification, also reference documents/frontend/coding-standard.md if exists
- For infrastructure verification, also reference infrastructure-specific standards if exist
- This is a read-only analysis command - never modify implementation files
- Use Glob for file discovery, Grep for pattern search (avoid bash find/grep)
- Report findings with file paths and line numbers for easy navigation
