# Release observation bundles

This directory contains only public, signed observations selected by the
reviewed Product Hub state. Each bundle records one bounded provider fact and
expires:

- the GitHub adapter requires an annotated tag;
- it resolves that tag through GitHub's public API to one commit and tree;
- the detached signature protects the exact redacted observation after the
  live fetch; and
- the Product Hub verifies the signature, freshness, product binding, and
  release URL before publication.

The observer key is separate from the maker and economics keys but remains
under the same Lumen operator. Its private half stays root-only outside Git. A
valid bundle proves what GitHub's public API returned at one time. It is not an
independent witness and does not prove test execution, reproducibility, code
quality, origin control, authorship, use, or independent operation.
