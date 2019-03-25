# IPTables

Configure and install iptables.
this role will also install iptables-persistent to ensure the rules are applied across reboots

>Note: This role will only run if the file `{{ iptables_ipv4_rules_file }}` does not exist

>Note: iptables rules apply immediately

## Requirements

N/A

## Role Variables

This role uses a dictionary to determine how to configure iptables rules.

| Name | Description | Default | Required |
|:----:|:-----------:|:-------:|:-------:|
|port_set|**list**; port set to use when calling the role|default_port_list|yes|
|iptables_default_rules|**string**; will insert default-rules|none|no|
|iptablesForce|**boolean**; run the role by bypassing the condition check|False|yes|

The following condition is defined in `main.yml` of `iptables`:

```yaml
when: not iptablesRules_file.stat.exists or iptablesForce
```

The following condition is defined in the `baseconfig-playbook`:

```yaml
when: iptables_default_rules is defined and iptables_default_rules == 'true'
```

> **port_set** is the variable that indicates which set of ports we are going to apply.
for example, `port_set: '{{ default_ports }}'` refers to another variable called `default_ports`

The default yaml for this role contains pre-defined port sets you can specify using the below parameter.  
This will change the 'with_items' directive passed to the task.

```yaml
with_items:
    - "{{ port_set }}"
```

Example of the expected data-structure:

```yaml
docker_swarm_manager_ports:
  - port: 2377
    action: append
    chain: INPUT
    protocol: tcp
    destination_port: 2377
    source_port: 2377
    state: present
    comment: 'docker-swarm-management'
```

>*The dictionary must contain all key-value pairs*  
>Pay attention to source_port and destination_port in regards to which chain is used

## Dependencies


N/A

## Example Playbook

In order to decide when to run we're using `inventory_hostname` which should be available to us when using an inventory file.
`ansible_host` will get populated only when `gather_facts: true`.

The role itself can be called from any Playbook like so:

```yaml
  - role: iptables
    vars:
      port_set: "{{ docker_swarm_manager_ports }}"
```

Example of `iptables-playbook.yml`

```yaml
  - hosts: all
    gather_facts: true
    become: true
    roles:
      - role: iptables
        vars:
          port_set: "{{ default_ports }}"
        tags: [iptables.default]
      - role: iptables
        vars:
          port_set: "{{ docker_swarm_manager_ports }}"
        when: inventory_hostname in groups['swarm-manager']
        tags: [iptables.dockerswarm-mgmt]
```
