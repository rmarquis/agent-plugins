# Language Profiles

Quick-reference conventions for supported implementation languages.

## Kotlin

- **Source root**: `src/main/kotlin/`
- **Test root**: `src/test/kotlin/`
- **Build tool**: Gradle (Kotlin DSL preferred)
- **Test framework**: JUnit 5 + Kotest or Strikt
- **Naming**: `PascalCase` types, `camelCase` functions/variables, `SCREAMING_SNAKE_CASE` constants
- **Packages**: reverse domain, lowercase
- **Error handling**: `Result<T>` for expected failures, exceptions for exceptional cases
- **Documentation**: KDoc for public APIs
- **Idioms**: data classes, sealed interfaces, extension functions, coroutines for async

## TypeScript

- **Source root**: `src/`
- **Test root**: `src/` or `tests/`
- **Build tool**: `tsc`, `vite`, `esbuild`, or `tsx`
- **Test framework**: Vitest or Jest
- **Naming**: `PascalCase` types/classes, `camelCase` functions/variables, `SCREAMING_SNAKE_CASE` constants
- **Error handling**: `Result<T, E>` or `neverthrow`, avoid throwing for expected errors
- **Documentation**: TSDoc for public APIs
- **Idioms**: discriminated unions, readonly, functional transformations

## Python

- **Source root**: project root or `src/`
- **Test root**: `tests/`
- **Build tool**: `pytest`
- **Test framework**: pytest
- **Naming**: `PascalCase` classes, `snake_case` functions/variables, `SCREAMING_SNAKE_CASE` constants
- **Error handling**: explicit `Result` or exceptions; prefer explicit returns for domain errors
- **Documentation**: Google-style or NumPy-style docstrings
- **Idioms**: dataclasses, type hints, list comprehensions, generators

## Java

- **Source root**: `src/main/java/`
- **Test root**: `src/test/java/`
- **Build tool**: Maven or Gradle
- **Test framework**: JUnit 5 + AssertJ
- **Naming**: `PascalCase` types, `camelCase` members, `SCREAMING_SNAKE_CASE` constants
- **Packages**: reverse domain, lowercase
- **Error handling**: checked exceptions or `Optional`; consider `Result` with libraries like Vavr
- **Documentation**: Javadoc for public APIs
- **Idioms**: records (Java 16+), sealed classes, interfaces, streams

## Go

- **Source root**: project root
- **Test root**: alongside source (`*_test.go`)
- **Build tool**: `go build`, `go test`
- **Test framework**: standard `testing` + testify
- **Naming**: `PascalCase` exported, `camelCase` unexported
- **Error handling**: explicit `error` returns, no exceptions
- **Documentation**: package and function comments starting with the name
- **Idioms**: structs, interfaces, channels, `err` as last return value

## Rust

- **Source root**: `src/`
- **Test root**: inline modules (`#[cfg(test)]`) or `tests/`
- **Build tool**: Cargo
- **Test framework**: built-in test harness
- **Naming**: `PascalCase` types/traits, `snake_case` functions/variables, `SCREAMING_SNAKE_CASE` constants
- **Error handling**: `Result<T, E>`, `?` operator, `thiserror` for custom errors
- **Documentation**: `///` for public APIs
- **Idioms**: ownership, pattern matching, enums, traits

## C#

- **Source root**: project root or `src/`
- **Test root**: `tests/` or `*.Tests/`
- **Build tool**: `dotnet`
- **Test framework**: xUnit, NUnit, or MSTest
- **Naming**: `PascalCase` types/methods/properties, `camelCase` parameters/local variables
- **Namespaces**: `Company.Product`
- **Error handling**: exceptions for exceptional cases; `Result` or `OneOf` for domain errors
- **Documentation**: XML documentation comments
- **Idioms**: records, LINQ, async/await, dependency injection

## General Rules

1. Match the existing project's conventions first.
2. Use the project's established build and test commands.
3. Follow the language's idiomatic error-handling style.
4. Document only public APIs; keep internal comments minimal and meaningful.
5. Prefer immutable data structures where the language supports them well.
