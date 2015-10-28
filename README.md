# gulcin/pgconfeu2015

This repository stores Ansible codes of my [PGConfEU 2015 talk](http://www.postgresql.eu/events/schedule/pgconfeu2015/session/1014-managing-postgresql-with-ansible/) ([Presentation](http://slides.com/apatheticmagpie/managing-postgres-with-ansible)).

This [Ansible playbook](http://docs.ansible.com/ansible/playbooks.html):

  * Provisions [Amazon VPC](https://aws.amazon.com/vpc/) and [Amazon EC2](https://aws.amazon.com/ec2/) instances
  * Installs latest [PostgreSQL](http://postgresql.org) packages
  * Configures a streaming replication with 1 master and 2 standbys

---

## Table of Contents

  * [Dependencies](#dependencies)
    * [Debian/Ubuntu Installation](#debianubuntu-installation)
    * [Mac OS X Installation](#mac-os-x-installation)
  * [Installation](#installation)
  * [Run](#run)
  * [Verification](#verification)
    * [Prereqiusites](#prereqiusites)
    * [Verify](#verify)
  * [Videos](#videos)

---

## Dependencies

[Ansible](http://www.ansible.com) and the [Boto](https://github.com/boto/boto) package is required to run this playbook.

### Debian/Ubuntu Installation

```shell
apt-get install python-dev python-setuptools
easy_install pip
pip install ansible boto
```

### Mac OS X Installation

```shell
sudo easy_install pip
sudo pip install ansible boto
```

---

## Installation

1. Clone this repo

  ```shell
  git clone https://github.com/gulcin/pgconfeu2015.git 
  ```

2. Edit `~/.boto` to provide AWS access credentials

  ```ini
  [Credentials]
  aws_access_key_id = YOUR_ACCESS_KEY_ID
  aws_secret_access_key = YOUR_SECRET_ACCESS_KEY
  ```

3. _[optional]_ Edit `~/.ansible.cfg` to disable host key checks

  ```ini
  [defaults]
  host_key_checking = False
  ```
  
  You can skip this step, however Ansible will first ask you about a confirmation to connect to new created hosts.

---

## Run

You can run this playbook simply by using the `ansible-playbook` command.

```bash
ansible-playbook -i hosts.ini main.yml
```

---

## Verification

We can see details about provisioned AWS EC2 instances by using the [AWS Command Line Interface](https://aws.amazon.com/cli/).

### Prereqiusites

First time installation and configuration of `awscli` is required.

```bash
sudo pip install awscli
```

After installing `awscli` you can configure it and provide your AWS credentials:

```bash
aws configure
```

### Verify

To see created instances with their public IP addresses, you can issue the following command:

```bash
aws ec2 describe-instances --no-paginate --output=text \
    --filters 'Name=instance-state-name,Values=running' \
    --query 'Reservations[].Instances[].[Tags[?Key==`Name`].Value, PublicIpAddress]'\
    | sed '$!N;s/\n/ /' | grep pg | sort -k2
```

You can also check if the replication is working correctly with a scenario like this:

  * Connect to the master instance

    ```bash
    ssh ubuntu@<IP_ADDRESS>
    ```

    * See if PostgreSQL processes are running

      ```bash
      ps axw | grep "postgres:"
      ```

    * Change to the `postgres` system user

      ```bash
      sudo su - postgres
      ```

    * Connect to the `vienna` database

      ```bash
      psql vienna
      ```

      * Create a test table

        ```sql
        CREATE TABLE test (title text);
        ```

      * Insert some data to this table

        ```sql
        INSERT INTO test VALUES ('Test row 1');
        INSERT INTO test VALUES ('Test row 2');
        INSERT INTO test VALUES ('Test row 3');
        ```

  * Connect to standby instances

    ```bash
    ssh ubuntu@<IP_ADDRESS>
    ```

    * See if PostgreSQL processes are running

      ```bash
      ps axw | grep "postgres:"
      ```

    * Change to the `postgres` system user

      ```bash
      sudo su - postgres
      ```

    * Connect to the `vienna` database

      ```bash
      psql vienna
      ```

      * Check the recovery status:

        ```sql
        SELECT is_in_recovery();
        ```

      * Select some data from the test table

        ```sql
        SELECT * FROM test;
        ```

---

## Videos

You can find [Asciinema](http://asciinema.org) videos shown in my [presentation](http://slides.com/apatheticmagpie/managing-postgres-with-ansible) here

  * [Install Ansible on Debian-based Systems](https://asciinema.org/a/dusomo0z73ypofj6h8iv8j8hd)
  * [Playbook: First Run](https://asciinema.org/a/b1xjvmxvnmhv8ooljc1kogzg9)
  * [Playbook: Verify Setup](https://asciinema.org/a/0jdsb9e2pxlf863ppg5ii2otw)
  * [Playbook: New Standby](https://asciinema.org/a/5v7yvcboeh8ih7cya66pf887k)
