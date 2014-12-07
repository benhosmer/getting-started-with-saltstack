## Managing Salt Master With Itself

As you work with Salt, you'll begin to notice something…the Salt Master could be managed, too!

Here's the use-case that made this clear for me.  I apologize if I'm not using your particular flavour of use-case syntax.  I use "The English."

* I have files stored in a Git repository that need to be sync'd with various minions.

* I don't want the minions to have access to the Git repository for security reasons.

* I also don't want to waste resources on the minions storing the Git history on each minion.

* I want to have the Salt Master keep its checkout of the repository up to date.

* When updating the repository changes any files, I want that to trigger a refresh of any minion that requires that Git repository's contents.



