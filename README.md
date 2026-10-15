# Palimpsest

**Build software by improving what teaches it.**

Palimpsest explores a knowledge-patch workflow for executable codebooks. Engineers preserve discoveries in the instruction and graders that teach a system; agents build, review, and independently reconstruct the affected exercises.

The project connects three records:

- **Knowledge Patches (KPs):** what was learned, why, and how the teaching changes.
- **The integrated codebook:** chapters with explicit prerequisites and checked exercises.
- **Implementation CLs:** ordinary append-only changes against HEAD, each owned by exactly one KP.

A separate chapter construction chain reproduces the accepted source tree without rewriting submitted implementation history. Human learning through visible engineering explanations is part of the workflow, not an optional final report.

## Read the book

1. [Introduction](book/chapter-01.md) — what we are building, why, and how.

## Status

This repository begins with the Introduction. The Palimpsest CLI, construction manager, and automated KP workflow are not yet implemented. The book and software will develop together through small, tested exercises.

## License

Licensed under the [Apache License, Version 2.0](LICENSE).
