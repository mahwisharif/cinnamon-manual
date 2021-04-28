## 1. Setup and Configuration

[`Cinnamon`](https://github.com/CompArchCam/Cinnamon) can currently target three different binary frameworks; [`Janus`](https://github.com/mahwisharif/Janus/tree/cinnamon), [`Intel Pin`](https://software.intel.com/content/www/us/en/develop/articles/pin-a-dynamic-binary-instrumentation-tool.html) and [`DynInst`](https://dyninst.org/dyninst).

### 1.1 Download Scripts for building and running frameworks and tools

`git clone https://github.com/mahwisharif/cinnamon-scripts.git`

### 1.2 Download and setup Janus
You can get the `Janus` source code with placeholders, templates and utility libraries for `Cinnamon` from https://github.com/mahwisharif/Janus.git and switching to `cinnamon` branch.

```
git clone https://github.com/mahwisharif/Janus.git
cd Janus
git checkout -b cinnamon origin/cinnamon
```

Set `janus_root` in `cinnamon-scripts/janus_build_tool` to `your-path-to/Janus`.

### 1.3 Download and setup Pin

Download Pin 3.13 version and extract source files
```
wget https://software.intel.com/sites/landingpage/pintool/downloads/pin-3.13-98189-g60a6ef199-gcc-linux.tar.g
mkdir -p pin-3.13 && tar xvzf pin-3.13-98189-g60a6ef199-gcc-linux.tar.gz --directory pin-3.13 --strip-components 1
```

You can get the relevant files for `Pin`  with placeholders, templates and utility libraries for `Cinnamon` from https://github.com/mahwisharif/pin-cinnamon

`git clone https://github.com/mahwisharif/pin-cinnamon`

Copy ``MyDSLTool`` folder from ``pin-cinnamon`` under ``your-path-to/pin-3.13/source/tools``

`cp -r pin-cinnamon/MyDSLTool pin-3.13/source/tools/`

Set `pin_root` in `cinnamon-scripts/pin_build_tool` and `pin_run_tool` to `your-path-to/pin-3.13`.

### 1.4 Download and setup Dyninst

You can get Dyninst version 10.1.0 from the following source and untar it.

`wget https://github.com/dyninst/dyninst/archive/v10.1.0.tar.gz`
`tar xzvf v10.1.0.tar.gz `
`cd dyninst-10.1.0`

Open `instructionAPI/h/InstructionCategories.h` file and add `c_LoadInsn` and `c_StoreInsn` in `enum InsnCategory{}`

 You can get the relevant files for `Dyninst`  with placeholders, templates and utility libraries for `Cinnamon` from https://github.com/mahwisharif/dyn-cinnamon.git

 ``git clone https://github.com/mahwisharif/dyn-cinnamon``
 
 Copy ``MyDSLTool`` folder from ``dyn-cinnamon`` under ``your-path-to/dyninst-10.1.0/examples``
 
`` cp -r your-path-to/dyn-cinnamon/MyDSLTool your-path-to/dyninst-10.1.0/examples/``

Set `dyn_root` in `cinnamon-scripts/dyn_build_tool` and `dyn_run_tool` to `your-path-to/dyninst-10.1.0` 

### 1.5 Downlaod and Set up Cinnamon

`git clone https://github.com/CompArchCam/Cinnamon.git`

`cd Cinnamon`

in `Scripts/compileToJanus.py` set `JanusPATH=your-path-to/Janus`, set `ParserPATH=your-path-to/Cinnamon`

in `Scripts/compileToPin.py` set `PinPATH=your-path-to/pin-3.13`, set `ParserPATH=your-path-to/Cinnamon`

in `Scripts/compileToDyn.py` set `DynPATH=your-path-to/dyninst-10.1.0`, set `ParserPATH=your-path-to/Cinnamon`

## 2. Bootstrapping OS and dependency installation using Singularity Container 

This section describes the steps to set up and run a singularity container with the required OS and software dependencies.

**Pre-requisite**: You need to have [singularity (version 3.x and above)](https://sylabs.io/guides/3.0/user-guide/quick_start.html) installed on your machine. 

### Option 1: Download pre-built singulairty image:
```
singularity pull --name cinn.sif library://mahwisharif/default/cinnamon-bootstrap:sha256.4c201006a9318ac439a1fdf1329ddab22e23f27288b894087593dd2b232b4bda
singularity shell cinn.sif                                          //enter inside the container to run interactive commands
```

### Option 2: Build singularity image from the recipe: 
You will need to have root access on the machine to use this option: 

```
git clone https://github.com/mahwisharif/singularity-cinnamon.git`
cd singularity-cinnamon
sudo singularity build cinn.sif Singularity                        //build the container
sudo singularity shell cinn.sif                                    //enter inside the container to run interactive commands
```


## 3. Building and running tools with Cinnamon

All the commands in this section will be run from inside the container. 

`export PATH=your-path-to/cinnamon-scripts:your-path-to/Janus/janus:your-path-to/Cinnamon/Scripts:$PATH`

### 3.1 Build & run a tool with cinnamon targetting Janus
```
cd Cinnamon
make TARGET=janus
compileToJanus tests/<ins.dsl>
janus_build_tool
jdsl_run <target_binary>
```

If using spec benchmarks, add this line in your `.cfg` file

`submit = jdsl_run $command`

### 3.2 Build & run a tool with cinnamon targetting Pin
```
cd Cinnamon
make TARGET=pin
compileToPin tests/<ins.dsl>
pin_build_tool
pin_run_tool <target_binary>
```

If using spec benchmarks, add this line in your `.cfg` file

`submit = pin_run_tool $command`


### 3.3 Build and run a tool with cinnamon targetting Dyninst
```
cd Cinnamon
make TARGET=dyn
compileToDyn tests/<ins.dsl>
```

The following commands build Dyninst framework. You may need to run this only once. 
```
mkdir build
cmake . -DCMAKE_INSTALL_PREFIX=build
make install -j1
```

Build dyninst-based tool
```
dyn_build_tool
dyn_run_tool <target_binary>
```

If using spec benchmarks, add this line in your `.cfg` file

`submit = dyn_run_tool $command`



**NOTE**: If you just want to compile the cinnamon program e.g. `ins.dsl` and not yet intergate in target framework, run the following command from `Cinnamon` source directory after you have run the  build script for the respective target. This will generate a number of different files containing relevant code for the target framework:
```
$ ./bdc tests/ins.dsl
```
