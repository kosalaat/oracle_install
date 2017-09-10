# How to Use

From the roles directory of your project folder clone in to this repository.

```
~/project/roles/ # git clone https://github.com/kosalaat/oracle_install.git

```

Once the repository is cloned, you're almost good to go. Most of the default vaules are functional, and they're listed under the defaults/main.yml.

There are two parameters which need to be customized to suite your environment.

1. oracle_installer_path 

This is the path to the installer archives, that's downloaded from Oracle.

2. oracle_images

This is a array of all the available oracle images listed. Based on the version specified in the array, the correct image will be picked for installation.

```
oracle_images:
    - { image: "{{ oracle_installer_path }}/linux.x64_11gR2_database_1of2.zip", version: "11.2.0.1" }
    - { image: "{{ oracle_installer_path }}/linux.x64_11gR2_database_2of2.zip", version: "11.2.0.1" } 
    - { image: "{{ oracle_installer_path }}/linuxx64_12201_database.zip", version: "12.2.0.1" }
```

# Ansible Playbook

For a typical installation you can follow the following Syntax. 

The following playbook can be used to install Oracle 11gR2 instance with SID "demo",  
```
---

- name: install oracle
  hosts: "{{ host_group }}"
  become: yes
  become_method: sudo
  roles:
    - oracle_install
  vars:
    oracle_edition: EE
    oracle_version: 11.2.0.1
    install_mode: INSTALL_DB_AND_CONFIG
    oracle_db_name: demo
```

However, if the requirement is to install only the software binaries and not to create the database....

NOTE: We're specifying oracle 12c in the case, but oracle 11g would work the same way.

```
---

- name: install oracle
  hosts: "{{ host_group }}"
  become: yes
  become_method: sudo
  roles:
    - oracle_install
  vars:
    oracle_edition: EE
    oracle_version: 12.2.0.1
    install_mode: INSTALL_DB_SWONLY
```


# Pre-requisites for the playbook

The playbook will ensure the paths for the binaries and data managed the way as needed by a standard installation. Playbook will assume, a volume group (default: oravg) specified by the variable oracle_vg, if it does not exisit role will try to create a oravg on the disk drive spcified by oracle_pvs (default: /dev/sdb). However, you have the option to specify an exisiting Volume Group, which however need to have enough free capacapacity to create the logical volumes.

As a base line, this playbook wos successfully tested with many RHEL variants (6/7), with two disks of 10GB each for Oracle 11g. 

The oracle binaries are restored to /tmp/oracle as specified in the defaults/main.yml, which can be overridden, however, for Oracle 12c will require more than 10GB for the rool volume group, as the Binary is a single file which consume more space when copying and unarchiving. Therefore for 12c you will need more capacity than 10GB for the root. I had success with 15GB.