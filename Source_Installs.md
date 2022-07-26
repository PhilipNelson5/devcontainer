# Install software from source or precompiled binaries

## Git

[versions](https://github.com/git/git/tags)

```docker
RUN set -eux; \
  GIT_VERSION=2.37.0; \
  yum install --assumeyes \
    openssl-devel \
    curl-devel \
    expat-devel \
    openssh-clients; \
  yum --assumeyes clean all; \
  wget \
    --progress=dot:mega \
    https://mirrors.edge.kernel.org/pub/software/scm/git/git-${GIT_VERSION}.tar.gz; \
  tar -xzf git-${GIT_VERSION}.tar.gz; \
  cd git-${GIT_VERSION}; \
  # make --silent --jobs "$(nproc)" prefix=/usr NO_TCLTK=NoThanks profile-fast; \
  # make --silent --jobs "$(nproc)" prefix=/usr PROFILE=BUILD install; \
  make --silent --jobs "$(nproc)" prefix=/usr NO_TCLTK=NoThanks all; \
  make --silent --jobs "$(nproc)" prefix=/usr install; \
  cd ..; \
  rm -rf git-${GIT_VERSION}.tar.gz git-${GIT_VERSION};
```
vscode extensions
```
"eamodio.gitlens",
"mhutchie.git-graph",
```

---
## Cmake

[latest](https://cmake.org/download/)

```docker
# Install a newer version of CMake
RUN set -eux; \
  CMAKE_VERSION=3.23.2; \
  wget \
    --progress=dot:mega \
    --output-document=cmake.sh \
    https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}-Linux-x86_64.sh; \
  chmod 744 cmake.sh; \
  mkdir /opt/cmake; \
  ./cmake.sh --prefix=/opt/cmake --skip-license --exclude-subdir; \
  ln -s /opt/cmake/bin/ccmake /usr/bin/ccmake; \
  ln -s /opt/cmake/bin/ctest /usr/bin/ctest; \
  ln -s /opt/cmake/bin/cpack /usr/bin/cpack; \
  ln -s /opt/cmake/bin/cmake /usr/bin/cmake; \
  rm cmake.sh;
```
vscode extension
```
"ms-vscode.cpptools-extension-pack",
```

---
## Valgrind

[latest](https://valgrind.org/downloads/current.html)

```docker
RUN set -eux; \
  VALGRIND_VERSION=3.19.0; \
  wget \
    --progress=dot:mega \
    https://sourceware.org/pub/valgrind/valgrind-${VALGRIND_VERSION}.tar.bz2; \
  tar -xjf valgrind-${VALGRIND_VERSION}.tar.bz2; \
  cd valgrind-${VALGRIND_VERSION}; \
  ./configure; \
  make --silent --jobs "$(nproc)"; \
  make --silent --jobs "$(nproc)" install; \
  cd ..; \
  rm -rf valgrind-${VALGRIND_VERSION}.tar.bz2 valgrind-${VALGRIND_VERSION};
```

---
## Python3

python [versions](https://www.python.org/ftp/python/)

pip [latest](https://pypi.org/project/pip/)

```docker
RUN set -eux; \
  PYTHON_VERSION=3.9.13; \
  PIP_VERSION=22.2; \
  yum install --assumeyes \
    openssl-devel \
    libffi-devel; \
  yum --assumeyes clean all; \
  wget \
    --progress=dot:mega \
    --output-document=Python-${PYTHON_VERSION}.tar.gz \
    https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz; \
  tar -xzf Python-${PYTHON_VERSION}.tar.gz; \
  cd Python-${PYTHON_VERSION}; \
  ./configure --enable-optimizations --with-lto; \
  make --silent --jobs "$(nproc)"; \
  make --silent --jobs "$(nproc)" install; \
  cd ..; \
  rm -rf Python-${PYTHON_VERSION}.tar.gz Python-${PYTHON_VERSION}; \
  pip3 install --no-cache-dir --upgrade pip==${PIP_VERSION};
```

---
## gcovr and lizard

gcovr [latest](https://pypi.org/project/gcovr/)

lizard [latest](https://pypi.org/project/lizard/)

```docker
# Install gcovr and lizard
RUN set -eux; \
  GCOVR_VERSION=5.0; \
  LIZARD_VERSION=1.17.10; \
  pip3 install --no-cache-dir \
    gcovr==${GCOVR_VERSION} \
    lizard==${LIZARD_VERSION};
```

---
## Hadolint (dockerfile lint)
[latest](https://github.com/hadolint/hadolint/releases)
```docker
RUN set -eux; \
  HADOLINT_VERSION=2.10.0; \
  wget \
    --progress=dot:giga \
    --output-document=/usr/bin/hadolint \
    https://github.com/hadolint/hadolint/releases/download/v${HADOLINT_VERSION}/hadolint-Linux-x86_64; \
  chmod +x /usr/bin/hadolint;
```
vscode extension
```
"exiasr.hadolint",
```

---
## Shellcheck (shell script lint)

[latest](https://github.com/koalaman/shellcheck/releases)

```docker
RUN set -eux; \
  SHELLCHECK_VERSION=0.8.0; \
  wget \
    --progress=dot:mega \
    https://github.com/koalaman/shellcheck/releases/download/v0.8.0/shellcheck-v${SHELLCHECK_VERSION}.linux.x86_64.tar.xz; \
  tar -xJf shellcheck-v${SHELLCHECK_VERSION}.linux.x86_64.tar.xz; \
  cp shellcheck-v${SHELLCHECK_VERSION}/shellcheck /usr/local/bin; \
  rm -rf shellcheck-v${SHELLCHECK_VERSION}.linux.x86_64.tar.xz shellcheck-v${SHELLCHECK_VERSION};
```
vscode extension
```
"timonwong.shellcheck",
```

---
## Cppcheck (C++ static analysis)

[latest](https://github.com/danmar/cppcheck/tags)

```docker
RUN set -eux; \
  CPPCHECK_VERSION=2.8.1; \
  git clone --depth 1 --branch ${CPPCHECK_VERSION} https://github.com/danmar/cppcheck.git; \
  scl enable devtoolset-11 \
    "cmake -B build \
    -D BUILD_TESTING=off \
    -D USE_MATCHCOMPILER=on \
    -D CMAKE_BUILD_TYPE=Release \
    cppcheck \
    && cmake --build build --target install --parallel \"$(nproc)\""; \
  rm -r build cppcheck;
```

---

## NodeJs

[versions](https://nodejs.org/en/about/releases/)

```docker
WORKDIR /opt/
RUN set -eux; \
  NODE_VERSION=12.22.3; \
  wget \
    --progress=dot:mega \
    https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz; \
  tar -xJf node-v${NODE_VERSION}-linux-x64.tar.xz; \
  rm /opt/node-v${NODE_VERSION}-linux-x64.tar.xz; \
  ln -s /opt/node-v${NODE_VERSION}-linux-x64/ /opt/node; \
  npm install --global --quiet npm;
ENV PATH "$PATH:/opt/node/bin"
```

---

## Protobuf

[latest](https://github.com/protocolbuffers/protobuf/releases)

```docker
WORKDIR /opt/
RUN set -eux; \
  PROTO_VERSION=3.15.8; \
  wget
    --progress=dot:mega \
    https://github.com/protocolbuffers/protobuf/releases/download/v${PROTO_VERSION}/protoc-${PROTO_VERSION}-linux-x86_64.zip; \
  unzip protoc-${PROTO_VERSION}-linux-x86_64.zip -d protoc-${PROTO_VERSION}; \
  ln -s /opt/protoc-${PROTO_VERSION}/bin/protoc /usr/local/bin/protoc; \
  rm protoc-${PROTO_VERSION}-linux-x86_64.zip;
```
vscode extension
```
"zxh404.vscode-proto3",
```

---

## ZSH and Oh My ZSH

[repo](https://github.com/ohmyzsh/ohmyzsh#getting-started)

```docker
RUN set -eux; \
  yum install --assumeyes \
    zsh; \
  yum --assumeyes clean all; \
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"; \
  chsh --shell $(which zsh);
```

---

## FZF

[repo](https://github.com/junegunn/fzf#-)

```docker
RUN set -eux; \
  git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf; \
  cd ~/.fzf/; \
  ./install --all; \
```

---

## TeX Live (LaTeX)

Installs latest

**Note:** This is a very long install, put it early in the dockerfile to 
keep it cached when making changes to the dockerfile.

```docker
RUN set -eux; \
  yum install --assumeyes \
    wget \
    ca-certificates \
    perl-Digest-MD5; \
  yum --assumeyes clean all; \
  wget \
    --progress=dot:mega \
    https://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz; \
  tar -xzf install-tl-unx.tar.gz; \
  cd install-tl-*; \
  perl install-tl \
    --paper=letter \
    --texdir=/opt/texlive \
    --no-interaction; \
  ln -s /opt/texlive/bin/x86_64-linux /usr/local/bin; \
  cd ..; \
  rm -rf install-tl-unx.tar.gz install-tl-*;
```
vscode extension
```
"James-Yu.latex-workshop",
```