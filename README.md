<img alt="Vagrant" src="https://img.shields.io/badge/vagrant%20-%231563FF.svg?&style=for-the-badge&logo=vagrant&logoColor=white"/>

# Cisco Catalyst 8000V Vagrant box

A procedure for creating a Cisco Catalyst 8000V Vagrant box for the [libvirt](https://libvirt.org) provider.

## Prerequisites

  * [Git](https://git-scm.com)
  * [Python](https://www.python.org)
  * [Ansible](https://docs.ansible.com/ansible/latest/index.html) >= 2.7
  * [libvirt](https://libvirt.org) with client tools
  * [QEMU](https://www.qemu.org)
  * [Expect](https://en.wikipedia.org/wiki/Expect)
  * [Telnet](https://en.wikipedia.org/wiki/Telnet)
  * [Vagrant](https://www.vagrantup.com) >= 2.2.19
  * [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)

## Steps

0\. Verify the prerequisite tools are installed.

<pre>
$ <b>which git python ansible libvirtd virsh qemu-system-x86_64 expect telnet vagrant</b>
$ <b>vagrant plugin list</b>
vagrant-libvirt (0.9.0, global)
</pre>

1\. Log in and download the Cisco Catalyst 8000V Edge Software software (Cupertino-17.8.1a or later) from your [Cisco](https://software.cisco.com/download/home/286327102/type) account. Save the file to your `Downloads` directory.

2\. Copy (and rename) the disk image file to the `/var/lib/libvirt/images` directory.

<pre>
$ <b>sudo cp $HOME/Downloads/c8000v-universalk9_8G_serial.17.08.01a.qcow2 /var/lib/libvirt/images/cisco-cat8kv.qcow2</b>
</pre>

3\. Modify the file ownership and permissions. Note the owner may differ between Linux distributions.

> Ubuntu 18.04

<pre>
$ <b>sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/cisco-cat8kv.qcow2</b>
$ <b>sudo chmod u+x /var/lib/libvirt/images/cisco-cat8kv.qcow2</b>
</pre>

> Arch Linux

<pre>
$ <b>sudo chown nobody:kvm /var/lib/libvirt/images/cisco-cat8kv.qcow2</b>
$ <b>sudo chmod u+x /var/lib/libvirt/images/cisco-cat8kv.qcow2</b>
</pre>

4\. Create the `boxes` directory.

<pre>
$ <b>mkdir -p $HOME/boxes</b>
</pre>

5\. Start the `vagrant-libvirt` network (if not already started).

<pre>
$ <b>virsh -c qemu:///system net-list</b>
$ <b>virsh -c qemu:///system net-start vagrant-libvirt</b>
</pre>

6\. Clone this GitHub repo and _cd_ into the directory.

<pre>
$ <b>git clone https://github.com/mweisel/cisco-catalyst-8kv-vagrant-libvirt</b>
$ <b>cd cisco-catalyst-8kv-vagrant-libvirt</b>
</pre>

7\. Run the Ansible playbook.

<pre>
$ <b>ansible-playbook main.yml</b>
</pre>

8\. Copy (and rename) the Vagrant box artifact to the `boxes` directory.

<pre>
$ <b>cp cisco-cat8kv.box $HOME/boxes/cisco-cat8000v-17.08.01a.box</b>
</pre>

9\. Copy the box metadata file to the `boxes` directory.

<pre>
$ <b>cp ./files/cisco-cat8000v.json $HOME/boxes/</b>
</pre>

10\. Change the current working directory to `boxes`.

<pre>
$ <b>cd $HOME/boxes</b>
</pre>

11\. Substitute the `HOME` placeholder string in the box metadata file.

<pre>
$ <b>awk '/url/{gsub(/^ */,"");print}' cisco-cat8000v.json</b>
"url": "file://<b>HOME</b>/boxes/cisco-cat8000v-VER.box"

$ <b>sed -i "s|HOME|${HOME}|" cisco-cat8000v.json</b>

$ <b>awk '/url/{gsub(/^ */,"");print}' cisco-cat8000v.json</b>
"url": "file://<b>/home/marc</b>/boxes/cisco-cat8000v-VER.box"
</pre>

12\. Also, substitute the `VER` placeholder string with the Cisco IOS XE version you're using.

<pre>
$ <b>awk '/VER/{gsub(/^ */,"");print}' cisco-cat8000v.json</b>
"version": "<b>VER</b>",
"url": "file:///home/marc/boxes/cisco-cat8000v-<b>VER</b>.box"

$ <b>sed -i 's/VER/17.08.01a/g' cisco-cat8000v.json</b>

$ <b>awk '/\&lt;version\&gt;|url/{gsub(/^ */,"");print}' cisco-cat8000v.json</b>
"version": "<b>17.08.01a</b>",
"url": "file:///home/marc/boxes/cisco-cat8000v-<b>17.08.01a</b>.box"
</pre>

13\. Add the Vagrant box to the local inventory.

<pre>
$ <b>vagrant box add --box-version 17.08.01a cisco-cat8000v.json</b>
</pre>

## Debug

To view the telnet session output for the `expect` task:

<pre>
$ <b>tail -f ~/cat8kv-console.explog</b>
</pre>

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details
