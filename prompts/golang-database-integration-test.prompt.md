---
agent: edit
description: Generate Go integration tests for a repository following the established conventions and style.
---

Generate Go integration tests for a repository using the project’s conventions.

Behavior and conventions (must follow):

- Use github.com/stretchr/testify/assert for assertions.
- Suite pattern:
  - Define a suite: When<WHAT_YOU_ARE_TESTING>TestingSuite.
  - BeforeEach: spins up a fresh Postgres via container_builders.CreatePostgreSQLDatabase(), finds migrations path via common_tests.FindMigrationFolderPath(), builds the database client with zap.NewNop(), applies migrations, constructs the repository under test (and any sibling repos needed for seeding).
  - AfterEach: destroys the DB via container_builders.DestroyPostgreSQLDatabase.
- Test entry:
  - Root: TestWhen<WHAT_YOU_ARE_TESTING>(t \*testing.T).
  - Nest with t.Run using Given -> Should.
  - Use t.Parallel() at the root and subtests only when safe (no shared mutable state); otherwise omit and add a brief comment.
- Seeding:
  - Use model_builders (e.g., New<Entity>Builder) to create entities, then save via repos to set up state.
- Coverage expectations:
  - Include positive and negative paths: found / not found, updates, existence checks, empty results.
  - Assert both returned values and errors (nil vs. not found vs. other).
- Imports:
  - Derive module path from go.mod; import the db client package, repositories, settings, container_builders, common_tests, model_builders, zap, assert, gorm (if checking not-found), and testing.
  - Prefer package_test (external) unless unexported symbols force same-package; if so, note briefly why.
- File naming: when\_<what_you_are_testing>\_test.go.
- Output: Return only the full Go test file content (buildable with `go test`), no extra commentary.

Example shape (adapt to the target repo and scenarios):

```golang
package repositories_test

import (
    "testing"

    "<module>/src/<db_client>"
    "<module>/src/<db_client>/repositories"
    "<module>/src/<db_client>/settings"
    "<module>/tests/common_tests"
    "<module>/tests/common_tests/container_builders"
    "<module>/tests/common_tests/model_builders"
    "github.com/stretchr/testify/assert"
    "go.uber.org/zap"
    "gorm.io/gorm"
)

type When<WHAT_YOU_ARE_TESTING>TestingSuite struct {
    dbName string
    sut    *repositories.<RepoType>
    // other repos if needed for seeding
}

func When<WHAT_YOU_ARE_TESTING>BeforeEach() *When<WHAT_YOU_ARE_TESTING>TestingSuite {
    dbName, conn := container_builders.CreatePostgreSQLDatabase()
    migrationsPath, err := common_tests.FindMigrationFolderPath()
    if err != nil { panic(err) }

    dbClient, err := <db_client>.NewClient(conn, zap.NewNop(), &settings.MigrationsSettings{MigrationsFolderPath: *migrationsPath})
    if err != nil { panic(err) }
    dbClient.Migrate()

    sut := repositories.New<RepoType>(dbClient)
    // init sibling repos if seeding requires them

    return &When<WHAT_YOU_ARE_TESTING>TestingSuite{
        dbName: dbName,
        sut:    sut,
    }
}

func When<WHAT_YOU_ARE_TESTING>AfterEach(suite *When<WHAT_YOU_ARE_TESTING>TestingSuite) {
    container_builders.DestroyPostgreSQLDatabase(suite.dbName)
}

func TestWhen<WHAT_YOU_ARE_TESTING>(t *testing.T) {
    t.Parallel()

    t.Run("Given <context>", func(t *testing.T) {
        t.Parallel()
        suite := When<WHAT_YOU_ARE_TESTING>BeforeEach()
        defer When<WHAT_YOU_ARE_TESTING>AfterEach(suite)

        // seed using model_builders and suite.sut / sibling repos

        t.Run("Should <expected>", func(t *testing.T) {
            t.Parallel()
            // Act
            // Assert (values + errors)
        })
    })

    t.Run("Given not found", func(t *testing.T) {
        t.Parallel()
        suite := When<WHAT_YOU_ARE_TESTING>BeforeEach()
        defer When<WHAT_YOU_ARE_TESTING>AfterEach(suite)

        // no seed

        t.Run("Should return not found error", func(t *testing.T) {
            t.Parallel()
            _, err := suite.sut.Get(...)

            assert.Error(t, err)
            assert.ErrorIs(t, err, gorm.ErrRecordNotFound)
        })
    })

    // Add update/existence/empty-list cases similarly
}
```
