ansible all -m setup --tree $HOME/host_facts

Manuall Installation: Elastic 5x in Ubuntu 16

wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list

check:
cat /etc/apt/sources.list.d/elastic-5.x.list

Install :

sudo apt-get update && sudo apt-get install elasticsearch

Note:
Error during installation
Setting up elasticsearch (5.2.0) ...
Couldn't write '1' to 'net-ipv4/icmp_ignore_bogus_error_responses', ignoring: No such file or directory


Running Elasticsearch with systemd edit
To configure Elasticsearch to start automatically when the system boots up, run the following commands:

sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service

Elasticsearch can be started and stopped as follows:


To finalise installation, ensure Elasticsearch starts automatically on boot with:
sudo update-rc.d elasticsearch defaults 95 10 for init.d systems (like the 16.04.1 image on ElasticHosts)
sudo /bin/systemctl daemon-reload && sudo /bin/systemctl enable elasticsearch.service for systemd systems


sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service

Enable journalctl
These commands provide no feedback as to whether Elasticsearch was started successfully or not. Instead, this information will be written in the log files located in /var/log/elasticsearch/.

By default the Elasticsearch service doesn’t log information in the systemd journal. To enable journalctl logging, the --quiet option must be removed from the ExecStart command line in the elasticsearch.service file.

When systemd logging is enabled, the logging information are available using the journalctl commands:

To tail the journal:

sudo journalctl -f
To list journal entries for the elasticsearch service:

sudo journalctl --unit elasticsearch
To list journal entries for the elasticsearch service starting from a given time:

sudo journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"


Server VLAN setup:
https://bayton.org/2016/09/deploying-an-elasticsearch-cluster-on-ubuntu-16-04/

14.04 LTS  you use physical servers, VMs or containers, as long as they each have a dedicated IP address and can communicate on a private (v)LAN.

 servers setup with dedicated IPs and a private network
 
 Server VLAN setup
 3.2. Server VLAN setup
Earlier all three containers were added to a private VLAN, now it can be set up.

First, check the NIC is present with:

ifconfig -a

This should return something similar to the below:

eth0 Link encap:Ethernet HWaddr ee:27:5a:2c:7f:68
inet addr:94.196.69.193 Bcast:5.152.178.255 Mask:255.255.255.0
inet6 addr: fe80::ec27:5aff:fe2c:7f68/64 Scope:Link
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
RX packets:19450 errors:0 dropped:0 overruns:0 frame:0
TX packets:9514 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:218803196 (218.8 MB) TX bytes:1000777 (1.0 MB)

eth1 Link encap:Ethernet HWaddr f2:27:5a:2c:7f:68
BROADCAST MULTICAST MTU:1500 Metric:1
RX packets:9 errors:0 dropped:0 overruns:0 frame:0
TX packets:17 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0
RX bytes:714 (714.0 B) TX bytes:1362 (1.3 KB)
eth1 is the NIC that needs to be edited in this case, and that can be done by running:

sudo vim /etc/network/interfaces (Using LXD, this will likely be /etc/network/interfaces.d/50-cloud-init.cfg)

Now add the new private network interface configuration under the existing eth0:

auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
     address 10.11.12.10
     netmask 255.255.255.0
     network 10.11.12.0
The bold entry above is what needs to be added to all three servers, where address 10.11.12.10 may be any IP range, as long as each server has a different IP on one universal subnet (eg: 10.11.12.10, 10.11.12.11, 10.11.12.12)

Finally, run:

ifup eth1

And now it should be possible to ping all servers in the cluster from each node:

jason@ubuntu:~# ping 10.11.12.11
PING 10.11.12.11 (10.11.12.11) 56(84) bytes of data.
64 bytes from 10.11.12.11: icmp_seq=1 ttl=64 time=0.048 ms
64 bytes from 10.11.12.11: icmp_seq=2 ttl=64 time=0.036 ms

