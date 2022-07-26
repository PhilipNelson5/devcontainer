# Devcontainer snippets and helpful tips

[Install various software from source code or precompiled binaries](Source_Installs.md)

---

Create a persistent home directory to maintain personal configuration between
new instances of the devcontainer by
[mounting a docker volume as the home directory](Mount_Home_Directory.md).

---

Orchestrate multiple containers to create a complex development environment
[using docker-compose](Use_Docker_Compose.md);

---

Mount a unique out of source build directory so that repositories in the same
host directory do not conflict.

```json
  "mounts": [
    "source=${localWorkspaceFolder}/../${localWorkspaceFolderBasename}-build,target=/workspaces/build,type=bind,consistency=delegated",
  ],
```

---

If you need to use a ptrace based debugger for C++, Go, Rust, etc.
```json
  "runArgs": [
    "--cap-add=SYS_PTRACE",
    "--security-opt",
    "seccomp=unconfined",
  ]
```

---

If you need access to host docker from inside the container
```json
  "runArgs": [
    "--privileged",
    "-v",
    "/var/run/docker.sock:/run/docker.sock",
  ]
```

---

If you need your container on a special docker network

```json
  "runArgs": [
    "--network=<name>",
  ]
```

---

If you have a local X11 server and want to run a graphical application inside
the container

```bash
export DISPLAY=host.docker.internal:0.0
```

---
