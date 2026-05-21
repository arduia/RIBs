# Agent Context System - How Agents Use Research Documentation

## Overview

This document explains how AI agents will discover, load, and use the comprehensive research documentation for the RIBs project.

## The System Architecture

```
┌─────────────────────────────────────────────────────┐
│         Agent Receives Task                         │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
        ┌────────────────────────┐
        │ Reads CLAUDE.md         │ ◄─ Entry point
        │ (Agent Instructions)   │
        └────────────┬───────────┘
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
    ┌───────────┐         ┌──────────────┐
    │ Task Type │         │ Check Task   │
    │ Analysis  │         │ Guidance     │
    └─────┬─────┘         └──────────────┘
          │
          ▼
    ┌──────────────────────────┐
    │ Load Relevant Research   │
    │ Documentation            │
    └────────┬─────────────────┘
             │
    ┌────────┴──────────────────────┐
    │                               │
    ▼                               ▼
┌─────────────────┐      ┌──────────────────┐
│ Module-Specific │      │ Platform-Specific│
│ Docs (RIB_*)   │      │ Docs (iOS/Android)│
└─────────────────┘      └──────────────────┘
    │                               │
    └──────────────┬────────────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │ Review Code Examples │
        │ in Documentation     │
        └──────────┬───────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │ Implement Following  │
        │ Documented Patterns  │
        └──────────┬───────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │ Reference Tests in   │
        │ TESTING_GUIDE.md     │
        └──────────┬───────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │ Create Tests & Code  │
        │ Commit with Reference│
        └──────────────────────┘
```

## Discovery Mechanism

### 1. CLAUDE.md - Primary Entry Point
When an agent starts working on the RIBs codebase, it should:
1. **Read CLAUDE.md** first for project context
2. **Identify task type** using "Task-Specific Guidance"
3. **Find relevant documentation** listed for that task
4. **Follow pattern examples** in the documentation

### 2. .claude/settings.json - Automatic Configuration
The settings file tells Claude Code:
- Which files to load as context
- Where documentation is located
- Which patterns apply to different tasks
- Project-specific instructions

### 3. README.md - Quick Links
Prominent placement ensures agents see:
- Links to CLAUDE.md
- Links to research documentation
- Reference to docs/research/

## Workflow Examples

### Example 1: Adding a New Android RIB

**Agent receives task**: "Create a new detail screen RIB for displaying item details"

**Agent follows this workflow**:

```
1. Reads CLAUDE.md
   ↓
2. Finds "New RIB Implementation" in Task-Specific Guidance
   ↓
3. Loads recommended docs:
   - ARCHITECTURE_OVERVIEW.md
   - RIB_BASE.md
   - RIB_ANDROID.md
   - RIB_TEST.md
   ↓
4. Reviews code examples in each doc
   ↓
5. Understands:
   - Router/Interactor/Builder pattern
   - ViewRouter for Android
   - Lifecycle management
   - Testing approach
   ↓
6. Implements new RIB following patterns
   ↓
7. Creates tests using patterns from RIB_TEST.md
   ↓
8. Commits with clear message
```

### Example 2: Implementing Navigation Flow

**Agent receives task**: "Add navigation from list to detail with back stack management"

**Agent follows this workflow**:

```
1. Reads CLAUDE.md
   ↓
2. Finds "Navigation Features" in Task-Specific Guidance
   ↓
3. Loads recommended docs:
   - RIB_ROUTER_NAVIGATOR.md (primary)
   - RIB_WORKFLOW.md (for complex flows)
   ↓
4. Reviews navigation patterns:
   - Basic navigation
   - Back stack management
   - Deep linking
   ↓
5. Checks example code for router navigation
   ↓
6. Implements following best practices
   ↓
7. Creates integration tests
   ↓
8. Commits with reference to RIB_ROUTER_NAVIGATOR.md
```

### Example 3: Adding Compose Feature

**Agent receives task**: "Convert traditional ViewRouter to use Jetpack Compose"

**Agent follows this workflow**:

```
1. Reads CLAUDE.md
   ↓
2. Finds "Compose Integration" in Agent Instructions
   ↓
3. Loads recommended docs:
   - RIB_ANDROID_COMPOSE.md
   - RIB_ANDROID.md (for reference)
   ↓
4. Learns:
   - ComposePresenter pattern
   - State observable → Compose State conversion
   - Lifecycle binding in Compose
   ↓
5. Reviews integration examples
   ↓
6. Implements using Compose + RIBs patterns
   ↓
7. Tests using TESTING_GUIDE.md patterns
   ↓
8. Commits with reference to RIB_ANDROID_COMPOSE.md
```

## How Agents Load Context

### Method 1: Explicit Reading
Agents read the documentation files directly:
```
Read CLAUDE.md
↓
Find task-specific guidance
↓
Load referenced documentation files
↓
Use examples and patterns
```

