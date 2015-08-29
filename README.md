mirrorrepo-git : git mirroring tool with shared objects
========================================================


CAUTION
--------

This tool is still in development.
It might not be safe enough to use this tool.

Currently, this tool has no GC feature. This tool stores many
references to prevent objects to be deleted. However, most references
will be unnecessary after multiple fetches. I will add a feature to remove
such unnecessary references in later development stages.


What is this tool?
-------------------

This tool creates remote git repositories' snapshot and you can recover
them as you need. You can also fetch multiple repositories to save
your disk space.


Why this tool is required?
---------------------------

There are three reasons why I need to make this tool.

First reason is that `git fetch` is (basically) not aware of deleted
branches and tags. I needed to grab exact state of remote repository.

Second reason: sometimes branches/tags are overwritten (by forced update)
and old trees can be lost. When I was working with multiple Linux SoC
repositories, I found a branch is replaced by completely different one.
I wanted history to be preserved, forever.

Third reason: this tool can save your disk space. If you are working
with multiple Linux-kernel repositories, you'll find that most contents
are the same. A git repository is basically a delta-compressed object
storage and we can store contents of multiple repositories and
save disk space (by sharing contents, as this tool achieves).


Required Environment
---------------------

*	POSIX-compliant shell
*	POSIX-compliant commands
*	`mktemp` utility  
	This tool is not a part of POSIX. You should have
	this tool on most environment (GNU Coreutils / BSD...).
*	`git`  
	This tool heavily depends on git.


Data archived/not archived by this tool
----------------------------------------

Data archived:
*	Snapshot of all git references including...
	*	Branches
	*	Tags
	*	Notes
*	Contents corresponding snapshots
	*	Commit tree(s)
	*	Source files

Data not archived:
*	Symbolic references  
	Because `git upload-pack` sends all git references as SHA-1 ID,
	this tool (and all `git fetch`-based tools) cannot preserve
	remote symbolic references.
*	Repository description  
	`git fetch` does not care about repository description.
	Sometimes, we don't have a way to retrieve remote description.


How to use it? (examples)
--------------------------

First, you need to create your directory.

	$ mkdir linux-kernels
	$ cd linux-kernels
	$ mirrorrepo-git init

You will need to add remote repository.

	$ mirrorrepo-git add korg git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git

You might want to add multiple repositories.

	$ mirrorrepo-git add linux-next git://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git

You can remove remote repository but not recommended because mirrorrepo-git
does not try to remove git references associated to deleted repository.
But it might be helpful if you need to rename a remote name.

	$ mirrorrepo-git del linux-next
	$ mirrorrepo-git add korg-next git://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git

You can fetch a remote repository using `fetch` command.

	$ mirrorrepo-git fetch korg

You can also use `fetch-all` command to fetch all remote repositories.

	$ mirrorrepo-git fetch-all

Once you fetch remote repository, you can update local mirror repository.

	$ mirrorrepo-git update korg-next

Of course you can use `update-all` just like `fetch-all`.

	$ mirrorrepo-git update-all

Local mirrors can be found at `mirrors/korg` and `mirrors/korg-next`.
These local mirrors are intended for quick and dynamic mirror.
If you don't specify any snapshot ID, local mirrors are always up to date
(after running proper `update` or `update-all` command).

You can create frozen snapshot too.
However, this tool has no feature to check snapshot ID yet.
You have to confirm snapshot ID by running `git log` on refs-history.

	$ cd refs-history/korg
	$ git log --format=oneline
	....
	$ cd ../..

If you identify that snapshot you need has SHA-1 ID `01234567...`,
you can create frozen snapshot.

	$ mirrorrepo-git snapshot korg korg-snapshot-1 01234567

You can find snapshot at `snapshots/korg-snapshot-1`.

If you want to export a snapshot, use `export` command.
To export as `snapshot` state (default):

	$ mirrorrepo-git export korg /path/to/repository.git 01234567

You can also create ordinal (non-bare) development tree to work on by `-d`.
Also, you can specify `--secure` to detach from shared object store.
Detached (secure) repository can be securely copied to another machine.

	$ mirrorrepo-git export -d --secure korg /path/to/repository 01234567


Combined Repository Format
---------------------------

By running `init` command, it creates a combined repository
which has a bare git repository which contains all archived objects
on the current directory.

In the combined repository directory,
there are some other bare repositories.

*	refs-history/NAME  
	Repository which holds all reference information in the "refs" file.
	It's used when making mirrors and updated on fetch.
*	mirrors/NAME  
	Local mirrors created and updated on `update` command.
*	snapshots/NAME  
	Snapshots created by `snapshot` command.


Security Considerations
------------------------

When fetching from remote repositories, it can leak other remote
repositories' information. This is the default behavior. If you set
`--secure` flag when fetching from remote, mirrorrepo-git uses a bit
inefficient way to fetch remote objects (not as inefficient as holding
whole local repository per remote).

When fetching from local mirrors/snapshots, it can leak other repositories'
information. This is by-design. If you don't want to leak such information,
just export a repository with `--secure` flag. Exported repository
(with `--secure` flag) no longer shares object storage and
no information leaks should occur.


License
--------

This program is licensed under the ISC License.

	Copyright(C) 2015 Tsukasa OI.
