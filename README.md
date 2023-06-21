# cloudfared (Cloudflare Zero Access Tunnel)

This role installs cloudflared (Cloudflare Zero Access Tunnel) on Debian, RedHat and Archlinux family systems. It can also create a service and bring up the connection to Cloudflare if a credential is provided.

This is my first Ansible role and it's not feature complete. Feel free to send me a PR if you would like to add more cloudflared functionality!

## Requirements

- systemd init system
- Debian family systems (Pretty much all currently maintained)
- RedHat family systems with the dnf package manager
- Archlinux family systems with pacman
- ansible-core 2.15.1 due to the usage of the deb822_repository module

## Role Variables

`cloudflare_tunnel_enable` - Whether to create a service for the tunnel or not. This option requires `cloudflare_tunnel_credential` to be set.
`cloudflare_tunnel_credential` - You need to provide a credential to bring up the tunnel.

## Dependencies

None.

## Example Playbook

    - hosts: localhost

      roles:
        - role: sebdanielsson.cloudflared
          vars:
            cloudflare_tunnel_enable: true
            cloudflare_tunnel_credential: <your_cf_tunnel_credential>
          state: present

## License

MIT

## Author Information

Created by Sebastian Danielsson 2023  
GitHub Profile: <https://github.com/sebdanielsson>  
Website: <https://sebbo.io>
