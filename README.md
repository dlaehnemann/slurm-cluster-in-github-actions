# deploy Slurm cluster on GitHub Actions via juju and lxd

## Notes on the experience with this setup

This repository was a trial of setting up a slurm cluster on a GitHub Actions runner using juju and the OSD `slurm` bundle.
It now works, and you can inspect how to use it in [.github/workflows/main.yaml](.github/workflows/main.yaml).
However, I decided against using it further for several reasons:

1. as from what I can see you always have to run every slurm command (`srun`, `sbatch`, `sacct`, `squeue`, etc.) on the `--unit slurmctld/0`, but from within a juju command, so for example:
  ```
  juju run --unit slurmctld/0 'srun --partition=osd-slurmd -vvvv echo "hello world"'
  ```
  Thus, any use of this would need to at least set an alias for every slurm command, so for the above something like (but this then collides with the quoting that is necessary around the `'srun ...'` command, that is necessary to avoid parsing arguments as juju arguments):
  ```
  alias srun='juju run --unit slurmctld/0 srun'
  ```

2. This setup uses quite a lot of disk space in the process, and even after a dedicated cleanup step included in this example workflow here, 3GB of disk space are still blocked (compared to before the cluster setup).

3. While the OSD `slurm` bundle has detailed documentation (see below), there are some nitty gritty details missing, where I had to dig into juju documentation or figure things out by trial-and-error.
   In addition, juju documentation is not very accessible and there [isn't much community docs / usage of `juju`](https://www.reddit.com/r/devops/comments/iuw3po/comment/g5o2sog/?utm_source=share&utm_medium=web3x).
   For example, only very few stackoverflow Q&As and almost no blogs or working example repositories on GitHub about it.
   So understanding how to do things in juju is not straightforward.

4. Versioning of the OSD `slurm` bundle repository and of the respective `juju` charms on charmhub.io is murky, at best.
   So pinning stuff to ensure you have a working version of something is not easily done.

## Resources used

Using Omnivector Solutions Slurm Distribution (OSD) slurm-bundle to set up a slurm cluster in GitHub Actions via juju.
Resources used are:

* [Omnivector Solutions documentation for OSD](https://omnivector-solutions.github.io/osd-documentation/master/installation.html#)
* [OSD `slurm` bundle on `charmhub.io`](https://charmhub.io/slurm)
* [OSD `slurm` bundle repository](https://github.com/omnivector-solutions/slurm-bundles)
* [rpository of individual charms for the OSD `slurm` bundle](https://github.com/omnivector-solutions/slurm-charms)
* [juju documentation](https://juju.is/docs/sdk)

## Possible alternatives

Generally, [`Ansible` seems to be what most people use](https://www.reddit.com/r/devops/comments/iuw3po/anyone_ever_use_juju_for_config_management/), and has much more uptake than `juju`.
So [`Ansible` roles for `slurm`](https://galaxy.ansible.com/search?keywords=slurm&order_by=-relevance&page=1&deprecated=false&type=role) are probably the best place to start looking.
At the time of writing, these three stick out:
1. [StackHPC Ansible slurm setup](https://github.com/stackhpc/ansible-slurm-appliance): it is open, builds on top of [`OpenStack`](https://docs.openstack.org/) and comes with quite a bit of documentation.
2. The sciCORE at the Univerity of Basel has [an Ansible role for slurm setup ](https://github.com/scicore-unibas-ch/ansible-role-slurm), and has nice-looking documentation for it.
