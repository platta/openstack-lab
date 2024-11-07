# OpenStack Lab

Ansible playbooks for configuring a single-node OpenStack lab.

## OpenStack Notes

`-e _oslodb_setup_nolog=False`
`-e pki_regen_cert=true`

### Storage Configuration

Create an LVM volume called `cinder-volumes`.

```console
pvcreate --metadatasize 2048 /dev/nvme1n1p2
vgcreate cinder-volumes /dev/nvme1n1p2
```

### HAProxy

For monitoring HAProxy, you can use this command:

```console
watch 'echo "show stat" | nc -U /run/haproxy.stat | cut -d "," -f 1,2,5-11,18,24,27,30,36,50,37,56,57,62 | column -s, -t'
```

You might need to install `bsdmainutils` from `apt` if you don't already have
the `column` binary on your system.

`while openssl x509 -noout -text; do :; done < cert-bundle.pem`

## Setup

1. Install and configure Debian how you'd like.
2. You must use LVM and ensure a volume group called `cinder-volumes` exists.
   The volume group should be empty. This is where Cinder will create all of its
   block storage volumes.
3. Set up a python virtual environment (see below)
4. Configure a file called `inventory.yml`, using `inventory.sample.yml` as a
   base.
5. Configure a host variables file in the `host_vars` folder using `SAMPLE.yml`
   as a base. The file name should match the host name you used in your
   inventory file.
6. Prepare Ansible on your control node.
    - `ansible-galaxy collection install -r ansible-collection-requirements.yml`
    - `ansible-galaxy role install -r ansible-role-requirements.yml`
7. Run the role for the pre-OpenStack install setup.
    - `ansible-playbook -i inventory.yml os_pre_role.yml`
8. Connect to your host and follow the instructions for deploying openstack-
   ansible. Bootstrap the necessary Ansible components.
    - `/opt/openstack-ansible/scripts/bootstrap-ansible.sh`
9. Next, run the playbooks in the `/opt/openstack-ansible/playbooks` folder.
   This step will take a while, and once complete you've technically got a
   working OpenStack environment.
    - `openstack-ansible setup-hosts.yml`
    - `openstack-ansible setup-infrastructure.yml`
    - `openstack-ansible setup-openstack.yml`
10. Prepare `clouds.yaml` file (see below)
11. Run the role for the post-OpenStack setup.
    - `ansible-playbook -i inventory.yml os_post_role.yml`

At this point, you should be all set to log in as either the admin or your
specified project user.

On your lab host, check `/etc/openstack_deploy/user_secrets.yml` for the key
`keystone_auth_admin_password`, and that value will be the admin user's
password. Your project user's password will have been specified in your host
vars file.

Most entities are created for you by Ansible, but you will want to create or
import an SSH Keypair for yourself.

## Python Virtual Environment

Connect to the target host as the user that Ansible will use to connect to it.

Install python and pip

```console
sudo apt install python3 python3-pip
```

**Note** You may have to `sudo apt install python3-pip` or something like that
to get `pip` and it might be `python` or `python3`,  `pip` or `pip3`.

Create a venv and install `openstacksdk`

```console
cd ~
python -m venv openstack.cloud
source openstack.cloud/bin/activate
pip install --upgrade pip
pip install openstacksdk
```

**Note** You can type `deactivate` to leave the venv.

In the inventory file, specify the python binary in your venv for the
`ansible_python_interpreter`. For example
`/home/your_user/openstack.cloud/bin/python`.

## Initial setup of stuff once ansible has deployed everything

On the target host as root or via `sudo`.

1. Ensure directory `/etc/openstack` exists.
2. Connect to the utility container.
    - ``lxc-attach -n `lxc-ls -1 | grep utility` ``
3. Copy the contents of `/root/.config/openstack/clouds.yaml`.
4. If you've changed the value of `cloud_name` in your host vars from `default`,
   you'll need to change the name of the entry in `clouds.yaml` to match.
5. `exit` the utility container.
6. Create `/etc/openstack/clouds.yaml` with the copied contents from the utility
   container.

### Get set up

Go into the utility container

```console
lxc-attach -n `lxc-ls -1 | grep utility`
```

Get ready to run openstack commands.

```console
cd /root
source openrc
```

### Create network(s)

Create the public network.

```console
$ openstack network create public \
    --provider-network-type flat \

```

Create the public subnet.

```console
$ openstack subnet create public \
    --network public \
    --subnet-range 192.168.4.0/24
```

### Upload image(s)

Download a debian image.

```console
$
```

Upload it to glance.

```console
$
```

### Create flavor(s)

```console
$
```

### Create a project

### Create a user in the project

### Create a project network

Create network.

Create subnet.

Create router.

Attach router to public network and project network.
