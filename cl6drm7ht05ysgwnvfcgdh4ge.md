## A case for using git worktree

# Context

Let's say I am working on a project named "short-ruby" 😉

That project is currently located somewhere like 

`/home/lucian/projects/short-ruby` 

Let's say that I am currently working on a complex feature named "automate newsletter".

This feature has multiple branches:
- `twitter-integration` branched from `main`
- `substack-integration` branched from `main`
- `collect-tweets` branched from `automate-n/twitter-integration`

To test all this stuff, I am also creating a temp merge branch `all` where I want to test if all work well together before deciding to integrate them to `main`. 

For simplicity 

# Current task

The first 2 branches are implemented, and I am currently working on `collect-tweets`.

Now imagine that while working on this branch, I discover that there is something that needs to be changed, and it is related to integration with Twitter, like I need access to also `moments`, not only lists. 

This change should be part of `twitter-integration` branch.

## Git solution without git worktree

What are my options now? 

### Simple solution

The simplest one is to `git stash` my current changes from `collect-tweets` and then `git checkout` the `twitter-integration` branch do the changes there, commit them and then come back to `collect-tweets` and `git rebase`. 

It might look something like this:

```bash
# work on collect-tweets
(short-ruby)$ git stash
(short-ruby)$ git checkout twitter-integration

#... do the work

(short-ruby)$ git commit -m "Added support for querying moments"
(short-ruby)$git checkout collect-tweets
(short-ruby)$ git rebase twitter-integration
(short-ruby)$ git stash pop

# resume previous work on collect-tweets
```

### Elegant but more complex solution

Of course, another solution would be to commit the changes in the branch `collect-tweets` and then `git cherry-pick` them to the correct branch. 

This is an elegant solution, but you have to be careful and remember to make sure your commit was atomic and did not contain anything from the current branch. Of course, you can also fix that with `git rebase -i`, but still, that is also a solution that works when you always do logical atomic commits. 


## More changes

Now imagine that you resumed work on "collect-tweets" and then the next day, a bug needs to be fixed on `main` and the bug means changing a method in a class that was refactored in `twitter-integration`

You will need again to do a series of `stash`/`checkout`/`rebase` that now will also mean multiple git checkout and rebases as you will want to propagate the changes from `main` to `twitter-integration` to `collect-tweets`. 

Probably cannot do cherry picking while you are in `twitter-integration` or `collect-tweets` because you refactored that code already so cherry-picking will fail. 

Here I think comes in handy git worktree

## Git worktree

Git worktree is a simple command that will map a local folder to a branch.

Imagine I said I have the project in `/home/lucian/projects/short-ruby`

Here is what I can do before going into all branches: 

```bash
# Rename folder short-ruby to main and create a new folder short-ruby and put main inside
(short-ruby)$ cd ... ; mv short-ruby main ;  mkdir short-ruby ; cd short-ruby
(short-ruby)$ mv ../main ./ ; cd main
(short-ruby/main)$ git checkout main
(short-ruby/main)$ git worktree add ../twitter-integration witter-integration
(short-ruby/main)$ git worktree add ../substack-integration substack-integration
(short-ruby/main)$ git worktree add ../collect-tweets collect-tweets
(short-ruby/main)$ git worktree add ../all all
``` 
Now if I execute `git worktree list` it will show an output like this:

```bash
/home/lucian/projects/short-ruby/main                   bff36930 [main]
/home/lucian/projects/short-ruby/twitter-integration    f840805c [twitter-integration]
/home/lucian/projects/short-ruby/substack-integration   8286fd79 [substack-integration]
/home/lucian/projects/short-ruby/collect-tweets         52d9bf8c [collect-tweets]
/home/lucian/projects/short-ruby/all                    7cc380a4 [all]
```
What happen is that it created those folders and inside each one it did a checkout with that specific branch.

Now my flow will look like this:
I will `cd short-ruby/collect-tweets` and work in that folder everything related to the branch `collect-tweets`. 

If I need to do a change on `twitter-integration` branch: 

```bash
cd twitter-integration
# do the changes
git commit -m "Added support for querying moments"

cd collect-tweets
git rebase twitter-integration
# continue work 
```
Do I need to do some changes on `main`?

Simple: 
```bash
cd main
# do the changes
git commit -m "Bug is fixed"

cd twitter-integration
git rebase main

cd collect-tweets
git rebase twitter-integration
```

Of course, I can automate rebase on all branches (probably) with something like:

```bash
# rebase-automate-newsletter.sh
cd ~/projects/short-ruby/main ; git pull

cd ~/projects/short-ruby/twitter-integration; git pull ; git rebase main
cd ~/projects/short-ruby/substack-integration; git pull ; git rebase main

cd ~/projects/short-ruby/collect-tweets; git pull ; rebase twitter-integration
```
