Source: CIS Docker Benchmark v1.9.0, Section 5 (Container Runtime) (Center for Internet Security (https://www.cisecurity.org/benchmark/docker))


5.2 Ensure that, if applicable, an AppArmor Profile is enabled (Manual)
- Secure: (no flag, Docker applies as default)
- Open: `--security-opt apparmor=unconfined`

5.4 Ensure that Linux kernel capabilities are restricted within containers (Manual)
- Secure: `--cap-drop=ALL`
- Open: `--cap-add=SYS_ADMIN`

5.5 Ensure that privileged containers are not used (Manual)
- Secure: (no flag)
- Open: `--privileged`

5.6 Ensure sensitive host system directories are not mounted on containers (Manual)
- Secure: (no host folders shared)
- Open: `-v /:/host:ro`

5.10 Ensure that the host's network namespace is not shared (Manual)
- Secure: (no flag; container uses the per-trial network)
- Open: `--network=host`


**5.11 Ensure that the memory usage for containers is limited (Manual) (negative control)**
- Secure: `--memory=2g`
- Open: (no limit)

5.13 Ensure that the container's root filesystem is mounted as read only (Manual)
- Secure: `--read-only --tmpfs /tmp`
- Open: (no flag- filesystem is writable)


5.16 Ensure that the host's process namespace is not shared (Manual)
- Secure: (no flag)
- Open: `--pid=host`

5.17 Ensure that the host's IPC namespace is not shared (Manual)
- Secure: (no flag)
- Open: `--ipc=host`

5.18 Ensure that host devices are not directly exposed to containers (Manual)
- Secure: (no devices passed in)
- Open: `--device=<host disk>:<host disk>:r`

5.22 Ensure the default seccomp profile is not Disabled (Manual)
- Secure: (no flag; Docker applies its default profile)
- Open: `--security-opt seccomp=unconfined`

5.24 Ensure that docker exec commands are not used with the user=root option (Manual)
- Secure: `docker exec --user 1000`
- Open: `docker exec --user root`

5.26 Ensure that the container is restricted from acquiring additional privileges (Manual)
- Secure: `--security-opt no-new-privileges`
- Open: (no flag)

5.32 Ensure that the Docker socket is not mounted inside any containers (Manual)
- Secure: (socket not shared)
- Open: `-v /var/run/docker.sock:/var/run/docker.sock`

