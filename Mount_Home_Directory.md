When using a devcontainer you often want personal settings like global git
configuration, ssh keys, bash history, etc. to be maintained regardless of the
container image. By default, when a new container image is used, the home
directory is not preserved. This can be mitigated by mounting a persistent
volume to be used as the home directory.

Here are the steps to enable this:

1. Using Docker, create a linux $HOME directory volume.

```bash
docker volume create linux-home-dir
```

2. Open the devcontainer using VSCode and in the terminal (in the container)
   run  the following commands to determine the absolute path to the $HOME
   directory.

```bash
cd ~
pwd
# (Example result: "/opt/app-root/src")
```

3. In VSCode, update your _devcontainer.json_ file and add the following.

```js
// The optional 'runArgs' property can be used to specify additional runtime arguments
 "runArgs": [
   // Mount home directory as a volume to preserve settings like .ssh and .gitconfig
   "-v",
   "linux-home-dir:/opt/app-root/src"
 ],
```

4. In VSCode, run **Ctrl+Shift+P** `Remote-Container: Rebuild Container`.

5. Your _linux-home-dir_ volume will now be mounted as the _$HOME_ directory of
   the container making it much more convenient to use between updates to the 
   devcontainer image.
