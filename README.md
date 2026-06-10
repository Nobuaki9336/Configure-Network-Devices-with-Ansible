# Cisco Network Configuration Automation with Ansible

This repository demonstrates how to orchestrate and automate Cisco IOS-XE network devices using **Ansible Playbooks**. It showcases structured configuration management across different device groups (CPE and Core), focusing on Access Control Lists (ACLs), dynamic interface provisioning, and global SNMP settings.

## Network Topology

The Ansible automation playbooks are executed from a Linux Student Workstation (Control Node) and tested against a 3-router Cisco CSR1000v topology:

<img src="network_topology.png" alt="Network Topology" width="900">

### Automation Environment Details:
* **Ansible Control Node (Linux):** 10.254.0.10/24 (`ens192`)
* **R1 (CPE Router):** 10.254.0.1/24 (`Gi4`)
* **R2 (Core Router):** 10.254.0.2/24 (`Gi4`)
* **R3 (CPE Router):** 10.254.0.3/24 (`Gi4`)

---

## Inventory & Variable Structure (`inventoryfile`)

The project utilizes a structured INI inventory file with parent/child groups and variable inheritance to drive dynamic configurations:

* **Groups:** `[cpe]` (R1, R3) and `[core]` (R2) nested under a parent `[usa]` group.
* **Host Variables:** `loopback_id` is assigned per host to dynamically generate unique IP addresses.
* **Group Variables:** Custom variables like `location` are defined at the group level (`California` vs `NYC`), demonstrating multi-site scalability.

---

## Playbooks Overview

### 1. Atomic ACL Management (`ACL_config.yml`)
Demonstrates security baseline enforcement. It enters the `ip access-list extended INBOUND` context and utilizes `replace: block` with `match: exact`. This ensures the ACL on the router matches the playbook exactly, safely replacing the old block without compounding duplicate rules.

### 2. Dynamic Interface Provisioning (`manage_interface.yml`)
Showcases data-driven automation by utilizing Jinja2 templating. It configures `interface Loopback0` and dynamically injects the IP address (`172.16.1.{{ loopback_id }}`) based on the unique host variable defined in the inventory file.

### 3. Global SNMP Standardization (`snmp_config.yml`)
A straightforward configuration playbook used to standardize corporate network management settings (Community strings, Location, and Contact fields) globally across all active nodes simultaneously.

---

## How to Run

Execute any of the playbooks against the defined inventory file using the standard `ansible-playbook` command:

```bash
# Example: Deploying the SNMP configuration
ansible-playbook -i inventoryfile snmp_config.yml

# Example: Deploying dynamic Loopbacks
ansible-playbook -i inventoryfile manage_interface.yml
