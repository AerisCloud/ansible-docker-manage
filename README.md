Docker Manage
=============

Manager networks, containers and volume files purely through inventory.

Requirements
------------

Docker running somewhere.

Role Variables
--------------

```yaml
# a list of networks to setup on the node, see http://docs.ansible.com/ansible/docker_network_module.html for options
docker_networks:
  - name: custom_net
    driver: overlay

# a list of containers to run on the node, for example here we start prometheus and grafana
docker_containers:
  # all the options from the docker_container module (http://docs.ansible.com/ansible/docker_container_module.html)
  # can be defined here, the only extra one is "sync_files" that will invoke the copy module with the proper options
  # before starting the container, "sync_templates" does the thing but runs the template module
  - name: prometheus
    image: "prom/prometheus"
    volumes:
      - "/etc/prometheus.yml:/etc/prometheus/prometheus.yml"
    sync_files:
      - src: prometheus.yml
        dest: /etc/prometheus.yml
        mode: 0644
  # graphana to read the logs from prometheus
  - name: grafana
    image: grafana/grafana:4.1.1
    links:
      - prometheus
    published_ports:
      - "3000:3000"
    sync_templates:
      - src: some_file.json.j2
        dest: /etc/some_file.json
```

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
# run grafana in a container
- hosts: grafana
  roles:
    - role: AerisCloud.docker-manage
      docker_containers:
        - name: grafana
          image: grafana/grafana
          published_ports:
            - "3000:3000"
```

Missing Features
----------------

 * Reload/Restart container on file change (user action is expected)
 * `docker secret` management (not supported by module)
 * `docker service` management (not supported by module, current docker_service module is not swarm mode)

License
-------

MIT
