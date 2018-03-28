ansible-role-slurm
------------------
[![GitPitch](https://gitpitch.com/assets/badge.svg)](https://gitpitch.com/CSCfi/ansible-role-slurm/master) 
[![Build Status](https://travis-ci.org/CSCfi/ansible-role-slurm.svg?branch=master)](https://travis-ci.org/CSCfi/ansible-role-slurm)

# Creates a SLURM cluster

Tested with SLURM versions:
 - 14.11.x
 - 15.08.x
 - 16.05.x
 - 17.02.x
 - 17.11.x

Tested with these Linux distributions:
 - CentOS 6
  - Only 14.11.x
 - CentOS 7
  - 15.08.x (travis ci automatic testing)
  - 16.05.x (travis ci automatic testing)
  - 17.02.x (travis ci automatic testing)
  - 17.11.x (travis ci automatic testing)

## Dependencies

 - https://github.com/jabl/ansible-role-pam
  - configure PAM - limit access to the compute nodes
 - https://github.com/CSCfi/ansible-role-nhc
  - Configure Node Health Checker

## How-To

### Initial playbook configuration

*Playbook variables:*

All variables should be defined in defaults/main.yml

All nodes run munge. Nodes which are part of the slurm\_compute host
group will additionally run slurmd. Nodes which are part of the
slurm\_service host group will additionally runs slurmctld and
slurmdbd (unless {{ slurm_accounting_storage_host }} is not the same as {{ slurm_service_node }}). Nodes which are in neither of these two host groups are 
assumed to be submit hosts.

You also need to add a mysql\_slurm_password: "PASSWORD" string
somewhere. This will be used to set a password for the slurm mysql
user. See http://docs.ansible.com/ansible/playbooks_vault.html

To add your own nodes and queues define the slurm_nodelist and slurm_partitionlist lists.

It is possible to run the slurmdbd on a different host than the slurmctld by changing the slurm_accounting_storage_host variable.

It is also possible to setup a backup slurm controller by defining slurm_backup_controller variable. Please read the [SLURM HA documentation](https://slurm.schedmd.com/quickstart_admin.html#HA). For example you'll need a shared directory (for example NFS) available on both the slurm_service_node and slurm_backup_controller.

SLURM 16.05 can be gotten from the FGCI yum repo by setting:
<pre>
fgci_slurmrepo_version: "fgcislurm1605"
</pre>

### Implementation

A playbook that uses this role: https://github.com/CSCfi/fgci-ansible

Or you can check out the [tests/test.yml](tests/test.yml) in this repo.

Example Playbook
----------------

<pre>
    - hosts: compute
      strategy: free
      roles:
         - { role: ansible-role-pam }
         - { role: ansible-role-nhc }
         - { role: ansible-role-slurm }

    - hosts: install
      roles:
         - { role: ansible-role-slurm }
</pre>

### Known Issues

 - This role used to be able to build slurm rpms, distribute them and install them. The last tag/release that had this feature was v1.5.0
 - Setting up a shared directory á la NFS for running a SLURM in HA is out of scope for this role. There are many [NFS server roles](https://github.com/CSCfi/ansible-role-nfs) and [Mount Filesystem roles](https://github.com/CSCfi/ansible-role-nfs_mount) roles out there.

### Testing and contributions

Testing is done with [Travis](.travis.yml). New SLURM release can be tested after the RPMs are built and available in the FGCI repo. After that one needs to add a new tests/test1702.yml and a new IMAGE_BUILD_PLATFORM env in .travis.yml.

 - PRs to master
 - if possible make sure that the new feature is also tested
 - strive for backwards compatibility

**Adding testing of a new SLURM release**

Using 17.11 as an example

 - Get CSC to build new rpms and put them in a new yum repo
 - New branch in ansible-role-slurm with the following changes/additions:
   - IMAGE_BUILD_PLATFORM=fgcislurm1711 in .travis.yml env:
   - tests/test1711.yml with fgci_slurmrepo_version: "fgcislurm1711"
   - tests/fgcislurm1711 directory a symlink of tests/fgcislurm1508
 - Then make changes if needed to the role that does not break older SLURM versions

# Authors / Contributors:

 - Marco Passerini (original author)
 - https://github.com/martbhell
 - https://github.com/tiggi
 - https://github.com/A1ve5
 - https://github.com/jabl
 - https://github.com/mhakala
