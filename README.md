# TimeSeriesKit

Explicit, tested operations for timestamped numeric series.

> Planning repository — documentation and Git initialization only. No application, package, solution, database, tests, CI workflow, or runnable demo has been implemented yet. Everything below describes the proposed implementation, unless explicitly marked as verified.

## Goals

Offer reusable time-series primitives with clear semantics for irregular timestamps, missing data, window boundaries, and numerical errors. Support exploratory telemetry analysis and simulation metrics.

## Target users

Developers building telemetry workbenches, recorded-session viewers, and simulation dashboards.

## MVP scope

- Immutable timestamped samples with unit metadata and quality flags.
- Range selection, bounded windows, min/max/mean, and online variance.
- Fixed-interval resampling with explicit aggregation/interpolation policy.
- Moving averages and chart downsampling that preserves extrema.
- Detection of gaps, duplicate timestamps, and out-of-order input according to a named policy.

## Stretch goals

- Rolling quantiles and robust outlier indicators.
- FFT utilities with explicit uniform-sampling preconditions.
- Incremental operators and vectorized implementations justified by benchmarks.

## Proposed tech stack

C# / .NET 10 dependency-light library, xUnit, property-based testing, BenchmarkDotNet. Prefer double for samples and documented numerical methods; unit conversion is an explicit boundary operation.

Select compatible stable dependency versions during milestone 1, record them in project files, and pin the SDK/toolchain after the compatibility spike. No packages or development workloads are installed by this documentation task.

## Architecture

Core defines samples, series views, and timestamp conventions. Validation classifies ordering and quality issues. Pure batch operators and bounded streaming accumulators share documented policies. Algorithms never depend on charts, databases, vehicle PIDs, or UI frameworks. Separate display downsampling from resampling used in calculations.

## Proposed folder structure

Only README.md and .gitignore exist initially, alongside Git metadata. The following tree is a plan; create directories when their implementation begins.

```text
TimeSeriesKit/
  README.md
  .gitignore
  TimeSeriesKit.slnx
  src/TimeSeriesKit/
  tests/TimeSeriesKit.UnitTests/
  tests/TimeSeriesKit.PropertyTests/
  benchmarks/TimeSeriesKit.Benchmarks/
  samples/TimeSeriesKit.ConsoleDemo/
  docs/algorithms/
```

## Data models and invariants

Sample(TimestampUtcTicks, Value, Quality); SeriesMetadata(Name, Unit); TimeRange(StartInclusive, EndExclusive); ResampleOptions(Interval, Origin, Aggregation, GapPolicy); WindowResult(Range, Count, MissingCount, Min, Max, Mean, Variance). Document UTC tick precision, timestamp overflow checks, and NaN/infinity handling separately from missing quality.

## Proposed API and contracts

Proposed API: Series.Validate(samples, orderingPolicy); Series.Resample(samples, options); Series.WindowStats(samples, window); Series.DownsampleMinMax(samples, maxPoints); OnlineStatistics.Add(value). Reject mixed units unless a consumer explicitly converts them. Output ordering and endpoint behavior are part of the contract.

Version persisted/exported formats from their first implementation. Validate at boundaries and return structured errors rather than leaking internal exceptions. The contracts above remain proposals until backed by code and contract tests.

## Implementation milestones

1. Specify timestamp, quality, duplicate, unit, and window semantics with hand-calculated examples.
2. Implement validation and numerically stable statistics.
3. Add resampling, moving windows, and extrema-preserving downsampling.
4. Benchmark representative irregular series and package with API/algorithm documentation.

Each milestone should end with a small demonstrable result, a written acceptance check, and tests for its important failure modes. Work through a vertical slice before expanding the stretch scope.

## Testing and CI plan

Compare statistics with hand-computed and independent reference values; test empty/singleton series, huge offsets with tiny variance, duplicates, gaps, overflow, endpoint boundaries, and non-finite values. Property tests check ordering, count conservation where applicable, and downsample bounds. CI builds/tests on Windows/Linux, packs locally, and tracks benchmarks separately from noisy per-commit pass/fail thresholds.

CI is planned, not configured. Once a solution exists, add .github/workflows/ci.yml with read-only default permissions, explicit toolchain setup, dependency restore, build, and the checks above. Keep ordinary CI offline from real services and hardware. Store test reports as artifacts; do not publish packages, create releases, or deploy from the first workflow. Record hardware/manual results separately from automated passes.

## Setup and development

### Current checkout

Open this README in a Markdown viewer. Git is initialized locally; no build or run step exists yet. Check the working tree with `git status --short --branch`. Documentation files start untracked; no initial commit or remote is created.

### Prerequisites and first implementation

After scaffolding, create the library and test projects and implement documented numerical examples before optimizing. Run the console sample on a tiny irregular series, then on DataSynth output through a sample adapter. Keep the core independent of that generator.

Install only the dependencies needed for the selected milestone. Add a checked-in sample configuration with harmless defaults; keep local databases, recordings, generated datasets, credentials, and device identifiers outside version control. Pick a license before public distribution; no license is implied by this plan.

### Future commands

These commands are examples for after the referenced projects/files have been created; they will not build this documentation-only repository.

```powershell
dotnet restore TimeSeriesKit.slnx
dotnet test TimeSeriesKit.slnx
dotnet pack src/TimeSeriesKit -c Release -o artifacts/packages
```

## Supporting libraries and boundaries

Shared by SignalBench, AutoDiag session analysis, and EcoGenesis metrics. It has no mandatory dependency on DataSynth or ConfigLens.

Consume stable library versions through local package artifacts initially. Avoid absolute ProjectReference paths into sibling repositories and avoid Git submodules for the MVP. Keep a consumer's first vertical slice runnable using a small local interface or fixture while a supporting package is under development.

## Realistic limitations

No forecasting, causal inference, hard real-time guarantees, or certified signal processing in the MVP. Interpolation can fabricate apparent measurements; derived results must retain method/provenance. Double precision and downsampling have explicit error/visibility tradeoffs.

Scope and performance targets are hypotheses until measured. Document reproducible dataset sizes, hardware, dependency versions, and failure cases alongside any benchmark claim. Do not present planned capabilities as shipped features.

## Documentation provenance

Prepared 2026-10-23. The previous chat supplied the project names and the AutoDiag platform change, but its detailed assistant responses were unavailable content references. The user confirmed the seven proposed application themes in this session; individual designs here are newly proposed plans, not a recovered transcript.

