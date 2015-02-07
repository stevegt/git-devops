git-devops
==========

Git-devops (gdo) is an automated systems administration tool which
uses git's backend plumbing as a distributed nosql database,
replicating chunks of large blobs across multiple machines.


Target Use Cases
----------------

- Automated systems administration
    - Provide an underlying DVCS for managing binaries, large blobs,
      and configuration files for existing DevOps tools, supporting
      diverse algorithms and philosophies
    - Provide a native management algorithm for host management,
      supporting the two major models in use today:
        - Convergence -- take over hosts in unknown state, bring them
          towards a known state over time
        - Congruence -- start with hosts in known state, keep them
          that way as they evolve 
    - Bare-metal installation (with git-devops in PXE miniroot)
- Backups, including archival and offsite
- Media storage, both online and offline


Status and Roadmap
------------------

Prototype in development right now, first production release by
Summer 2015.

I'll push frequently to github as I go along.  Pull requests are
welcome and encouraged.


Design Goals
------------

Conceptually, git-devops is new porcelain on top of the git plumbing.
Normally it is called as 'gdo' rather than 'git devops'.  

- Keep code simple -- one python script and one C shared library
  - Can be dropped into place manually, via 'pip install git-devops',
    or by your method of choice
  - Use python ctypes so script can be translated to any other
    language which supports a native C shared library FFI
- Support a CPAN-like plugin architecture, hosted by github
    - plugins can be written in any language
- Use ssh for all network communications
    - Topology can be star, tree, ad-hoc mesh, or fully connected
      graph
- Support the ability to check in blobs which are larger than local
  free disk space
- Use librabinpoly for slicing large files into small chunks
- Use git's porcelain for merge, commit, and branch management
    - small files, journal, and large file blob index are all here
    - one branch per machine type
    - ordinary git repo
        - safe to clone, prune, pack, gc, fsck, merge, etc.
    - large blobs are sliced, hashed, and stored as chunks in repo
    - Each chunk is tagged
    - Maintain a sparse cache -- using the tags, automatically pull
      only those chunks needed for current operations on target
      machine, then drop tags and gc later
- Use gnupg for release signatures
- Pull only -- simplifies security model and overall design
    - No ssh'ing into other machines to change them; we only ssh into
      other machines to fetch changes for ourselves


Architecture
------------

Gdo manages one repository on each machine, with GIT_DIR set to
/var/git-devops/repo.  No two machines hold the exact same repository
content; they each use the repo as a sparse cache.  It's possible that
no machine holds the entire set of objects.  The entire set of objects
may be larger than the largest disk on any one machine.  

Gdo creates an ssh connection to one or more other machines, and runs
gdo remotely there.  Gdado takes advantage of openssh multiplexing to
re-use each connection for git fetches and pushes.


Wire Protocol
-------------

When gdo opens an ssh connection to another machine, it does so in
multiplex master mode.  The command which the remote user runs is XXX.


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
writing directly over them, or by running another process that does
so.

Git is a pretty good distributed version control system, but its DVCS
frontend is optimized for application development, and lacks many of
the features needed for maintaining live root filesystems. Its file
content control does not scale well to large blobs, and its process
control is limited to a few hook entry points.  It does support some
of the algorithms needed for convergence and congruence (merging and
branching), but only for file content. There are other nits -- like
most VCSs, git's frontend assumes it owns the entire workspace, and
doesn't play so well when its workspace is a subset of the entire root
filesystem tree. It doesn't manage permissions, special files, and so
on. 

However, git's backend plumbing is a lightweight, distributed,
content-addressable nosql database, with native type support for blobs
and trees.  The API for this backend is published, stable, and
multilingual.  The git plumbing is vastly underutilized by existing
DevOps tools -- most of these tools were created prior to the
mainstream acceptance of git.

Let's build on that.


How is "gdo" pronounced?
------------------------

I use both "gee dee oh" and "gee doo" in conversation; both are
equally valid.  Interview candidates and coworkers should not be
derided for using either of these pronuciations.  Project managers can
call it "Bob" to signify that they've actually skimmed this far.


What is GDO an abbreviation of?
-------------------------------

Git DevOps
Git Distributed Objects
Global Distributed Operations
Galactic Dollar Orchestration
Global Dealing Operations
General Delivery Office
Governance Distributed Osomething
Garage Door Opener
[...]
