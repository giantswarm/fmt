# Javascript Style Guide

Our Javascript Code Style is determined by `prettier`. Please make sure files have
passed through `prettier` before committing them to the repo.

A CI step will enforce that this has happened, failing the CI if it detects that
`prettier` would make any changes.

We use the following config params:
- `--jsx-single-quote`
- `--single-quote`
- `--trailing-comma es5`

