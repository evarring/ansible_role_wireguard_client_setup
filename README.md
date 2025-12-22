# ansible_role_wireguard_client_setup

An Ansible role to install and configure a WireGuard client using NetworkManager.

## Features

- Installs required packages (`NetworkManager`, `wireguard-tools`).
- Loads the WireGuard kernel module.
- Generates/imports a NetworkManager WireGuard connection from a template.
- Controls connection state (up/down) and autoconnect.

**Supported tags**: `installation`, `config`, `up`, `down`, `always`

## Requirements

- Ansible >= 2.18.6 (see role `meta/main.yml`).
- The `community.general` collection (for `nmcli` and `modprobe` modules). Install with:

```bash
ansible-galaxy collection install community.general
```

- Target systems using NetworkManager (the role uses `nmcli` and NetworkManager system-connections).

## Role variables

All variables listed below are expected to be provided by the playbook or inventory (the role asserts their presence). Example variable names and purpose:

- `wireguard_client_setup_connection_name` (string) — name of the NetworkManager connection (used for filename and `nmcli` operations).
- `wireguard_client_setup_endpoint` (string) — peer endpoint, e.g. `vpn.example.com:51800`.
- `wireguard_client_setup_allowedips` (string) — comma-separated allowed IPs, e.g. `10.1.1.0/24,10.2.2.0/24`.
- `wireguard_client_setup_persistentkeepalive` (string|int) — persistent keepalive seconds; defaults to `25` in the template if not provided.
- `wireguard_client_setup_dns` (string) — DNS server to set in the WireGuard interface (e.g. `10.1.2.1`).
- `wireguard_client_setup_address` (string) — client IP address with prefix, e.g. `10.1.2.23/32`.
- `wireguard_client_setup_privatekey` (string) — WireGuard private key for the client.
- `wireguard_client_setup_publickey` (string) — peer public key.
- `wireguard_client_setup_presharedkey` (string) — optional preshared key for the peer.
- `wireguard_client_setup_autoconnect` (boolean) — whether NetworkManager should autoconnect the connection (default: `false`).

Default values are provided in `defaults/main.yml` as commented examples — you must set real values in your playbook or host vars.

## Files

- Template: `templates/wireguardprofile.conf.j2` — the WireGuard profile used to import the connection into NetworkManager.
- Tasks: `tasks/main.yml` — installs packages, loads kernel module, imports connection, sets autoconnect and brings the connection up/down.
- Meta: `meta/main.yml` — role metadata and minimum Ansible version.

## Example playbook

```yaml
---
- name: Configure WireGuard client on hosts
  hosts: vpn_clients
  become: true
  vars:
    wireguard_client_setup_connection_name: mywg0
    wireguard_client_setup_endpoint: vpn.example.com:51820
    wireguard_client_setup_allowedips: 10.0.0.0/24
    wireguard_client_setup_dns: 10.0.0.1
    wireguard_client_setup_address: 10.0.0.10/32
    wireguard_client_setup_privatekey: <your-client-private-key>
    wireguard_client_setup_publickey: <peer-public-key>
    wireguard_client_setup_presharedkey: <optional-preshared-key>
    wireguard_client_setup_autoconnect: true

  roles:
    - role: ansible_role_wireguard_client_setup
      tags: ['installation','config']
```

To bring the connection up or down via tags:

```bash
ansible-playbook -i inventory.yml playbook.yml --limit vpn_clients --tags up
ansible-playbook -i inventory.yml playbook.yml --limit vpn_clients --tags down
```

## Usage notes

- The role asserts all required variables at runtime; missing variables will fail the run with a clear message.
- The role imports a `.conf` file into NetworkManager under `/etc/NetworkManager/system-connections/`. The temporary `.conf` file is created in the remote user's home (uses `ansible_user`).
- If you re-run the role and the connection already exists, the import step is skipped.

## Dependencies

- No role dependencies, but requires `community.general` collection and NetworkManager on remote hosts.

## Testing

- You can test by running the role in a playbook with the `--tags` filter. Example:

```bash
ansible-playbook configure_wireguard_client.yml --limit myhost --tags installation,config
```
