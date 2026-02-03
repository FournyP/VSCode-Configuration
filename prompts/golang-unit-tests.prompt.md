---
agent: agent
description: Generate Go unit tests following the established conventions and style.
---

Generate Go tests that mirror the existing style in this repo.

Behavior and conventions (must follow):
- Assertions: github.com/stretchr/testify/assert only.
- Suite pattern:
  - Define a suite type: When<WHAT_YOU_ARE_TESTING>TestingSuite.
  - Provide When<WHAT_YOU_ARE_TESTING>BeforeEach(t *testing.T) that builds a fresh suite (create gomock controller, mocks, sut, logger = zap.NewNop()).
  - If cleanup is required (e.g., temp DB), also add When<...>AfterEach(suite *When<...>TestingSuite).
  - Root entry: TestWhen<WHAT_YOU_ARE_TESTING>(t *testing.T).
- Naming/layout:
  - File name: when_<what_you_are_testing>_test.go.
  - Package: prefer external package_test when possible; otherwise same package with a brief comment if unexported symbols force it.
  - t.Run nesting: Given -> Should. Use small helper initSuite closures inside Given blocks to set mock expectations.
- Parallelism:
  - Call t.Parallel() at the root test and inside subtests when safe (no shared mutable state / data races). Omit and add a short comment if not safe.
- Mocks and SUT setup:
  - Use gomock.NewController(t) inside BeforeEach; create mocks from the *_mocks packages provided in context.
  - Build the SUT with the project constructors (e.g., services.NewX, repositories.NewX) and inject mocks.
- Data builders:
  - Prefer existing builders/helpers (common_tests.RandomEthAddress/RandomBigInt, model_builders.*) instead of ad-hoc randomness.
  - For floats use assert.InDelta with a tolerance (default 1e-9 or options.float_tolerance).
- Error checks: assert both the presence/absence of error and the returned values.
- Table-driven cases:
  - Add a small table when appropriate; call tc := tc and t.Run(tc.name, ...) with optional t.Parallel() per case if safe.
- Imports:
  - Derive module path from go.mod when present; import the target package via that path.
  - Include gomock, zap, and any *_mocks, model_builders, common_tests helpers that you actually use.
- Output:
  - Return only one complete Go test file (buildable with `go test`), no extra commentary.

Example shape:

```golang
package <pkg>_test

import (
    "testing"

    "<module>/src/.../targetpkg"
    "<module>/tests/.../mocks"
    "<module>/tests/common_tests"
    "<module>/tests/common_tests/model_builders"
    "github.com/stretchr/testify/assert"
    "go.uber.org/mock/gomock"
    "go.uber.org/zap"
)

type When<WHAT_YOU_ARE_TESTING>TestingSuite struct {
    sut *targetpkg.SomeType
    // mocks...
}

func When<WHAT_YOU_ARE_TESTING>BeforeEach(t *testing.T) *When<WHAT_YOU_ARE_TESTING>TestingSuite {
    ctrl := gomock.NewController(t)
    // create mocks with ctrl
    // sut := targetpkg.New...
    return &When<WHAT_YOU_ARE_TESTING>TestingSuite{
        sut: /* ... */,
        // mocks...
    }
}

func TestWhen<WHAT_YOU_ARE_TESTING>(t *testing.T) {
    t.Parallel()
    tol := 1e-9

    t.Run("Given <preconditions>", func(t *testing.T) {
        t.Parallel()
        suite := When<WHAT_YOU_ARE_TESTING>BeforeEach(t)

        initSuite := func(s *When<...>TestingSuite /*, args...*/) {
            // set EXPECTs on mocks
        }

        t.Run("Should <expected behavior>", func(t *testing.T) {
            t.Parallel()
            initSuite(suite /*, args...*/)
            // Act
            // Assert
            // assert.NoError(t, err)
            // assert.InDelta(t, want, got, tol) // for floats
        })

        // optional table-driven cases...
    })
}