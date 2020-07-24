# Pull Requests

This page provides guidelines on how to craft pull requests. The goal is to have some semi-structure and checklist which make the pull request easier to review.

## General Guidelines

- Think about the reader while crafting a PR. Take time to split the work after prototyping.
- Keep the scope of the PR small.
- Try to change things in as few packages as possible. If changes are needed in too many packages at once it's an indication that the PR scope is too broad or package design is wrong.
- Vendoring should go into a separate PR unless impossible (that shouldn't be an issue with go modules).
- Renaming/moving/general refactoring should go into a separate PR.
- Always squash and merge.

## Recommendations for a PR author

1. Create a `Pull Request Draft` instead of a `Pull Request`

This might help you if you don't know if your idea is the right one but you still want to get feedback. Draft pull request cannot be merged until it is marked as ready for review.
See [Introducing draft pull requests](https://github.blog/2019-02-14-introducing-draft-pull-requests/).

2. Be your first reviewer

Review your own PR once it got crafted. You yourself get the chance to spot issues you can address immediately, which also makes the life of your colleagues easier.

3. Make smaller PR's

To speed up review time and to make it easier for reviewers you want to keep your changes as small as possible.

4. Add a useful description

When creating PR's you should think about writing a useful description which guides reviewers through the code.
Try to explain what you're trying to solve. This should save reviewers a lot of time and speeds up the review process. Also consider to add some screenshots which help to visualize your code changes.

5. Referencing issue(s)

When your change fixes an already existing issue it is helpful to link it in your PR. However you still want add some context around it in your PR.

6. Documenation

Think about documenation and comments within your proposed changes so that it becomes self explanatory, put your own review comments first in order to provide more context.

## Structure

- For the title and the description we follow guidelines described in official Go wiki:

[https://github.com/golang/go/wiki/CommitMessage#commit-messages](https://github.com/golang/go/wiki/CommitMessage#commit-messages).

## Content

- Unit tests.
    - There is a page describing how to write unit tests [here](../go/unit_tests.md).
