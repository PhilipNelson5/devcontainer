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

```text
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

```text
"ms-vscode.cpptools-extension-pack",
"brobeson.ctest-lab",
```

---

## Clang / Clang-tools

[latest](https://github.com/llvm/llvm-project/tags)

```docker
RUN set -eux; \
  LLVM_VERSION=12.0.1; \
  git clone --depth 1 --branch llvmorg-${LLVM_VERSION} https://github.com/llvm/llvm-project.git; \
  mkdir build; \
  cmake \
    -G "Unix Makefiles" \
    -Wno-dev \
    -DCMAKE_BUILD_TYPE=Release \
    -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra" \
    -S llvm-project/llvm \
    -B build; \
  cd build; \
  make --silent --jobs "$(nproc)" install clang clang-format clang-tidy; \
  cd ..;
```

vscode task

```json
{
  "label": "Clang-Tidy",
  "type": "shell",
  "command": "find",
  "args": [
    ".",
    "-name",
    "'*.[ch]pp'",
    "-exec",
    "clang-tidy",
    "-p=../build/compile_commands.json",
    "--extra-arg='-Wno-unknown-warning-option'",
    "{}",
    "\\;"
  ],
  "options": {
    "cwd": "${workspaceFolder}"
  },
  "problemMatcher": {
    "owner": "clang-tidy task",
    "fileLocation": "absolute",
    "source": "(",
    "pattern": {
      "regexp": "^(.*):(\\d+):(\\d+):\\s+(error):\\s+(.*)\\[(.*),",
      "file": 1,
      "line": 2,
      "column": 3,
      "severity": 4,
      "message": 5,
      "code": 6
    }
  }
},
```

vscode extension

```text
"xaver.clang-format",
"notskm.clang-tidy",
```

---

## Boost
[latest](https://www.boost.org/users/download/)

```docker
RUN set -eux; \
  BOOST_VERSION_MAJOR=1; \
  BOOST_VERSION_MINOR=80; \
  BOOST_VERSION_BUILD=0; \
  BOOST_VERSION_UNDER=${BOOST_VERSION_MAJOR}_${BOOST_VERSION_MINOR}_${BOOST_VERSION_BUILD}; \
  BOOST_VERSION_DOT=${BOOST_VERSION_MAJOR}.${BOOST_VERSION_MINOR}.${BOOST_VERSION_BUILD}; \
  wget \
    --progress=dot:mega \
    https://boostorg.jfrog.io/artifactory/main/release/${BOOST_VERSION_DOT}/source/boost_${BOOST_VERSION_UNDER}.tar.gz; \
  tar -zxf boost_${BOOST_VERSION_UNDER}.tar.gz; \
  cd boost_${BOOST_VERSION_UNDER}; \
  ./bootstrap.sh; \
  ./b2 install --prefix=/usr; \
  cd ..; \
  rm -r boost_${BOOST_VERSION_UNDER} boost_${BOOST_VERSION_UNDER}.tar.gz; \
  ldconfig;
```

---

## Include What You Use

[latest](https://github.com/include-what-you-use/include-what-you-use/branches/stale) match to the clang version you are using

```docker
RUN set -eux; \
  IWYU_VERSION=clang_12; \
  git clone --depth 1 --branch ${IWYU_VERSION} https://github.com/include-what-you-use/include-what-you-use.git; \
  cmake \
    -G "Unix Makefiles" \
    -Wno-dev \
    -DCMAKE_BUILD_TYPE=Release \
    -DLLVM_ENABLE_PROJECTS=clang \
    -DLLVM_EXTERNAL_PROJECTS=iwyu \
    -DLLVM_EXTERNAL_IWYU_SOURCE_DIR=include-what-you-use \
    -S llvm-project/llvm \
    -B build; \
  cd build; \
  make --silent --jobs "$(nproc)" install include-what-you-use; \
  cd ..; \
  rm -rf llvm-project include-what-you-use build;
```

vscode task

```json
{
  "label": "Include What You Use",
  "type": "shell",
  "command": ".vscode/task-iwyu.sh",
  "options": {
    "cwd": "${workspaceFolder}"
  },
  "problemMatcher": {
    "owner": "iwyu",
    "fileLocation": "absolute",
    "pattern": [
      {
        "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
        "file": 1,
        "line": 2,
        "column": 3,
        "severity": 4,
        "message": 5
      },
      {
        "regexp": "^(#include.*)$",
        "message": 1,
      }
    ]
  }
}
```

.vscode/task-iwyu.sh

```bash
shopt -s globstar
iwyu_tool.py \
  -o clang \
  -j "$(nproc)" \
  -p ../build/ \
  ./**/*.cpp -- \
  -Xiwyu --mapping_file="$(pwd)"/.iwyu.imp \
  -Xiwyu --quoted_includes_first \
  -Xiwyu --cxx17ns \
  -Wno-unknown-warning-option
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
    bzip2-devel \
    openssl-devel \
    libffi-devel; \
  yum --assumeyes clean all; \
  wget \
    --progress=dot:mega \
    --output-document=Python-${PYTHON_VERSION}.tar.gz \
    https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz; \
  tar -xzf Python-${PYTHON_VERSION}.tar.gz; \
  cd Python-${PYTHON_VERSION}; \
  ./configure --enable-optimizations --with-lto CFLAGS=-fPIC; \
  make --silent --jobs "$(nproc)"; \
  make --silent --jobs "$(nproc)" install; \
  ln -s /usr/local/bin/python3 /usr/local/bin/python; \
  cd ..; \
  rm -rf Python-${PYTHON_VERSION}.tar.gz Python-${PYTHON_VERSION}; \
  pip3 install --no-cache-dir --upgrade pip==${PIP_VERSION};
