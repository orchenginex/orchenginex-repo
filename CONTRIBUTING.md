# Contributing to OrchEngineX

Thank you for your interest in contributing to OrchEngineX. This document provides guidelines for contributing to the project.

## Getting Started

1. Fork the repository
2. Clone your fork: `git clone https://github.com/your-username/orchenginex.git`
3. Install dependencies: `npm install`
4. Create a feature branch: `git checkout -b feature/your-feature`
5. Start development: `npm run dev`

## Code Style

- **TypeScript** is required for all source files
- Use **semantic Tailwind tokens** from the design system — never hardcode colors
- Follow existing component patterns in `src/components/`
- Keep components focused and composable

## Architecture Conventions

### Node Types

When adding new node types to the DWFG model:

1. Add the type to the `NodeType` union in `src/types/architecture.ts`
2. Add a catalog entry to `NODE_TYPE_CATALOG` with:
   - `type`, `label`, `shortLabel`, `icon`
   - `defaults` (latencyBaseline, retryPolicy, failureProbability, resourceCapacity)
   - `allowedOutgoing` and `allowedIncoming` edge types
   - `category` (edge, ingress, compute, data, messaging, mesh, egress)
3. Update the import validator in `src/utils/importArchitectureValidator.ts`
4. Update the graph layout zones in `ArchitectureGraphView.tsx`
5. Update the backend `computeStructuralAnalysis` in `supabase/functions/data-api/index.ts`

### Simulation Engines

When adding new mechanics engines:

1. Create the compute function in `src/components/mechanics/`
2. Register it in `computeMechanics.ts`
3. Add UI controls in `MechanicsSimulator.tsx`
4. Update the `deriveFromArchitecture.ts` mapping

### Fragility Score

The fragility score formula is shared between:
- `src/utils/graphAnalysis.ts` → `computeStructuralRisk()`
- `supabase/functions/data-api/index.ts` → `computeStructuralAnalysis()`

**These must always produce identical results.** Any change to the formula must be applied to both locations.

## Pull Request Process

1. Ensure your code compiles without errors: `npx tsc --noEmit`
2. Run tests: `npm test`
3. Update documentation if you've changed APIs, formulas, or added node types
4. Write a clear PR description explaining what changed and why
5. Reference any related issues

## Reporting Issues

### Bug Reports

Include:
- Steps to reproduce
- Expected vs. actual behavior
- Architecture JSON (if applicable) — use examples from `examples/`
- Browser and OS information
- Console errors (if any)

### Feature Requests

Include:
- Use case description
- How it relates to structural fragility modeling
- Proposed approach (if any)

## Areas for Contribution

- **New node types** — additional infrastructure component models
- **Simulation engines** — new mechanics or failure modes
- **Visualization** — graph layouts, export formats, chart types
- **CLI commands** — additional analysis or automation capabilities
- **Documentation** — tutorials, examples, methodology explanations
- **Research** — new failure case studies with reproducible simulations
- **Performance** — optimization of graph analysis algorithms

## Code of Conduct

Be respectful, constructive, and focused on engineering quality. OrchEngineX is a technical project — contributions are evaluated on technical merit.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
