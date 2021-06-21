<h4 align="center">An Ansible role for creating users and groups.</h4>

<p align="center">
    <a href="https://github.com/totaldebug/ansible-role-wireguard/commits/master">
    <img src="https://img.shields.io/github/last-commit/totaldebug/ansible-role-wireguard.svg?style=flat-square&logo=github&logoColor=white"
         alt="GitHub last commit">
    <a href="https://github.com/totaldebug/ansible-role-wireguard/issues">
    <img src="https://img.shields.io/github/issues-raw/totaldebug/ansible-role-wireguard.svg?style=flat-square&logo=github&logoColor=white"
         alt="GitHub issues">
    <a href="https://github.com/totaldebug/ansible-role-wireguard/pulls">
    <img src="https://img.shields.io/github/issues-pr-raw/totaldebug/ansible-role-wireguard.svg?style=flat-square&logo=github&logoColor=white"
         alt="GitHub pull requests">
</p>

<p align="center">
  <a href="#configuration">Configuration</a> •
  <a href="#features">Features</a> •
  <a href="#contributing">Contributing</a> •
  <a href="#author">Author</a> •
  <a href="#support">Support</a> •
  <a href="#donate">Donate</a> •
  <a href="#license">License</a>
</p>

---

## About

<table>
<tr>
<td>

**ansible-role-wireguard** is a **high-quality** _Ansible Role_ that sets up **Wireguard VPNs** on your ansible clients.

</td>
</tr>
</table>

## Configuration

### Install

```shell
ansible-galaxy install totaldebug.wireguard
```

### Role Variables

A lot of roles setup the tunnel by using `wg-quick`. It is fine in most cases,
but sometimes some automatic behiaviors of `wg-quick` are not wanted. This role
allows to setup a tunnel by using `wg-quick`, `systemd-networkd` or just
generate a wireguard configuration that can be loaded by `wg`.

#### Dependencies

For distributions that are not providing packages for wireguard in their
official repositories, correct repositories need to be setup before using this
role.

#### How-To

Declare the wireguard profiles in `wireguard_profiles`

```yaml
wireguard_profiles:
  # Tunnel interface name
  tunnel_name:
    # How to setup the tunnel
    # Possible values are:
    #   wg-quick: using wg-quick
    #   systemd-networkd: generate a .netdev file to create the tunnel
    #   wireguard-conf: generate a wireguard configuration that can be used with `wg`
    method:

    # MTU variable, in bytes. Used by wg-quick and systemd-networkd. Optional
    mtu:

    # Listen port. Optional
    listen_port:

    private_key:
    fwmark:

    # String to dump at the end of the .netdev or configuration file, depending
    # on the method. Can be used to add additional options.
    additional_opts:

    ## Used by wg-quick only ##

    # Enable or not the tunnel
    enable: True
    # Addresses to set on the tunnel interface. Optional
    addresses: []
    # DNS addresses to use. Optional
    dns: []
    # Route table to use. Optional
    table:
    # Post up script. Optional
    post_up:
    # Post down script. Optional
    post_down:

    ###########################

    peers:
        # Description of the peer. Used to comment the config only. Optional
      - comment:
        public_key:
        preshared_key:
        allowed_ips: []
        # Endpoint address
        endpoint:
        persistent_keepalive:
```

The `wg-quick` method will setup the tunnel, peers and addresses. If
`systemd-networkd` is used, only a `.netdev` is created, and the user is
free to create their own `.network` to attach addresses on it. `wireguard-conf`
will just generate a configuration in `/etc/wireguard/` than can be used by
`wg`.


#### Example Playbook 

```yaml

- hosts: wireguard_servers
  vars:
    wg-test0:
      method: "wg-quick"
      enable: True
      listen_port: 51820

      private_key: "0FZwbRzUF1ZG2i4hqhr1+oJJAQ3NTJDZDhpX3c1Qz1g="

      addresses:
        - "192.0.2.0/31"
        - "2001:db8::/127"
      dns:
        - "192.0.2.1"

      peers:
        - comment: "Some client"
          public_key: "yEvY7Jm8hgWLE64ocDMpwvcE3MH27xac6u55I2R2tik="
          allowed_ips:
            - "192.0.2.1"
            - "2001:db8::1"
          endpoint: "203.0.113.5"
          persistent_keepalive: 25

    wg-test1:
      method: "wireguard-conf"
      enable: False
      listen_port: 51821

      private_key: "0FZwbRzUF1ZG2i4hqhr1+oJJAQ3NTJDZDhpX3c1Qz1g="

      peers:
        - comment: "Another client"
          public_key: "WLE64ocDMpwvcyEvY7Jm8hgE3MH27xac6u55I2R2tik="
          allowed_ips:
            - "192.0.2.3"
          endpoint: "203.0.113.9"

  roles:
    - role: totaldebug.wireguard
```

And add the following line in `/etc/network/interfaces` to setup the second
interface.

```
auto wg-test1
iface wg-test1 inet static
        address 192.0.2.2
        netmask 255.255.255.254
        pre-up ip link add $IFACE type wireguard
        pre-up wg setconf $IFACE /etc/wireguard/$IFACE
        post-down ip link del $IFACE
iface wg-test1 inet6 static
        address 2001:db8::2
        netmask 127
```

## Features

|                            |         🔰         |
| -------------------------- | :----------------: |
| Create users          |         ✔️         |
| Delete Users         |         ✔️         |
| Create Groups    |         ✔️         |


## Contributing

Got **something interesting** you'd like to **share**? Learn about [contributing](https://github.com/totaldebug/.github/blob/main/.github/CONTRIBUTING.md).

### Versioning

This project follows semantic versioning.

In the context of semantic versioning, consider the role contract to be defined by the role variables.

- Breaking Changes or changes that require user intervention will increase the major version. This includes changing the default value of a role variable.
- Changes that do not require user intervention, but add new features, will increase the minor version.
- Bug fixes will increase the patch version.

## Author

| [![TotalDebug](https://totaldebug.uk/assets/images/logo.png)](https://linkedin.com/in/marksie1988) |
|:--:|
| **marksie1988 (Steven Marks)** |

## Support

Reach out to me at one of the following places:

- via [Discord](https://discord.gg/6fmekudc8Q)
- Raise an issue in GitHub

## Donate

Please consider supporting this project by sponsoring, or just donating a little via [our sponsor page](https://github.com/sponsors/marksie1988)

## License

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-orange.svg?style=flat-square)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

- Copyright © [Total Debug](https://totaldebug.uk "Total Debug").