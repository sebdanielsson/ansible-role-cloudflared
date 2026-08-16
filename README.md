# cloudflared (Cloudflare Zero Trust Tunnel)

This role installs `cloudflared` from Cloudflare's official package repositories on Debian, RedHat
and Archlinux family systems. It can also install the systemd service and bring up a tunnel when
you provide a credential.

It's not feature complete — it covers install, service, and uninstall. PRs welcome if you want more
of `cloudflared`'s surface covered; see [CONTRIBUTING.md](CONTRIBUTING.md).

## Requirements

- A systemd init system
- One of:
  - **Debian family** (Debian, Ubuntu) — installs via `apt` from `pkg.cloudflare.com/cloudflared`
  - **RedHat family** (Fedora, RHEL, AlmaLinux, Rocky) — installs via `dnf` from the
    `cloudflared-stable` RPM repository
  - **Archlinux family** — installs via `pacman` from the distro repositories (untested)
- `ansible-core` >= 2.17.14, plus the collections in [`requirements.yml`](requirements.yml)
  (`community.general` for the pacman path, `ansible.posix`)

CI exercises Debian 13, Ubuntu 24.04, Fedora 42, and RHEL UBI 9 on every PR. Arch is supported on a
best-effort basis and is not covered by the test matrix.

## Role Variables

| Variable                       | Default | Description                                                                                                                                          |
| ------------------------------ | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `state`                        | —       | **Required.** `present` or `latest` to install, `absent` to remove. Has no default — the role fails with an undefined-variable error if you omit it. |
| `cloudflare_tunnel_enable`     | `false` | Install the `cloudflared` systemd service and bring the tunnel up. Requires `cloudflare_tunnel_credential`.                                          |
| `cloudflare_tunnel_credential` | `null`  | The tunnel token/credential passed to `cloudflared service install`.                                                                                 |

`state` is deliberately unprefixed rather than `cloudflared_state`, which is why the example
playbooks carry a `# noqa: var-naming[no-role-prefix]` comment when linted.

Setting `cloudflare_tunnel_enable: false` on a host that already has the service removes it, so the
same playbook can both create and tear down the tunnel without switching to `state: absent`.

## Example Playbooks

### Install only

```yaml
- hosts: all
  roles:
    - role: sebdanielsson.cloudflared
      state: present
```

### Install and bring up a tunnel

```yaml
- hosts: all
  roles:
    - role: sebdanielsson.cloudflared
      state: present
      cloudflare_tunnel_enable: true
      cloudflare_tunnel_credential: '{{ vault_cf_tunnel_credential }}'
```

Keep the credential in Ansible Vault or another secret store — it is passed on the
`cloudflared service install` command line.

### Remove

```yaml
- hosts: all
  roles:
    - role: sebdanielsson.cloudflared
      state: absent
```

## Debian repository migration

Earlier versions of this role (and Cloudflare's own instructions) configured
`/etc/apt/sources.list.d/cloudflare.sources` with a distro-specific suite, which breaks on release
upgrades. The role now pins `Suites: any` and, on each run, removes an existing `cloudflare.sources`
that doesn't use it — so upgrading the role migrates the host with no manual step. The
`molecule/migration` scenario covers exactly this path.

## Dependencies

None.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the development environment, linters, and how to run the
Molecule suite locally.

## Releases

Releases are automated with [Release Please](https://github.com/googleapis/release-please) and
published to [Ansible Galaxy](https://galaxy.ansible.com/ui/standalone/roles/sebdanielsson/cloudflared/).
See [CONTRIBUTING.md](CONTRIBUTING.md#releases).

## License

MIT

## Author Information

Created by Sebastian Danielsson 2024  
GitHub Profile: <https://github.com/sebdanielsson>  
Website: <https://sebdanielsson.dev>
