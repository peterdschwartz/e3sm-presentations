---
title: Revisiting E3SM's Components for Emulators
author: Peter Schwartz & Jeffrey Johnson
theme: 
  name: catppuccin-mocha
---

## Earth System Modeling: A System of Systems
<!-- use E3SM's component schematic -->

<!-- end_slide -->

## Components: Interacting Closed Physical Systems
<!-- same picture with hand-drawn circles around systems -->
   - Logical decomposition of the "earth system"
   - Two essential categories: *dynamical* and *data*
   - Typically written in Fortran, with a few newer ones in C++
   - One dynamical component implementation for each type (ATM, LND, OCN, ...)

<!-- end_slide -->

## Coupling: the "Carpet" Design Pattern
<!-- picture of components with a magical cloud in between them -->
   - Coupling logic implemented *for each pair of interacting components(!)* $\leftarrow N^2$
   - Very basic "interface" (a few function calls), very little structure

<!-- end_slide -->

## How Does This Change in the Age of AI?
   - What if components can be **emulators**?
   - What if components of a given type (ATM, LND, OCN) can have **multiple implementations**?
   - What if components contain **parameterizations** with **multiple implementations**?
      * This or that *dynamical* parameterization (difficult but possible)
      * This or that *emulated* parameterization (easier?)
      * Mix and match!

<!-- end_slide -->

## How Does This Change in the Age of AI?
   - What if $N$ (number of supported components) increases quickly?
      * $N^2$ hand-crafted couplings seems... *bad*
   - **What do we require of components**?
   - **How do we simplify the process of coupling components**?

<!-- end_slide -->

## Design Goals for Every Process:
   - Be able to replace process with emulator or toy model without changing the call site.
   - Be able to run process by itself.
   - Be able to answer __Key Questions__:
      * What are the inputs and outputs?
      * What are the prognostic and diagnostic variables?
      * What are the configuration options?
      * How should answers change under different configs? (Metamorphic relations)
      * What are the essential __properties__ for this process?

<!-- end_slide -->

## Why Baseline Comparison Is Not Enough

Common validation pattern:

> Run the model, compare against a previous answer, inspect difference maps.

This is useful, but limited:

- It tells us whether answers changed, not whether they are physically valid.
- It scales poorly: changing resolution, parameters, or physics options requires new baselines, while the tests themselves only cover a small number of timesteps
- It can miss bugs that preserve the "shape" of the answer.
- It does not explain what behavior a process is supposed to satisfy.

<!-- end_slide -->

## Property-Based Testing and Metamorphic Relations

> Documentation as executable code.

Instead of testing one hand-picked input, test many generated inputs based on physically valid ranges

Example Properties:
- Conserved quantities (mass, energy, etc...)
- Outputs are bounded.
- Fractions remain between 0 and 1.
- Reordering independent columns does not change the result.

### Metamorphic Relations:
> How should outputs change between configurations?
* Useful when it's difficult to specify exact ranges/answers.
- Examples:
  * How is lake temperature affected when puddling is enabled?

> __Ideal__: An emulator should pass the same property and metamorphic tests as the process it replaces.

#### Practical caveat:
> Some properties may only hold approximately, within a narrower validity domain, or may be intentionally relaxed. The key is to make these exceptions explicit and testable.

<!-- end_slide -->

## "Best" Programming Practices

> To make these tests and design goals possible, processes need clearer software boundaries.

Key practices:
- Avoid hidden global state.
- Use explicit input/output structs.
- Separate config, parameters, prognostic state, diagnostics, and scratch data.
- Keep physics kernels separate from infrastructure.
- Make units, dimensions, and valid ranges explicit.
- Prefer smaller clearly named functions over giant routines.
- Make each process runnable outside the full coupled model.

> If we can test it independently, we can emulate it independently.
<!-- end_slide -->

## Reusable Validation Infrastructure

Many checks have the same structure across processes:
- conservation of water, energy, carbon, nitrogen, ice, etc.
- bounded variables
- finite fluxes
- monotonic responses
- config invariance tests

We should not write custom diagnostic tests from scratch for every quantity.

> __Goal__: Make common scientific validation patterns reusable and declarative.

Benefits:
- less duplicated test logic
- easier to add tests for new processes
- same tests can apply to mechanistic models, toy models, and emulators
- exceptions become explicit
