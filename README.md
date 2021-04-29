# Cinnamon

This directory contains instructions for building and running the Cinnamon language compiler, described in the paper:

Cinnamon: A Domain-Specific Language for Binary Profiling and Monitoring,
Mahwish Arif, Ruoyu Zhou, Hsi-Ming Ho and Timothy M. Jones,
CGO 2021

Please cite this paper if you produce any work that builds upon this code and / or data.


## Licence

Cinnamon is released under an Apache licence.  Everything else in this repository is released under a CC BY-SA licence.


## Setup and configuration

[Cinnamon](https://github.com/CompArchCam/Cinnamon) can currently target three different binary frameworks; [Janus](https://github.com/timothymjones/Janus), [Pin](https://software.intel.com/content/www/us/en/develop/articles/pin-a-dynamic-binary-instrumentation-tool.html) and [Dyninst](https://dyninst.org/dyninst).

All the code and scripts for targeting Pin and Dyninst are available in the Cinnamon repository.  To target Janus, there is a branch in the Janus repository containing the relevant code.


### Downlaod and set up Cinnamon

Clone the Cinnamon repository:

`git clone https://github.com/CompArchCam/Cinnamon.git`

### Download and set up Janus

You can get the Janus implementation with placeholders, templates and utility libraries for Cinnamon from the main Janus repository at https://github.com/timothymjones/Janus.git, then switch to the `cinnamon` branch.

```shell-session
git clone https://github.com/timothymjones/Janus.git
cd Janus
git checkout -b cinnamon origin/cinnamon
```

Back in the Cinnamon repository, in the `janus_build_tool` script, set `janus_root` to be the location that you have cloned Janus to.  In the `compileToJanus.py` script, set `JanusPATH` to be the location that you have cloned Janus to and `ParserPATH` to be the location that you have cloned Cinnamon to.

### Download and set up Pin

You can obtain Pin version 3.13 as follows:

```shell-session
wget https://software.intel.com/sites/landingpage/pintool/downloads/pin-3.13-98189-g60a6ef199-gcc-linux.tar.g
mkdir -p pin-3.13 && tar xvzf pin-3.13-98189-g60a6ef199-gcc-linux.tar.gz --directory pin-3.13 --strip-components 1
```

Everything required for Pin is contained within the `targets/Pin` directory in the Cinnamon repository.  Copy the `MyDSLTool` directory to `path-to-your-pin-root-dir/source/tools`, where `path-to-your-pin-root` should be self-explanatory.

In the `pin_build_tool` and `pin_run_tool` scripts, set `pin_root` to point to the location you've extracted Pin to.  In the `compileToPin.py` script, set `PinPATH` to be `path-to-your-pin-root-dir/source/tools/MyDSLTool` and `ParserPATH` to be the location that you have cloned Cinnamon to.

### Download and set up Dyninst

You can obtain Dyninst version 10.1.0 as follows:

```shell-session
wget https://github.com/dyninst/dyninst/archive/v10.1.0.tar.gz
tar xzvf v10.1.0.tar.gz
```

Once extracted, add `c_LoadInsn` and `c_StoreInsn` into `enum InsnCategory` in `dyninst-10.1.0/instructionAPI/h/InstructionCategories.h`.

Everything else required for Dyninst is contained within the `targets/Dyninst` directory.  Copy the `MyDSLTool` directory to `path-to-your-dyn-root-dir/examples`, where `path-to-your-dyn-root-dir` should be self-explanatory.

In the `dyn_build_tool` and `dyn_run_tool` scripts, set `dyn_root` to point to the location you've extracted Dyninst to.  In the `compileToDyn.py` script, set `DynPATH` to be `path-to-your-dyn-root-dir/examples/MyDSLTool` and `ParserPATH` to be the location that you have cloned Cinnamon to.


## Bootstrapping an OS and dependency installation using a Singularity container

This section describes the steps to set up and run a Singularity container with the required OS and software dependencies.

**Pre-requisite**: You need to have [Singularity (version 3.x and above)](https://sylabs.io/guides/3.0/user-guide/quick_start.html) installed on your machine.

### Option 1: Download a pre-built singularity image

```shell-session
singularity pull --name cinn.sif library://mahwisharif/default/cinnamon-bootstrap:sha256.4c201006a9318ac439a1fdf1329ddab22e23f27288b894087593dd2b232b4bda
singularity shell cinn.sif    // enter the container to run interactive commands
```

### Option 2: Build a singularity image from the recipe

You will need to have root access on the machine to use this option.

```shell-session
cd path-to-cinnamon/containers
sudo singularity build cinn.sif Singularity    // build the container
sudo singularity shell cinn.sif                // enter the container to run interactive commands
```


## Building and running tools with Cinnamon

All the commands in this section will be run from inside the container.

```shell-session
export PATH=your-path-to/Janus/janus:your-path-to/Cinnamon/Scripts:$PATH
```

**NB**: You may skip section 2 altogether and run the commands in this section without using a singularity container. However, this may result in some build or configuration issues as required dependencies may not be installed on your system.

### Build & run a tool with Cinnamon targeting Janus

```shell-session
cd Cinnamon
make TARGET=janus
compileToJanus tests/<ins.dsl>
janus_build_tool
jdsl_run <target_binary>
```

If using SPEC CPU benchmarks, add this line into your `.cfg` file:

```shell-session
submit = jdsl_run $command
```

### Build & run a tool with Cinnamon targeting Pin

```shell-session
cd Cinnamon
make TARGET=pin
compileToPin tests/<ins.dsl>
pin_build_tool
pin_run_tool <target_binary>
```

If using SPEC CPU benchmarks, add this line into your `.cfg` file:

```shell-session
submit = pin_run_tool $command
```

### Build & run a tool with Cinnamon targeting Dyninst

```shell-session
cd Cinnamon
make TARGET=dyninst
compileToDyn tests/<ins.dsl>
```

The following commands build the Dyninst framework. You need to run this only once inside your `Dyninst` root dir:

```shell-session
cd dyninst-10.1.0
mkdir build
cmake . -DCMAKE_INSTALL_PREFIX=build
make install -j1
```

Build the Dyninst-based tool:

```shell-session
dyn_build_tool
dyn_run_tool <target_binary>
```

If using SPEC CPU benchmarks, add this line into your `.cfg` file:

```shell-session
submit = dyn_run_tool $command
```

### Build a tool with Cinnamon with no target

If you just want to compile a Cinnamon program, e.g. `ins.dsl`, and not yet integrate it into a target framework, run the following command from the Cinnamon source directory after you have run the build script for the respective target. This will generate a number of different files containing relevant code for the target framework:

```shell-session
./bdc tests/ins.dsl
```
