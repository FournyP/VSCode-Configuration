---
agent: agent
description: This prompt is used to generate a Go "model builder" type for a specified struct
---

I want you to generate a Go “model builder” type for the specified struct.

Behavior and conventions (must follow):
- Pattern:
  - A builder type named <Type>NameBuilder (e.g., VaultBuilder).
  - Private fields for each target struct field.
  - Constructor: New<Type>NameBuilder() that seeds sensible defaults using existing helpers (common_tests.RandomEthAddress, faker, time.Now(), etc.), matching existing builders’ style.
  - Fluent setters: With<FieldName>(value ...) *<Type>NameBuilder for each field; set and return the builder.
  - Build(): returns a pointer to the target model with any big.Int/string dual fields populated like the existing builders (e.g., both string and BigInt forms), timestamps preserved, and nested structs/maps filled.
- Imports:
  - Use the project module path derived from go.mod.
  - Reuse helpers: common_tests.RandomEthAddress/RandomBigInt, faker, time.Now(), etc., consistent with current builders.
- Naming/data conventions:
  - Keep field naming in line with the target model.
  - For numeric amounts that have string + big.Int representations, construct both (see existing RewardPoolLifetimeVaultBuilder and VaultBuilder).
  - For timestamps, prefer time.Now().UTC() unless a clear alternative is needed.
- Output:
  - Return only the builder source file content, no commentary.
  - Package should match the target folder (e.g., model_builders).

Example shape (adapt to the target model):
```golang
package model_builders

import (
    "time"
    "math/big"

    "<module>/src/.../models"
    "<module>/tests/common_tests"
    // faker or other helpers if already used in this codebase
)

type <Type>NameBuilder struct {
    // private fields mirroring the target struct
}

func New<Type>NameBuilder() *<Type>NameBuilder {
    // seed defaults using helpers and sensible values
    return &<Type>NameBuilder{/* ... */}
}

func (b *<Type>NameBuilder) WithFieldX(v <type>) *<Type>NameBuilder { b.fieldX = v; return b }
// ... other With* setters ...

func (b *<Type>NameBuilder) Build() *models.<TypeName> {
    // compute big.Int/string duals if needed
    // return &models.<TypeName>{ ... }
}