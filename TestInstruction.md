# LPhyBeast beast3 Branch — Testing Guide

**Last updated**: 3 August 2026

## Prerequisites

### Standard build (no SNAPSHOT dependencies)

Every dependency required for the active reactor build — beast3 core, BEASTLabs,
BEAST Classic, Mascot, FLC, ORC, substmodels, morph-models,
sampled-ancestors, coupled-mcmc, and LPhy — is now a released version on Maven
Central (see the root `pom.xml` `<properties>`/`<dependencyManagement>`). No
sibling repos need to be checked out or built from source; a plain build
resolves everything automatically:

```bash
cd ~/Git/LPhyBeast
mvn clean install -DskipTests
```

## Run existing tests

```bash
# Core tests (H5N1, basic scripts)
mvn -pl lphybeast test

# SSM tests (skyline plots -- exercises GTR via substmodels)
mvn -pl lphybeast-ssm test

# FLC tests
mvn -pl lphybeast-flc test

# Mascot tests
mvn -pl lphybeast-mascot test
```

## Test the CLI

```bash
# Show help (new subcommand structure)
mvn -pl lphybeast exec:exec -Dlphybeast.args="--help"

# Convert RSV2
mvn -pl lphybeast exec:exec -Dlphybeast.args="convert ../../linguaPhylo/tutorials/RSV2.lphy"

# Convert with replicates
mvn -pl lphybeast exec:exec -Dlphybeast.args="convert -r 3 ../../linguaPhylo/examples/coalescent/hkyCoalescent.lphy"

# Convert and run BEAST
mvn -pl lphybeast exec:exec -Dlphybeast.args="run -l 10000 ../../linguaPhylo/examples/coalescent/hkyCoalescent.lphy"

# Package management
mvn -pl lphybeast exec:exec -Dlphybeast.args="list"
```

## What changed (summary)

### Entry point

`LPhyBeastMain` replaces `LPhyBeastCMD` as the primary entry point.
Subcommands: `convert`, `run`, `install`, `list`, `remove`.

### Concatenate/Slice elimination

The old pattern of creating individual `RealParameter` state nodes and
joining them with feast's `Concatenate` is replaced by creating a single
`RealVectorParam` and using `VectorElement` (from beast3) to extract
scalar views where needed.

**Before** (RSV2 `r ~ WeightedDirichlet(...)`):
```xml
<stateNode id="r_0" spec="parameter.RealParameter">0.33</stateNode>
<stateNode id="r_1" spec="parameter.RealParameter">0.33</stateNode>
<stateNode id="r_2" spec="parameter.RealParameter">0.33</stateNode>
<x id="r" spec="feast.function.Concatenate">
    <arg idref="r_0"/><arg idref="r_1"/><arg idref="r_2"/>
</x>
```

**After**:
```xml
<stateNode id="r" spec="spec.inference.parameter.RealVectorParam"
           domain="NonNegativeReal">...</stateNode>
<mutationRate id="r_0" spec="spec.inference.parameter.VectorElement"
              vector="@r" index="0"/>
```

### Deprecated types removed

All `RealParameter`, `IntegerParameter`, `Parameter` usage removed from
`BEASTContext` (200 lines deleted). All value converters produce spec types.

### Operators

Spec `DeltaExchangeOperator` replaces deprecated `BactrianDeltaExchangeOperator`.
Tree operators unchanged (not affected by spec changes).

### Dependencies

- `lphybeast/lib/` deleted (25 JARs). All deps via Maven.
- `BEASTVector` vendored to `lphybeast.util` (removed from BEASTLabs).
- `SliceDoubleArrayToBEAST` moved from `lphybeast-feast` to core.
- feast dependency reduced to `ExpCalculator` only (expression handling).
- **Update**: feast has since been removed entirely — beast3 core's spec API
  (`beast.base.spec.inference.parameter.VectorElement`) covers the remaining
  functionality, so no feast dependency remains and `lphybeast-feast` was
  never created.

