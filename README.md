git-devops
==========

> *Check in* ***all*** *the things...*

Target Use Cases
----------------

- Automated systems administration
    - Provide an underlying DVCS for managing binaries, large blobs,
      and configuration files for existing DevOps tools, supporting
      diverse algorithms and philosophies
    - Provide a native management algorithm for host
      management, supporting the two major models in use today:
        - Convergence -- take over hosts in unknown state, bring them
          towards a known state over time
        - Congruence -- start with hosts in known state, keep them
          that way as they evolve 
    - Bare-metal installation (with git-devops in PXE miniroot)
- Backups, including archival and offsite
- Media storage, both online and offline


Design Goals
------------

- Keep installation simple -- 'pip install git-devops'
- Require no central or master server
- Use git's ssh protocol for network communications
- Use git's porcelain for merge, commit, and branch management.
- Use git's plumbing for external blob, tree, and journal storage 
- Use librabinpoly for slicing large blobs into smaller ones
- Use gnupg for release signatures
- Maintain sparse repositories -- automatically pull only those blobs
  needed for current operations on target machine, then clean up
  afterwards



Status and Roadmap
------------------

Prototype in development right now, first production release by
February 2015.

I'll push to frequently github as I go along.  Pull requests are
welcome and encouraged.



Thinking
--------

> *Any sufficiently complicated C or Fortran program contains an ad
> hoc, informally-specified, bug-ridden, slow implementation of half
> of Common Lisp.  -- Philip Greenspun*

Host configuration management tools are fundamentally version control
systems -- "it's all about the bits".  Those bits on disk are what
determine the machine's behavior.  Own the bits and you own the
machine. 

Any decent host management tool can be thought of as a special-case
implementation of a form of version control in which file content and
process execution are both first-class primitives.  The aim of these
primitives is ultimately to control the bits on disk, either by
writing directly over them, or running another process that does so.

Git is a pretty good distributed version control system, but its DVCS
frontend is optimized for application development, and lacks many of
the features needed for maintaining live root filesystems. Its file
content control does not scale well to large blobs, and its process
control is limited to a few hook entry points.  It does support some
of the algorithms needed for convergence and congruence (merging and
branching), but only for file content. There are other nits -- like
most VCSs, git assumes it owns the entire workspace, and doesn't play
so well when its workspace is the entire root filesystem tree. It
doesn't manage permissions, special files, and so on. 

At its core, however, git is a nice, lightweight, distributed,
content-addressable nosql database, with native type support for
blobs, lists, and trees.  The API for this backend is published,
stable, and multilingual.  The git plumbing is vastly underutilized by
existing DevOps tools -- this makes sense, because most of these tools
were created prior to the mainstream acceptance of git.

Let's build on that.


