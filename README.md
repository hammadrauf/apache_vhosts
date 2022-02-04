# Ansible apache_vhosts role

This is an [Ansible](http://www.ansible.com) role to configure apache server virtual hosts.

## Role Variables

A list of all the default variables for this role is available in `defaults/main.yml`.

## Example Playbook

This is an example playbook:

```yaml
---

- hosts: all
  roles:
    - role: amtega.apache_vhosts
    - role: amtega.apache_vhosts
      vars:
        apache_vhosts:
          - server_name: example.domain.local
            file: myhost.conf
            template: redirect_to_ssl
            port: 80

          - server_name: example.domain.local
            file: myhost_ssl.conf
            template: standard
            port: 443
            includes:
              - /etc/httpd/ssl.conf
            rewrite_rules:
              - "^/$ /myapp [R]"
            proxy_pass:
              - >-
                /myapp balancer://tomcat-cluster/myapp
            headers:
              - >-
                add Set-Cookie "ROUTEID=.%{BALANCER_WORKER_ROUTE}e; path=/myapp"
                env=BALANCER_ROUTE_CHANGED

        apache_vhosts_defaults:
          directory: /etc/httpd/conf.ansible.d
          owner: apache
          group: apache
          dir_mode: "0755"
          file_mode: "0640"
          addr: "*"
          proxy_preserve_host: yes
          rewrite_engine: yes
          balancer:
            name: tomcat-cluster
            members:
              - ajp://backend1.domain.local:8080 route=backend1-jvm
              - ajp://backend2.domain.local:8080 route=backend2-jvm
            lbmethod: byrequests
            sticky_session: ROUTEID
          error_log: /var/log/apache/my_error_log
          post_script: "service httpd restart"
          post_script_become: yes
          post_script_become_user: root
          post_script_become_method: sudo
          state: present
```

## Testing

Tests are based on [molecule with docker containers](https://molecule.readthedocs.io/en/latest/installation.html).

```shell
cd amtega.apache_vhosts

molecule test --all
```

## License

Copyright (C) 2022 AMTEGA - Xunta de Galicia

This role is free software: you can redistribute it and/or modify it under the terms of:

GNU General Public License version 3, or (at your option) any later version; or the European Union Public License, either Version 1.2 or – as soon they will be approved by the European Commission ­subsequent versions of the EUPL.

This role is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details or European Union Public License for more details.

## Author Information

- José Enrique Mourón Regueira
- Juan Antonio Valiño García.
