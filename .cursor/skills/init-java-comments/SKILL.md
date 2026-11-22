---
name: init-java-comments
description: >-
  Audits every Java file in the project against the java-comments rule and
  adds or replaces Javadoc, OpenAPI annotations, and inline comments in
  source. Use when the user asks to init-java-comments, init java comments,
  initialize Java comments, document the Java project, or apply the
  java-comments rule across the repo.
disable-model-invocation: true
---

# Init Java comments

Bring the whole project in line with [java-comments](.cursor/rules/java-comments.mdc). Edit Java sources directly. Do not add scripts, generators, or bulk-edit tooling that create or update comments.

## Workflow

1. Read `.cursor/rules/java-comments.mdc` and follow it. If the file is missing, stop and say so.
2. Decide whether the project uses Swagger OpenAPI (`io.swagger.v3.oas.annotations`): search dependencies and Java imports. Remember the result for every file.
3. List every `**/*.java` file. Skip generated and build output (`target/`, `build/`, `.git/`).
4. For each file, read the source and update comments in place:
   - Add a missing 1–2 line English Javadoc on each class, field, and method that the rule still documents with Javadoc.
   - Replace a Javadoc or OpenAPI description that no longer matches the code.
   - When OpenAPI is in use, document controllers and their methods with `io.swagger.v3.oas.annotations` and remove Javadoc from those types and methods.
   - When OpenAPI is in use, document informational fields of classes that appear in OpenAPI Schemas with `@Schema` and remove Javadoc from those fields.
   - On a complex method, add short English `//` comments at non-obvious steps. Do not restate obvious code. Remove `//` comments that only repeat the code.
5. Leave behavior, signatures, and formatting unchanged except for the comment and annotation edits the rule requires.
6. After the pass, report files changed, what was added or replaced, and any file you skipped and why.
