# cloudfared (Cloudflare Zero Access Tunnel)

This role installs cloudflared (Cloudflare Zero Access Tunnel) on Debian, RedHat and Archlinux family systems. It can also create a service and bring up the connection to Cloudflare if a credential is provided.

This is my first Ansible role and it's not feature complete. Feel free to send me a PR if you would like to add more cloudflared functionality!

## Requirements

- systemd init system
- Debian family systems
- RedHat family systems with dnf
- Archlinux family systems with pacman (untested)

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

Created by Sebastian Danielsson 2024  
GitHub Profile: <https://github.com/sebdanielsson>  
Website: <https://sebdanielsson.dev>
