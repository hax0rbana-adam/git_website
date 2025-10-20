A simple role to install a website that is stored in a git repo.

It's worth noting that this does not set up the web server, which enables you
to use the server software of your choice instead of locking you into nginx.

# Variables

See defaults/main.yml for the variables and an explanation as to what they do.

# Examples
## Playbook
Here's an example of a minimal playbook that expects you to already have set
up a web server.

```yaml
- hosts: all
  remote_user: root
  roles:
    - role: hax0rbana_adam.git_website
```

Below is a more comprehensive playbook which shows how to also set up nginx in
a way that not only serves up the content, but ensures your .git* files are not
publicly accessibe and sets up a TLS certificate from LetsEncrypt.

```yaml
- hosts: all
  remote_user: root
  roles:
    - role: nginxinc.nginx
    - role: hax0rbana_adam.git_website
    - role: geerlingguy.certbot
    - role: nginxinc.nginx_config
  vars:
    nginx_config_http_template_enable: true
    nginx_config_http_template:
      - backup: false
        config:
          ssl:
            certificate: "/etc/letsencrypt/live/{{ ansible_fqdn }}/fullchain.pem"
            certificate_key: "/etc/letsencrypt/live/{{ ansible_fqdn }}/privkey.pem"
          servers:
            - core:
                listen:
                  - address: 0.0.0.0
                    port: 80
                    default_server: true
                    ssl: false
              rewrite:
                return:
                  code: 301
                  text: 'https://$host$request_uri'
            - core:
                root: "{{ git_website_wwwroot }}"
                server_name: "{{ inventory_hostname }}"
                index: index.html
                listen:
                  - address: 0.0.0.0
                    port: 443
                    default_server: true
                    ssl: true
              locations:
                - location: ~ \.git
                  access:
                    deny: all
                - location: /
                  try_files: "$uri $uri/ /index.html"
```

# Official repo location
All activity takes place on the official GitLab instance:
[https://gitlab.hax0rbana.org/public-repos/ansible/git_website](https://gitlab.hax0rbana.org/public-repos/ansible/git_website)

Any other hosting providers, such as GitHub.com and GitLab.com, are just mirrors
and we do not monitor the issue trackers over there.

# Support
## Matrix channel
You can also join our Matrix channel: #ansible:hax0rbana.org

This is a good place to ask questions or make requests without having to sign
up for another account.

# Contributing
See [contributor guidelines](CONTRIBUTING.md).

# License
This project is licensed under MIT License. See [LICENSE](LICENSE) for more details.