### Method 2: Settings-Based Configuration
.claude/settings.json provides:
```json
{
  "contextFiles": {
    "files": ["CLAUDE.md", "docs/research/README.md", ...]
  },
  "templates": {
    "new_rib": "Read RIB_BASE.md, platform-specific doc, RIB_TEST.md...",
    "navigation": "Read RIB_ROUTER_NAVIGATOR.md and RIB_WORKFLOW.md..."
  }
}
```

### Method 3: Search and Discovery
Agents can search through:
- `docs/research/` directory
- Specific module documentation
- Code examples in docs
- Cross-references in README

## Key Documentation Paths

```
docs/research/
├── README.md (Index & Quick Reference)
├── ARCHITECTURE_OVERVIEW.md
├── TESTING_GUIDE.md
├── android/
│   ├── RIB_BASE.md ◄─ Foundational
│   ├── RIB_ANDROID.md ◄─ View integration
│   ├── RIB_ANDROID_COMPOSE.md
│   ├── RIB_WORKFLOW.md
│   ├── RIB_ROUTER_NAVIGATOR.md
│   └── RIB_TEST.md
└── ios/
    └── RIBs_iOS.md

CLAUDE.md ◄─ PRIMARY ENTRY POINT
├── Task-Specific Guidance
├── Pattern References
└── Quick Links to Docs

.claude/settings.json ◄─ CONFIGURATION
├── Context File Paths
├── Agent Templates
└── Project Metadata
```

## Documentation Referencing

### In Commit Messages
Agents should reference relevant documentation:
```
Implement detail RIB following rib-base pattern

Reference: docs/research/android/RIB_BASE.md
           docs/research/android/RIB_ANDROID.md
```

### In Code Comments
Use references to documentation for context:
```kotlin
// Router pattern from docs/research/android/RIB_BASE.md
class DetailRouter(
    interactor: DetailInteractor,
    component: DetailComponent
) : ViewRouter<DetailView, DetailComponent>(interactor, component)
```

### In PR Descriptions
When creating PRs, reference the research docs:
```
## Changes
Implemented new DetailRIB following RIBs architecture

## Research Reference
- docs/research/android/RIB_BASE.md
- docs/research/android/RIB_ANDROID.md
- docs/research/android/RIB_TEST.md

## Patterns Used
- Builder pattern for dependency injection
- ViewRouter for Android view management
- Unit tests with mocked dependencies
```

## Quality Assurance

### Verification Checklist for Agents

After completing a task, agents should verify:

- ✅ Read all relevant research documentation
- ✅ Followed established patterns from docs
- ✅ Used code examples from documentation as templates
- ✅ Implemented testing following TESTING_GUIDE.md
- ✅ Referenced documentation in commit/PR
- ✅ Code matches patterns described in research
- ✅ Architecture decisions explained by docs

## Continuous Improvement

### Updating Documentation
If agents discover:
- Missing patterns or examples
- Outdated information
- Better explanations needed
- New patterns to document

They should:
1. Note the gap in research documentation
2. Suggest improvements
3. Update relevant `.md` files
4. Commit updates to research docs

### Feedback Loop
```
Agent finds gap in documentation
↓
Updates/improves research doc
↓
New agents benefit from improved docs
↓
Better code quality and consistency
```

## Benefits of This System

### For Agents
- ✅ Clear guidance on project patterns
- ✅ Code examples to follow
- ✅ Consistent with team standards
- ✅ Faster implementation
- ✅ Better testing practices

### For Teams
- ✅ Consistent architecture across codebase
- ✅ Shared understanding of patterns
- ✅ Better code reviews
- ✅ Easier onboarding
- ✅ Documented design decisions

### For Projects
- ✅ Scalable architecture
- ✅ Maintainable codebase
- ✅ Clear technical standards
- ✅ Reduced technical debt
- ✅ Better collaboration

## Troubleshooting

### Agent Not Finding Documentation?
- Check that CLAUDE.md is in root directory
- Verify docs/research/ exists with all files
- Ensure .claude/settings.json is properly formatted
- Check README.md has links to research docs

### Agent Using Wrong Patterns?
- Verify CLAUDE.md task guidance is clear
- Check if documentation has examples for the task
- Ensure code examples are prominent in docs
- Review research docs for completeness

### Documentation Out of Date?
- Update relevant research `.md` files
- Add new pattern examples
- Update cross-references
- Commit with clear message about updates

## Next Steps

1. **For Current Work**: Ensure agents read CLAUDE.md before starting
2. **For New Tasks**: Check "Task-Specific Guidance" in CLAUDE.md
3. **For Improvements**: Update research docs with new patterns
4. **For Onboarding**: Have new agents read CLAUDE.md → ARCHITECTURE_OVERVIEW.md
5. **For Reviews**: Reference research docs in code review feedback

---

**System Version**: 1.0  
**Created**: May 21, 2026  
**Last Updated**: May 21, 2026  
**Part of**: Mental Alignment Initiative
