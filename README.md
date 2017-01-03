Git Pre-Receive Hook to Validate Commits
========================================

The developers tend to be obsessed about code style.  It is exhausting to edit
files again and again to have a consistent style.  This project provides
a Git pre-receive hook to validate pushed commits on the Git server side.
The hook avoids all issues by rejecting any commit with inconsistent style
to get in to the repository in the first place.

The pre-receive hook searches runs some checks on the commits on its own,
and searches for programming language specific syntax checkers on the PATH.
The process is pretty fast, because  only the changed files on the pushed
commits are passed to the syntax checkers in parallel.  It wouldn't slow you
down unless your commits are touching hundreds of files.

Installation
------------

Link the [script](igcommit-receive) to `hooks/pre-receive` on you Git
repositories on your Git server:

```shell
ln -s igcommit-receive /home/git/repositories/myproject.git/hooks/pre-receive
```

Features
--------

* Validate received commits one by one not just the last one
* Only validate added or modified files on the commits
* Report all problems before failing
* Check for duplicate commit summaries
* Check for misleading merge commits
* Check commit message best practices (80 lines, first line summary...)
* Check commit summary formatting
* Validate commit tags against a list `[BUGFIX]`, `[FEATURE]`, `[WIP]`...
* Check for changed file paths
* Accept commits tagged as `[WIP]` or `[MESS]` with issues
* Check executable bits and shebangs
* Check Puppet files with `puppet parser validate` and `puppet-lint`
* Check Python files with `flake8` or `pycodestyle` and `pyflakes`
* Check Ruby files with `rubocop`
* Check shell scripts with `shellcheck`
* Check JavaScript files with `jscs` (requires `.jscs.json` on repository)
* Run the external check commands in parallel

Here is an example problem output:

```
=== CheckDuplicateCommitSummaries on CommitList ===
* Add nagios check for early expiration of licenses
* Add nagios check for early expiration of licenses for Jira (#18)

=== CheckCommitSummary on 31d0f6b ===
* summary longer than 72 characters

=== CheckCommitSummary on 6bded65 ===
* past tense used on summary

=== CheckCommand "flake8" on src/check_multiple.py at 6bded65 ===
* line 7:80: E501 line too long (140 > 79 characters)
* line 8:80: E501 line too long (201 > 79 characters)
* line 9:80: E501 line too long (106 > 79 characters)
* line 10:5: E225 missing whitespace around operator
* line 17:80: E501 line too long (122 > 79 characters)
* line 17:85: E203 whitespace before ','

=== CheckCommitMessage on 6fdbc00 ===
* line 7 is longer than 80
* line 9 is longer than 80
```

Configuration
-------------

The script itself is currently configuration free.  Though, some of the syntax
checkers called by the script uses or requires configurations.  Those
configuration files has to be on the top level of the Git repository.

| Syntax Checker | Configuration File | Note     |
|----------------|--------------------|----------|
| flake8         | .flake8            | optional |
| jscs           | .jscs.json         | required |

Pros and Cons of Pre-receive Hook
--------------------------------

### Continuous Integration Server

A continuous integration server can run such checks with the many other things
it is doing.  Moving this job from it has many benefits:

* Synchronous feedback
* More efficient
* Disallow any commit violating the rules

### Pre-commit Hook

Even though, pre-receive hook gives later feedback than pre-commit hook, it
has many advantages over it:

* No client side configuration
* Plugins has to be installed only once to the Git server
* Everybody gets the same checks
* Enforcement, nobody can skip the checks
* Commit checking (pre-commit hook only gets what is changed in the commit)

### IDE Integration

The same advantages compared to pre-commit hooks applies to IDE integration.
Though, IDE integration gives much sooner and nicer feedback, so it is
still a good idea, even with the pre-receive hook.

Dependencies
------------

The script has no dependencies other than Python 2.7, 3.3 or above.  It runs
the validation commands using the shell.  The necessary ones for checked
repositories need to be installed separately.  See the complete list of
commands on the [script](igcommit-receive).  The commands which are not
available on the `PATH` is not going to be used.

Testing
-------

I found it useful to check what the script would have complained if it would
be active on different Git repositories.  You can run a command like this
to test it on a Git repository against last 50 commits:

```shell
git log --reverse --oneline HEAD~50..HEAD | sed 's/^/x /' | python ../igcommit/igcommit-receive
```

License
-------

The script is released under the MIT License.  The MIT License is registered
with and approved by the Open Source Initiative [1].

[1] https://opensource.org/licenses/MIT
