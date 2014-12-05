git-devops
==========

Git-devops is an automated systems administration tool which uses
git's backend plumbing as a distributed nosql database for blob
striped storage and transport.


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
        - GIT_DIR = /etc/git-devops/repo
        - GIT_WORK_TREE = /
- Use our own methods for large file storage 
    - large files are sliced, hashed, and stored as chunks in
      /var/cache/git-devops
    - git-devops manages and transfers these chunks itself 
    - Maintain a sparse cache -- automatically pull only those chunks
      needed for current operations on target machine, then clean up
      afterwards
- Use gnupg for release signatures
- Pull only -- simplifies security model and overall design
    - No ssh'ing into other machines to change them; we only ssh into
      other machines to fetch changes for ourselves


Status and Roadmap
------------------

Prototype in development right now, first production release by
February 2015.

I'll push frequently to github as I go along.  Pull requests are
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
most VCSs, git's frontend assumes it owns the entire workspace, and
doesn't play so well when its workspace is the entire root filesystem
tree. It doesn't manage permissions, special files, and so on. 

However, git's backend plumbing is a lightweight, distributed,
content-addressable nosql database, with native type support for blobs
and trees (nested lists, actually).  The API for this backend is
published, stable, and multilingual.  The git plumbing is vastly
underutilized by existing DevOps tools -- most of these tools were
created prior to the mainstream acceptance of git.

Let's build on that.


