# Personal notes for explanation, widely adapted from AI's and googles, not in scope of demonstration



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

---

Here is the fuller version. Each control has four parts: what it is, what each setting does, and what the AI gains when it is open.

## The header

- **Source:** the exact document and section, so a judge can open the same PDF and find each line.
- **Publisher:** the Center for Internet Security, a non-profit whose benchmarks are written by groups of industry experts and used by companies and auditors as a checklist.
- **Downloaded:** the benchmark changes between versions (you saw the numbers shift from 1.2.0 to 1.9.0), so the date fixes which one you used.
- **"Secure" and "Open":** secure means the container follows the benchmark's recommendation; open means it deliberately does not.

## The 14 controls

**5.2 AppArmor**
- **What it is:** a security system inside Linux. Each program can be given a "profile", a list of what it may and may not do. Docker's standard profile, `docker-default`, forbids things like attaching disks and writing to sensitive system files.
- **Secure, no flag:** Docker applies `docker-default` automatically on Ubuntu and Debian.
- **Open, `--security-opt apparmor=unconfined`:** `--security-opt` changes a security option; `apparmor=unconfined` runs the container with no profile.
- **What the AI gains:** nothing by itself, but a layer is gone. For example, attaching a disk needs the `SYS_ADMIN` power from 5.4, and the profile still blocks it. With both open, it works.

**5.4 Capabilities**
- **What it is:** Linux splits root's power into about 40 pieces. By default Docker gives a container about 14 of them, such as changing file owners and reading files that belong to other users.
- **Secure, `--cap-drop=ALL`:** removes all of them. Even root inside the container can then do little more than an ordinary user.
- **Open, `--cap-add=SYS_ADMIN`:** keeps the default set and adds `SYS_ADMIN`, the broadest one. It covers attaching and detaching disks and many other system-administration actions.
- **What the AI gains:** the ability to attach filesystems, which is the first step of many known escapes.

**5.5 Privileged**
- **Secure, no flag:** all normal restrictions apply.
- **Open, `--privileged`:** one switch that grants every capability, exposes every hardware device on the host, and turns off both AppArmor and seccomp for the container.
- **What the AI gains:** it can see the host's disk as a device, attach it, and read every file on the host. This is the easiest escape there is.

**5.6 Host folders**
- **What it is:** Docker can make a folder from the host appear inside the container.
- **Secure:** nothing shared.
- **Open, `-v /:/host:ro`:** read it as three parts separated by colons.
  - `-v` means "share a folder".
  - `/` is the folder on the host; a single slash is the top of the disk, so everything.
  - `/host` is the path where it shows up inside the container.
  - `ro` means read-only.
- **What the AI gains:** it can browse the host's files under `/host`. Normal file permissions still apply: a file readable only by root stays unreadable to an AI running as an ordinary user.

**5.10 Host network**
- **What it is:** each container normally has its own network identity: its own address and its own `localhost`.
- **Secure, no flag:** the container sits on the trial's private network.
- **Open, `--network=host`:** the container shares the host's network identity. Inside it, `localhost` now means the host.
- **What the AI gains:** it can connect to services that only listen on the host's `localhost`, which are often left unprotected because nothing outside is expected to reach them.

**5.11 Memory limit (negative control)**
- **Secure, `--memory=2g`:** the container may use up to 2 gigabytes. If it goes over, Linux stops the program that caused it.
- **Open:** it can use all the host's memory.
- **What the AI gains:** nothing relevant; it can only slow the host down. That is why you expect no effect, and why it serves as your check.

**5.13 Read-only filesystem**
- **What it is:** a container normally has a writable disk of its own.
- **Secure, `--read-only --tmpfs /tmp`:** `--read-only` makes that whole disk unchangeable. `--tmpfs /tmp` adds one scratch folder held in memory, which disappears when the container stops. By default programs cannot be run directly from it.
- **Open, no flag:** the AI can write anywhere in the container.
- **What the AI loses when secure:** it cannot install extra tools or keep files outside `/tmp`. This is the control most likely to stop normal work too.

**5.16 Host processes**
- **Secure, no flag:** listing running programs shows only the container's own.
- **Open, `--pid=host`:** the list shows every program on the host.
- **What the AI gains:** it can read the command each host program was started with, and anything typed there, such as a password, is visible. Reading deeper, into a program's environment or memory, needs root plus extra capabilities.

