# vlan-plumber

A distinctly sysadmin-grade script to generate switch configuration.

## Usage

<pre>
usage: plumb [-h] --from FROM_LIST --to TO_LIST --vlan VLAN [--cdp-cache PATH]

Work out what configuration is required to get a VLAN from one place to
another, based on CDP data. Michael Fincham &lt;michael.fincham@catalyst.net.nz&gt;.

arguments:
  -h, --help        show this help message and exit
  --from FROM_LIST  port or device definitions to plumb from
  --to TO_LIST      port or device definitions to plumb to
  --vlan VLAN       VLAN ID to use
  --cdp-cache PATH  file where CDP cache information is stored, defaults to
                    'cdp-cache'

`from' and `to' should be given in one of these formats:

  device,port,tagged or untagged,platform

  or

  device

Multiple `from' and `to' specifications may be given, separated by
spaces. If only `device' is given, it is assumed to be a Cisco device
and no port need be configured.

For example:

  --from="tor10-20-30,GigabitEthernet1/20,untagged,cisco" \
  --to="colo-edge-firewall1 colo-edge-firewall2" \
  --vlan=123

Will attempt to build a VLAN from the switch called `tor10-20-30',
untagged on GigabitEthernet1/20 to both colocation firewalls.

`cisco' and `ubuntu' are currently the only supported platforms.

The `cdp-cache' file may be generated using RANCID:

  clogin -c "show cdp ne d | inc (---|e ID|Port|Platform)" router [router...] > cdp-cache

Future versions will use SNMP to discover this, and maintain a local
cache automatically.
</pre>

## Example output

<pre>
$ ./plumb \
>   --from="tor10-20-30,GigabitEthernet1/20,untagged,cisco" \
>   --to="colo-edge-firewall1 colo-edge-firewall2" \
>   --vlan=123
>   --cdp-cache=example-cdp-cache
Plumbing VLAN 123 from tor10-20-30 gigabitethernet1/20 (untagged) to colo-edge-firewall1 (tagged)...

Found 1 shortest path:
tor10-20-30 -> core40-50-60 -> colo-edge-firewall1

Plumbing VLAN 123 from tor10-20-30 gigabitethernet1/20 (untagged) to colo-edge-firewall2 (tagged)...

Found 1 shortest path:
tor10-20-30 -> core40-50-60 -> colo-edge-firewall2

Configurations to apply:

colo-edge-firewall1
-------------------
auto eth0.123
iface eth0.123 inet static
    address xxx.xxx.xxx.xxx
    netmask yyy.yyy.yyy.yyy
    description Insert description of VLAN here

colo-edge-firewall2
-------------------
auto eth0.123
iface eth0.123 inet static
    address xxx.xxx.xxx.xxx
    netmask yyy.yyy.yyy.yyy
    description Insert description of VLAN here

core40-50-60
------------
interface gigabitethernet1/0/26
  switchport trunk allowed vlan add 123
interface tengigabitethernet1/51
  switchport trunk allowed vlan add 123
interface gigabitethernet1/0/25
  switchport trunk allowed vlan add 123

tor10-20-30
-----------
interface tengigabitethernet1/51
  switchport trunk allowed vlan add 123
interface gigabitethernet1/20
  switchport access vlan 123
  switchport mode access
  spanning-tree portfast
  description Insert description of VLAN here
</pre>
