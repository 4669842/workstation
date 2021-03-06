Quickly prepare (development) environment with Ansible.


= Workstation

..is a box with some operating system and personal tools
needeed to do the job.

The basic stuff I expect to see on every Linux machine:

  (browser: all firefox, google-chrome)
  git
  hg
  vim

On Windows the list is same, except vim is replaced with
Far Manager + Colorer. 

== Workstation: Linux, Ubuntu 13.04

  git
  hg
  vim

  google-chrome


= Bootstraping workstations
== ..with Ansible on Linux (Ubuntu 13.04)

Too long/didn't read:

  sudo apt-get install ansible
  git clone git://github.com/techtonik/workstation.git
  ./workstation/ansible.sh

== ..details ==
Feel free to skip ..details it's about  basic concepts.
You may start with example and return when ready.

=== command line tool ===
`ansible` is a command line tool that run stuff on
remote host. Stuff is packed into `modules` (-m), and
fine-tuned (selected) using module arguments (-a).
Default module is `command`, so these two commands
that execute `uname` on all configured hosts are equal:

  ansible '*' -i ansible/hosts -a uname
  ansible all -i ansible/hosts -m command -a uname

To ping all configured hosts, execute `ping` module:

  ansible all -i ansible/hosts -m ping

=== inventory ===
This `hosts` file is called "inventory file" and holds
network description. It is usually located at
/etc/ansible/hosts, but can be anywhere (-i option).

  $ cat ansible/hosts
  localhost 

Inventory file is required - you can't run commands on
arbitrary machine - it should be present in inventory.
This might keep you from mistake, but the real reason
is that everything in ansible is build around server
names and groups in inventory.

=== inventory for playbook ===
It is because the primary function of Ansible is
running playbooks, not single commands. Playbook is a
config file in YAML format that (unlike shell script)
can be automatically parsed, checked and changed with
tools that can handle YAML. Ansible is one such tool.

Playbooks usually contain rules or commands (tasks)
that are executed on *specific* hosts. Host list is a
requires attribute of any playbook - the list can be
hardcoded or sent as parameter. Rules in playbook
often require that specific kinds of servers are
present (database, for example) so the rules are very
tightly interconnected and tied to the structure of
your inventory. It would be rather hard to specify all
those servers each time on the command line.

=== remote and local execution ===
Ansible runs playbooks over network. It takes playbook
from local machine, connects to remote machine through
SSH and executes instructions there.

SSH access is not necessary when you executing
commands on local machine, but you need to specify
`--connection local` from command line explicitly. It
is possible to specify connection in `hosts`.

  $ cat ansible/hosts
  localhost ansible_connection=local

=== documentation ===
Ansible docs are located at:
http://www.ansibleworks.com/docs/

+-- WIP --
Comparison - Chef Cookbooks // Ansible Playbooks:
[ ] reusability
[ ] cross-platformedness
[ ] availability
[ ] dependent books
Questions:
[ ] when you don't know exact package names to
    install 'vim' and 'mysql' on different systems
    (Fedora: `vim-enhanced` and `community-mysql`)
/-- WIP --

== ..example ==
=== 1. install Ansible

- Ubuntu (13.10, Ansible 1.3):

  sudo apt-get install ansible

The 1.3 version is not available for Ubuntu 13.10, so
you need to install it from backports. I install it
from my own ppa:techtonik/ansible

- Fedora (19, Ansible 1.3):

  sudo yum install ansible

=== 2. create hosts configuration file

  $ vim ansible/hosts

The contents of the file for my machine:

  $ cat ansible/hosts
  localhost

=== 3. run Ansible locally

  $ ansible localhost -i ansible/hosts -a uname
  localhost | FAILED => SSH encountered an unknown error during the connection.

This means that Ansible tries to connect through SSH
and it is not working. For local machine SSH is not
necessary:

  $ ansible localhost -i ansible/hosts -c local -a uname
  localhost | success | rc=0 >>
  Linux

Here `localhost` is a mask - you can use '*' or 'all'
to run command on every host from inventory file.
`-a uname` is a parameter to default `command` module
that executes `uname`. `-c local` makes Ansible
execute all commands locally.

To avoid typing `-c local` every time, connection type
for localhost can be specified in `ansible/hosts`:

  $ cat ansible/hosts
  localhost ansible_connection=local

== 4. explore modules

Ansible commands are grouped into modules. For example,
there is a module named `ping` to check connections:

  $ ansible localhost -i ansible/hosts -m ping
  localhost | success >> {
      "changed": false, 
      "ping": "pong"
  }

"changed" value contains Ansible guess if machine state
was changed or not. "ping" is a result specific to ping
module, which is a standard "pong" response.

For a list of Ansible modules see:
http://www.ansibleworks.com/docs/modules.html

=== 4. run playbook

Playbook contains commands to run on specific hosts or
groups of hosts configured in inventory file. To run
playbook, use `ansible-playbook` command. Playbooks
already specify which hosts they need to run commands
on, so you don't need to do this from command line.
Playbook is written in subset of YAML format.

  $ cd ansible

Create playbook:

  $ vim playbook.yml
  $ cat playbook.yml
  ---
  - hosts: localhost
    tasks:
    - name: require git, mercurial and subversion
      apt: pkg=git
      apt: pkg=mercurial
      apt: pkg=subversion

Run playbook:

  $ ansible-playbook -i hosts playbook.yml

This playbook executes `apt` module which checks if a
given package is installed and installs it. Note that
it still needs inventory file to operate.

To avoid passing `-i hosts` every time, it may be
possible to create ansible.cfg configuration file in
playbook directory after [ ] ansible issue #5115 is
fixed.

  $ cd ..
  $ vim ansible/ansible.cfg
  $ cat ansible/ansible.cfg
  [defaults]
  hostfile = ./hosts

And it still won't help, because relative path to hosts
file is not properly expanded too at the moment. Need
another bug to fill after.

http://www.ansibleworks.com/docs/intro_configuration.html

==== playbook syntax is YAML

The '---' separator marks the start of the 'playbook',
' - ' is an list element (like HTML <li> tag), and at
the root level playbook is a list of 'play's. 'play'
is further divided into 'tasks'.

http://www.ansibleworks.com/docs/YAMLSyntax.html
