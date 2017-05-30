---
layout: post
title:  "Ansible - creating a list of IPs and ports"
date:   2017-05-30 05:14:49 +0200
categories: devops ansible
---
## Ansible tip - creating a list of IPs:Ports with Jinja2 loops

While working on setting up an  `ansible-playbook` for a Kafka cluster, a few challenges came up. In this, setting up the `zookeeper.connect` value in the Kafka server configuration file: `"{{ kafka_base_config_dir }}/server.properties"`

The line should look something like this:

{% raw %}
    zookeeper.connect=kafkaserver1:2181,kafkaserver2:2181,kafkaserver3:2181
{% endraw %}

## jinja2 loop

The simple loop will look something like this:

{% raw %}
    {% for host in groups['kafkaservers'] %}
      {{ host }}:2181,
    {% endfor %}
{% endraw %}

Thy issue here is that you will get redundant comma at the end of the line like so:

{% raw %}
    zookeeper.connect=kafkaserver1:2181,kafkaserver2:2181,kafkaserver3:2181,
{% endraw %}

In order to resolve this we will use `if not loop.last`

{% raw %}
    {% for host in groups['kafkaservers'] %}
      {{ host }}:2181
        {% if not loop.last %}
           ,
        {% endif %}
    {% endfor %}
{% endraw %}

## Ansible example task

{% raw %}
    - name: Set zookeeper.connect in server.properties
      lineinfile:
        path: "{{ kafka_server_config_file_path }}"
        regexp: "^zookeeper.connect"
        line: "zookeeper.connect={% for host in groups['kafkaservers'] %}{{ host }}:2181{% if not loop.last %},{% endif %}{% endfor %}"
      become: yes
{% endraw %}