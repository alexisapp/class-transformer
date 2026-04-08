# Repository Context

## Purpose

- TypeScript library for decorator-based transformation between plain JavaScript objects and class instances (serialization/deserialization).
- Provides decorators (`@Expose`, `@Exclude`, `@Type`, `@Transform`) to control how properties are mapped during conversions.
- Core API: `plainToInstance`, `instanceToPlain`, `instanceToInstance` with options for groups, versioning, circular references, and implicit type conversion.
- Originally from [typestack/class-transformer](https://github.com/typestack/class-transformer), forked under Simployer. Published to NPM as `class-transformer`.

## Codebase Map

```
src/
  index.ts                        # Public API: top-level functions delegating to ClassTransformer
  ClassTransformer.ts             # Core class wrapping TransformOperationExecutor
  TransformOperationExecutor.ts   # Main transformation logic (~550 lines), recursive object traversal
  MetadataStorage.ts              # Singleton registry for decorator metadata (type, transform, expose, exclude)
  storage.ts                      # Exports defaultMetadataStorage singleton
  constants/
    default-options.constant.ts   # Default ClassTransformOptions values
  decorators/
    expose.decorator.ts           # @Expose - mark properties for inclusion
    exclude.decorator.ts          # @Exclude - mark properties for exclusion
    transform.decorator.ts        # @Transform - custom value transformation
    type.decorator.ts             # @Type - specify nested type for deserialization
    transform-instance-to-*.ts    # Method-level transformation decorators
  enums/
    transformation-type.enum.ts   # CLASS_TO_PLAIN, PLAIN_TO_CLASS, CLASS_TO_CLASS
  interfaces/                     # TypeScript interfaces for options, metadata, decorator configs
  utils/                          # getGlobal, isPromise helpers
test/
  functional/                     # 15 spec files covering all major features
    basic-functionality.spec.ts
    custom-transform.spec.ts
    circular-reference-problem.spec.ts
    implicit-type-declarations.spec.ts
    ...
sample/                           # Usage examples
docs/                             # Documentation pages
```

## Standard Commands

| Command                  | Description                                      |
| ------------------------ | ------------------------------------------------ |
| `npm run build`          | Build CJS output (alias for `build:cjs`)         |
| `npm run build:cjs`      | Compile to CommonJS (`tsconfig.prod.cjs.json`)   |
| `npm run build:esm5`     | Compile to ESM5 (`tsconfig.prod.esm5.json`)      |
| `npm run build:es2015`   | Compile to ES2015 (`tsconfig.prod.esm2015.json`) |
| `npm run build:umd`      | Bundle UMD via Rollup                            |
| `npm run build:types`    | Emit type declarations only                      |
| `npm run test`           | Run Jest with coverage and verbose output        |
| `npm run test:ci`        | Run Jest single-threaded, no cache, with coverage|
| `npm run lint:check`     | ESLint check on `src/` (zero warnings allowed)   |
| `npm run lint:fix`       | ESLint auto-fix on `src/`                        |
| `npm run prettier:check` | Prettier format check on `*.ts` and `*.md`       |
| `npm run prettier:fix`   | Prettier auto-fix on `*.ts` and `*.md`           |

## Implementation Rules

1. All source code lives in `src/`. Tests live in `test/functional/`. Do not mix test files into `src/` (except `*.spect.ts` utility tests in `utils/`).
2. The public API surface is defined in `src/index.ts`. New features must be exported there.
3. Decorator metadata is stored in the `defaultMetadataStorage` singleton (`src/storage.ts`). All decorators register metadata through `MetadataStorage` methods.
4. `TransformOperationExecutor` is the single transformation engine. All conversion paths (plain-to-class, class-to-plain, class-to-class) go through its `transform` method.
5. TypeScript strict mode is enabled in the main `tsconfig.json`. The test tsconfig (`tsconfig.spec.json`) relaxes strict checks (`strict: false`, `noImplicitAny: false`).
6. Prettier config: 120 char line width, 2-space indent, single quotes, trailing commas (ES5), no tabs.
7. ESLint uses `@typescript-eslint/recommended` with type-checking and `plugin:jest/recommended`. `@typescript-eslint/no-explicit-any` is allowed (off). `member-ordering` is enforced.
8. Deprecated functions (`classToPlain`, `plainToClass`, `serialize`, `deserialize`, `deserializeArray`, `*FromExist`) remain for backward compatibility but should not be used in new code.
9. The library has zero runtime dependencies. `reflect-metadata` is a devDependency used only in tests.
10. Build outputs four formats: CJS, ESM5, ES2015, and UMD bundles. The `build/` directory is the publish root (see CD workflow).
11. Husky pre-commit hook runs `lint-staged` which applies Prettier to staged `.ts` and `.md` files.

## Definition of Done

- All existing tests pass: `npm run test` exits cleanly with no failures.
- Lint passes with zero warnings: `npm run lint:check` and `npm run prettier:check` both succeed.
- New functionality includes corresponding test(s) in `test/functional/`.
- Build succeeds for all targets: `build:cjs`, `build:esm5`, `build:es2015`, `build:umd`, and `build:types`.
- No new runtime dependencies introduced without explicit justification.
- Public API changes are exported from `src/index.ts`.

## Guardrails

- **Zero warnings policy**: ESLint is configured with `--max-warnings 0`. Any warning fails the check.
- **CI runs on push and PR**: Linters, tests (Node 10/12/14 matrix), and build are validated in GitHub Actions (`continuous-integration-workflow.yml`).
- **CD publishes on GitHub release**: The `continuous-deployment-workflow.yml` runs all checks, builds all formats, and publishes the `build/` directory to NPM.
- **Coverage**: Codecov is configured via `codecov.yml`; coverage is collected from `src/**/*.ts` excluding index and interface files.
- **Pre-commit hooks**: Husky + lint-staged enforce Prettier formatting before commits land.
- **No side effects**: `package.json` declares `"sideEffects": false` for tree-shaking support.

## References

- Upstream repository: https://github.com/typestack/class-transformer
- CI workflow: `.github/workflows/continuous-integration-workflow.yml`
- CD workflow: `.github/workflows/continuous-deployment-workflow.yml`
- ESLint config: `.eslintrc.yml`
- Prettier config: `.prettierrc.yml`
- TypeScript configs: `tsconfig.json` (base), `tsconfig.spec.json` (tests), `tsconfig.prod.*.json` (build variants)
- Documentation: `docs/` directory and `README.md`
