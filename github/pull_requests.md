# Pull Requests

This page provides a guidelines on how to craft pull requests. The goal is to have semi-structure and checklist which make the pull request easier to review.

## General Guidelines

- Think about the reader while crafting a PR. Take time to split the work after prototyping.
- Make the scope of the PR small.
- Try to change things in as few packages as possible. If changes are needed in too many packages at once it's an indication that the PR scope is too broad or package design is wrong.
- Vendoring should go into a separate PR unless impossible (that shouldn't be an issue with go modules).
- Renaming/moving/general refactoring should go into a separate PR.
- Review your own PR once it got crafted. You yourself get the chance to spot issues you can address immediately, which also makes the life of your colleagues easier.
- Always squash and merge.

## Structure

- For the title and the description we follow guidelines described in official Go wiki:

[https://github.com/golang/go/wiki/CommitMessage#commit-messages](https://github.com/golang/go/wiki/CommitMessage#commit-messages).

## Content

- Unit tests.
    - There is a page describing how to write unit tests [here](../go/unit_tests.md).
