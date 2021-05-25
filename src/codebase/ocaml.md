# OCaml

## Why OCaml?

- garbage collected
- type safety
- pure, somewhat

## Coding guidelines

- no more open... very hard to follow code
- functors are powerful but hard to follow imo
- lack of comments...
- lack of auto-generated doc somewhere
- lack of README files
- no one letter variables

## Issues with the current build system

- no lock files
- no local switch
- no versioning (would require a main opam package file)
- no clear pinning (would require a format to specify the pins and a wrapper for dune)
- a dev infra team would need to maintain the wrapper for dune
- too many internal packages that should be published to an opam repository
- git submodules instead of pins
