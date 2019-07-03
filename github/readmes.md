# Project Readme and Description

Projects must have a description and a readme that explains what it is for and
where it fits in to the big picture.

Some indication of what other services it collaborates with, what kind of
storage, data structures or CRD's it depends on would be good to have in the
README as well.

Depending on how 'far up the stack' they are, they can have usage examples.

If it is a tool that we make public, then usage examples are a must.

A great example of usage instructions is the `gsctl` repo:
https://github.com/giantswarm/gsctl

Things of note that make it good:

1. It shows you what output to expect when you run the command.

2. It shows you how to get to a state where you can actually do something.
   i.e. one of the first examples is about loging in.
   That's a great first usage example because none of the other examples
   are usable without it.

3. It shows some common tasks that you are most likely going to want to do.