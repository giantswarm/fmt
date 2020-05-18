# Changelog

In our repositories we keep `CHANGELOG.md` files describing changes between
releases of the project. In general we follow [keep
a changelog][keepachangelog] guidelines.

**The changelog should be able to answer two questions:**

- Is there a reason to upgrade to the new version?
- What breaking changes were introduced? How the migration to the new
  version looks like?

**Those questions should be answered from the perspective of the user of the
repository.** The user can be something else for different types of codebases:

- For CLI program: a person or entity calling the binary.
- For managed app: a person or entity installing the application.
- For library: another codebase using the library.
- For operator: a person or entity creating the CR consumed by this operator.

## Examples of reasons to upgrade

- Bug fix.
- Security fix.
- New functionality.
- Improved existing functionality.

## Examples of breaking changes

For CLI program:

- Renamed command or flags.
- Changed behaviour of a command.

For managed app:

- Renamed value in chart's values.yaml file.

For library:

- Changed API function name or arguments.
- Changed behaviour of API function.

For operator:

- Changed defaults for CR empty fields.
- Changed the desired state of the system for the same CR. E.g. not public but
  internal load-balancer is created for the same CR.

## Examples of noise entries

We should try to not mechanically write in changelog everything that is changed
in the repository because not all changes are breaking and not all of them are
the reason to upgrade for the end user.

Here are some examples of not useful changelog entires creating unnecessary
noise and polluting the changelog:

- Update documentation.

This is important for the end user to have up to date documentation. But
documentation changes are not a reason to upgrade. They are also not released
and non-breaking.

- Rename internal variable from A to B.
- Refactor internal package code.
- Update architect orb version in CircleCI.

Those are not a breaking changes. Code refactoring and way of building the code
is not a reason for the end user to update.
a reason for the end user to update.

[keepachangelog]: https://keepachangelog.com/en/1.0.0/