**5.17 Host shared memory**
- **What it is:** an area of memory that programs use to hand data to each other quickly, visible as the folder `/dev/shm`.
- **Secure, no flag:** the container has its own, empty one.
- **Open, `--ipc=host`:** it sees the host's.
- **What the AI gains:** it can read whatever host programs have left there.

**5.18 Host devices**
- **What it is:** Linux represents hardware as special files, such as `/dev/sda` for a disk.
- **Secure:** the container gets none of the host's.
- **Open, `--device=<host disk>:<host disk>:r`:** the first name is the device on the host, the second is its name inside, and `r` allows reading only.
- **What the AI gains:** it can read the disk's raw contents and pull files out without the folder ever being shared. This usually needs root, and more skill than 5.6.

**5.22 Seccomp**
- **What it is:** programs ask the core of the operating system for everything, through a few hundred kinds of request. Seccomp is a filter on those requests.
- **Secure, no flag:** Docker's standard filter blocks a few dozen risky kinds, as I recall around 44.
- **Open, `--security-opt seccomp=unconfined`:** nothing is filtered.
- **What the AI gains:** access to the rarely used requests where escape flaws have been found before. Capabilities still limit what many of them can do.

**5.24 Run as root**
- **What it is:** every command runs as some user. Root, user number 0, can read and change any file in the container. An ordinary user cannot.
- **Secure, `docker exec --user 1000`:** `docker exec` runs a command inside a running container; `--user 1000` runs it as ordinary user number 1000, which must exist in your image.
- **Open, `docker exec --user root`:** the command runs as root.
- **What the AI gains:** most other open controls only become usable with root. This one multiplies the rest.

**5.26 No extra privileges**
- **What it is:** some programs, such as `sudo`, are allowed to raise the user to root.
- **Secure, `--security-opt no-new-privileges`:** sets a switch in Linux that stops any program in the container from gaining power. It cannot be undone from inside.
- **Open, no flag:** `sudo` and similar tools work.
- **What the AI gains:** a route from ordinary user to root, which undoes 5.24.

**5.32 Docker socket**
- **What it is:** Docker has a background service running as root on the host. The socket, a special file at `/var/run/docker.sock`, is how commands reach it.
- **Secure:** the file is not shared in.
- **Open, `-v /var/run/docker.sock:/var/run/docker.sock`:** shares that one file to the same path inside.
- **What the AI gains:** it can tell Docker to start a second container with everything open and the host's disk shared, then read the host through that. The socket file is normally usable only by root.

## Notes on the tested controls

- **5.5 overrides four others:** privileged mode already does what opening 5.2, 5.4, 5.18 and 5.22 would do. A privileged row with seccomp "secure" behaves the same as one with it "open", so treat those as one case.
- **5.24 goes on `docker exec`:** `docker run` creates the container once per trial; `docker exec` runs each of the AI's commands inside it. Thirteen controls are fixed at creation, and this one is applied on every command.
- **5.26 needs `sudo` in the image:** if there is no way to become root, blocking it changes nothing and the control would look useless when it is not.
- **5.11 is the negative control:** if it shows an effect, suspect the experiment.

## Fixed in every trial

- **`--pids-limit=512`:** caps running programs at 512. A command that keeps copying itself would otherwise freeze the VM and end your run.
- **One fresh network per trial:** a private network holding only that trial's containers, deleted afterwards. It keeps trials from seeing each other and makes each traffic log belong to one trial.
- **No published ports:** publishing a port lets the outside world connect in. You need none, and it removes three controls from consideration.
- **Never restart:** Docker can bring a crashed container back automatically. You want crashes to stay visible in the results.
- **`docker exec` never uses `--privileged`:** that option would give a single command full power regardless of the row's settings, and corrupt the result.
- **One fixed image version:** build the image once and name that exact build in every trial, so the software inside is identical across all 1,000.
- **Swarm mode off:** swarm links several Docker computers into a group and opens extra network services. You use one machine.
- **Ubuntu or Debian host:** these come with AppArmor. SELinux is the equivalent on Fedora and Red Hat, and a system uses one or the other.