```

_apt-get_
```docker
RUN set -eux; \
    apt-get update; \
    apt-get install --assume-yes --no-install-recommends \
        libsqlite3-dev \
        pkg-config \
        zlib1g-dev \
        libbz2-dev \
        libssl-dev \
        libffi-dev; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*;
```

---

## SWIG

swig [versions](https://sourceforge.net/projects/swig/files/swig/)

pcre2 [versions](https://sourceforge.net/projects/pcre/files/pcre2/)

```docker
RUN set -eux; \
    SWIG_VERSION=4.2.1; \
    PCRE2_VERSION=10.37; \
    wget \
        --progress=dot:mega \
        --output-document=swig-${SWIG_VERSION}.tar.gz \
        https://sourceforge.net/projects/swig/files/swig/swig-${SWIG_VERSION}/swig-${SWIG_VERSION}.tar.gz/download; \
    tar -xzf swig-${SWIG_VERSION}.tar.gz; \
    cd swig-${SWIG_VERSION}; \
    wget \
        --progress=dot:mega \
        --output-document=pcre2-${PCRE2_VERSION}.tar.bz2 \
        https://sourceforge.net/projects/pcre/files/pcre2/${PCRE2_VERSION}/pcre2-${PCRE2_VERSION}.tar.bz2/download; \
    Tools/pcre-build.sh; \
    ./configure; \
    make --silent --jobs "$(nproc)"; \
    make --silent install; \
    cd ..; \
    rm -r swig-${SWIG_VERSION}.tar.gz swig-${SWIG_VERSION}
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

vscode task for gcovr

```json
{
  "label": "Test Coverage",
  "type": "shell",
  "command": "gcovr",
  "options": {
    "cwd": "${command:cmake.buildDirectory}"
  },
  "args": [
    "--root",
    "${workspaceFolder}",
    "--html-details",
    "test_coverage/index.html",
    "."
  ],
  "problemMatcher": []
},
```

vscode task for lizard

```json
{
  "label": "Lizard CCN",
  "type": "shell",
  "command": "lizard",
  "args": [
    "--CCN",
    "10",
    "--length",
    "50",
    "--arguments",
    "5",
    "--warnings_only",
    "--modified"
  ],
  "options": {
    "cwd": "${workspaceFolder}"
  },
  "problemMatcher": "$gcc"
},
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

```text
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

```text
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

### Just the compilers

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

### Full installation

```docker
# Install protobuf 3 (leave the 3 off the version here)
RUN set -eux; \
  PROTOBUF_VERSION=21.7; \
  wget \
    --progress=dot:mega \
    https://github.com/protocolbuffers/protobuf/releases/download/v${PROTOBUF_VERSION}/protobuf-cpp-3.${PROTOBUF_VERSION}.tar.gz; \
  tar -xzf protobuf-cpp-3.${PROTOBUF_VERSION}.tar.gz; \
  cd protobuf-3.${PROTOBUF_VERSION}; \
  ./configure --disable-shared --prefix=/usr; \
  make --silent --jobs $(nproc); \
  make --silent check; \
  make --silent --jobs $(nproc) install; \
  ldconfig; \
  cd ..; \
  rm -r protobuf-3.${PROTOBUF_VERSION} protobuf-cpp-3.${PROTOBUF_VERSION}.tar.gz
```

vscode extension

```text
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

```text
"James-Yu.latex-workshop",
```

---

## Plant UML

[latest](https://github.com/plantuml/plantuml/releases)

```docker
RUN set -eux; \
  PLANTUML_VERSION=1.2022.6; \
  yum install --assumeyes \
    java-11; \
  yum --assumeyes clean all; \
  wget \
    --directory-prefix=/opt/ \
    --progress=dot:mega \
    https://github.com/plantuml/plantuml/releases/download/v${PLANTUML_VERSION}/plantuml-${PLANTUML_VERSION}.jar; \
  echo >> /usr/local/bin/plantuml '#!/bin/sh'; \
  echo >> /usr/local/bin/plantuml "java -jar /opt/plantuml-${PLANTUML_VERSION}.jar" '"$@"'; \
  chmod +x /usr/local/bin/plantuml;
```

vscode extension

```text
"jebbs.plantuml",
```

---

## Helm

[latest](https://github.com/helm/helm/releases)

```docker
RUN set -eux; \
  HELM_VERSION=3.10.1; \
  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3; \
  chmod 700 get_helm.sh; \
  ./get_helm.sh --version ${HELM_VERSION}; \
  rm get_helm.sh;
```
