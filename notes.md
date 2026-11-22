## Notes on the tested controls

- 5.5 open (--privileged) overrides 5.2, 5.4, 5.18 and 5.22. Their settings have no effect in those rows.
- 5.24 is set on the docker exec command, not on docker run.
- 5.26 only matters if the image gives the ordinary user a way to become root (for example sudo).
- 5.11 is a negative control: it is expected to have no effect on the results.

## Fixed in every trial

- --pids-limit=512 (5.29)
- One fresh Docker network per trial, removed afterwards (5.30)
- No published ports (5.8, 5.9, 5.14)
- Restart policy: never restart (5.15)
- docker exec never uses --privileged (5.23)
- One fixed image version for all trials
- Swarm mode off (5.1)
- Host runs Ubuntu or Debian, so SELinux (5.3) is not applicable

## Skipped

5.1, 5.3, 5.7, 5.8, 5.9, 5.12, 5.14, 5.15, 5.19, 5.20, 5.21, 5.23, 5.25, 5.27, 5.28, 5.29, 5.30, 5.31

Reason: these are about the whole Docker installation, incoming traffic, resource use,
or working habits. They do not change what a process inside one container can reach.
5.31 (user namespaces) was left out because it cannot be combined with 5.5, 5.10 or 5.16.
