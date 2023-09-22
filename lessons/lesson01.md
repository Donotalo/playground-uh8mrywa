# Pre-requisites

This tutorial will be using several software packages. How to use these packages are outside of the scope of this tutorial. Nevertheless, the pre-requisites to understand this tutorial are listed below:

- How to compile code, preferrebly in C++
- CMake software build system
- Linux
- Shell scripting, preferrebly in Bash
- Docker

# Problem Statement

There is a source code repository (https://bitbucket.org/donotalo/starter/). Compile the source code in a Docker container. Extract the output of the compilation in the host system (Linux).

There is a build script (`build.sh`) in the repository. Calling this script will compile the code. The build directory, where the output will be located, can be found in the build script.
