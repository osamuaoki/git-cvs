# git-cvs
Access CVS using git (like git-svn)


    git-cvs - bidirectional operations between a single CVS tree and git
    
    usage: git cvs <command> [<optarg1> [<optarg2> [<optarg3>]]]
    
    Available commands:
    
    init    Initialize the local git repository only with the latest snapshot
            source checked out from the remote CVS repository to start the
            approach 1 workflow.  This does not use the cvsps command to be fast
            and does not require the SSH access to the remote CVS repository.
            Here:
              * <optarg1> : $CVSROOT (required)
              * <optarg2> : $MODULE  (required)
              * <optarg3> : alternative name for the rabase branch (opt.)
            Remote CVS parameters $CVSROOT and $MODULE are the ones used as:
              $ cvs -d $CVSROOT checkout $MODULE
            This command creates the $MODULE/ directory to store the source.
            The "master" branch tracks the HEAD of thev remote CVS repository.
            The $MODULE/.git/.gitcvsrc records the git-cvs parameter.
    
    sync    Initialize the local git repository with the complete history of the
            remote CVS repository to start the approach 2 workflow as follows.
    
            Remote CVS -> rsync -> Local CVS -> git-cvsimport (cvsps) ->+
                    ^     (ssh)           |                             :
                    |                     +-> cvs checkout ->+         hack
                    |                                        |          :
                    +<------------ git-cvsexportcommit <-----+<---------+ 
            Here:
              * <optarg1> : $CVSROOT (required)
              * <optarg2> : $MODULE  (required)
              * <optarg3> : use "<optarg3>/master" instead of "cvs/master" (opt.)
            Remote CVS parameters $CVSROOT and $MODULE are the ones used as:
              $ cvs -d :ext:$CVSROOT checkout $MODULE
            This command creates 3 directories: rsyncdir/ cvsdir/ gitdir/ and 1
            file: .gitcvsrc
              * rsyncdir/ : the local mirror of the CVS repository
              * cvsdir/   : the CVS checkout
              * gitdir/   : the local git repository (no CVS/* files)
                * "cvs/master" branch : track the remote CVS repository
                * "master" branch     : track the local changes
              * .gitcvsrc : records the git-cvs parameter.
            Try gitk in the gitdir/ directory to see the full project history.
    
    update  Update the local git repository by the latest remote CVS repository.
            For approach 1, execute this in the $MODULE directory.
            For approach 2, execute this in the gitdir/ directory.
            This is the default action without <command>.
    
    rebase  Update the local git repository by the latest remote CVS repository
            as in "git cvs update" and rebase the local changes to the remote CVS
            HEAD.
    
    dcommit Commit from the local git repository to the remote CVS repository.
            The CVS commit happens on each git commit.  So make sure to organize
            pending git commits using "git rebase -i ..." in advance.
            For approach 1, execute this in the $MODULE directory.
            For approach 2, execute this in the gitdir/ directory.
    
    Note:   For approach 1, the git and cvs packages are required.
            For approach 2, the rsync, ssh, git, cvs, and cvsps packages are
            required.
    
            You can prefix <command> with "Debug " to enable the shell trace.
            Commands may be shorted to the first character. (case sensitive)
    
            If the value of the CVS keyword expansion (e.g. $) needs to
            be identified, please seek it in the CVS/Entries file.

If you need to use host specific username, please export the USER environment
variable to it or seek to set it via  ~/.ssh/config .  See ssh_config(5).

If you don't have read access to some files in ${CVSROOT}/CVSROOT or
${CVSROOT}/$MODULE, you can't use approach 2.  Please pay attention to file
permissions of the CVS repository for your account.

If your remote account for CVS is "foo" at "cvs.example.org" and the data is
stored in the "/srv/cvs" directory, set $CVSROOT in the above as

    foo@cvs.example.org:/srv/cvs

The $MODULE should not have tailing / .

For example, if you have a CVS write access account `foo` via `ssh` for the
Debian web page hosted at cvs.debian.org
(https://www.debian.org/devel/website/)

    $ CVSROOT="foo@cvs.debian.org:/cvs/webwml"
    $ git cvs sync "$CVSROOT" webwml

This gives a nice git repository with the full history.

    $ tree --dirsfirst -aL 1
    .
    ├── cvsdir
    ├── gitdir
    ├── rsyncdir
    └── .gitcvsrc
    
    3 directories, 1 file

# Required packages

* git
* git-cvs (debian package for git-cvsimport)
* cvs
* rsync

# References for CVS <--> GIT

Here are some interesting things I found on the net.

* GIT overview
 * https://git.wiki.kernel.org/index.php/Interfaces,_frontends,_and_tools
* git-cvsexportcommit
 * https://lists.gnu.org/archive/html/bug-gnulib/2006-11/msg00363.html
* git-cvsexportcommit -W
 * http://vmiklos.hu/blog/new-in-git-1-5-6-git-cvsexportcommit-w.html
* Bidirectional script (I did not read it yet)
 * https://github.com/mikjo/bigitr

