---
layout: post
title:  "Ansible - Finding the server size from Dynamic Inventory variables"
date:   2017-08-30 05:14:49 +0200
categories: devops ansible
---
## Ansible tip - finding the server size from Dynamic Inventory variables

### Prerequisites

The tip in this post assumes you are working with Ansible AWS auto-discovery
See the documentation [here](http://docs.ansible.com/ansible/latest/intro_dynamic_inventory.html)

### Scenario

In the case you would like to "discover" what is the size of the server that ansible is working on in the playbook, and accordingly set a variable you should do the following:

Add to your tasks in the role a `set_fact`


{% raw %}
    - name: set_fact ec2 server type
      set_fact:
        ec2_server_type: "{{ vars.hostvars[ansible_host].ec2_instance_type | replace('.','_') }}"
{% endraw %}

The above finds the current `ansible_host`'s `ec2_instance_type`

Note the `replace` at the end 

This comes to replace the *dot* with an *underscore* for better searching of dictionary variables - see next post 
