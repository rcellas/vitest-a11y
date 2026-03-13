# Changelog

All notable changes to the Vitest A11y Testing Skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-03-14

### Added

#### Core Documentation
- Complete SKILL.md with comprehensive accessibility testing guidance
- README.md with quick start guide and installation instructions
- CONTRIBUTING.md with contribution guidelines and standards
- LICENSE file (MIT)
- This CHANGELOG.md

#### Reference Documentation
- **APG Patterns Reference** (`reference/apg-patterns.md`)
  - Complete documentation for all 17 W3C APG patterns
  - Test contract tables for each pattern
  - Browser Mode and jsdom examples
  - Keyboard interaction specifications

- **WCAG 2.2 Reference** (`reference/wcag-2.2.md`)
  - All WCAG 2.2 success criteria
  - 9 new WCAG 2.2 criteria with examples
  - Level A, AA, and AAA coverage
  - Code examples for each criterion

- **Common Mistakes Reference** (`reference/common-mistakes.md`)
  - 15 common accessibility testing mistakes
  - Solutions and correct approaches
  - Before/after code examples

#### Examples
- **React Examples**
  - Dialog/Modal pattern with Browser Mode
  - Tabs pattern with full keyboard navigation
  
- **Angular Examples**
  - Dialog component with jsdom mode
  - vitest-axe integration examples
  
- **Vue Examples**
  - Accordion pattern with Browser Mode
  - Multiple panel expansion testing

#### Features
- **Mandatory Autofix Behavior**
  - Automatic detection and fixing of common issues
  - DI provider patching for Angular
  - Path alias resolution
  - Test environment configuration

- **Angular v20+ Support**
  - Automatic `ng test` usage detection
  - Template and style URL handling
  - JIT compilation support

- **Framework Support**
  - React with vitest-browser-react
  - Angular with @testing-library/angular
  - Vue with vitest-browser-vue
  - Svelte with vitest-browser-svelte

- **Testing Modes**
  - Browser Mode for interactive testing
  - jsdom mode for structural testing
  - Clear guidance on when to use each

#### Testing Patterns
- Gherkin format (Given-When-Then) for all tests
- WCAG and APG citation requirements
- Semantic query prioritization
- Focus management testing
- Keyboard navigation testing
- ARIA attribute verification
- axe-core integration with WCAG AA tags

### Documentation Standards
- Complete code examples with imports
- Both correct (✅) and incorrect (❌) patterns
- WCAG/APG citations in every test
- Runnable, copy-paste ready examples
- TypeScript throughout

### Testing Coverage
- WCAG 2.2 Level AA compliance
- All 17 APG patterns documented
- Focus management (WCAG 2.4.7)
- Keyboard accessibility (WCAG 2.1.1)
- Name, Role, Value (WCAG 4.1.2)
- Error identification (WCAG 3.3.1)
- No keyboard trap (WCAG 2.1.2)

## [Unreleased]

### Planned
- Additional APG pattern examples
  - Combobox pattern (React)
  - Menu Button pattern (Angular)
  - Slider pattern (Vue)
  - Disclosure pattern
  - Radio Group pattern
  
- More framework examples
  - Svelte component examples
  - Solid.js examples
  
- Advanced topics
  - Form validation patterns
  - Live region testing
  - Color contrast testing
  - Focus-visible testing
  - Custom ARIA widgets
  
- Tooling
  - ESLint plugin for test quality
  - VS Code snippets
  - GitHub Actions workflow examples

### Under Consideration
- Video tutorials
- Interactive playground
- Migration guides from other testing libraries
- Performance testing integration
- Visual regression testing

---

## Release Notes Format

### Added
New features, documentation, or examples

### Changed
Changes to existing functionality or documentation

### Deprecated
Features or patterns that will be removed in future versions

### Removed
Features or patterns that have been removed

### Fixed
Bug fixes or documentation corrections

### Security
Security-related changes or fixes

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines on proposing changes and additions to this skill.

## Versioning

- **Major version** (X.0.0) - Breaking changes to skill structure or major pattern updates
- **Minor version** (0.X.0) - New patterns, examples, or significant documentation additions
- **Patch version** (0.0.X) - Bug fixes, typos, clarifications, minor improvements

[1.0.0]: https://github.com/yourusername/vitest-a11y-skill/releases/tag/v1.0.0
[Unreleased]: https://github.com/yourusername/vitest-a11y-skill/compare/v1.0.0...HEAD
