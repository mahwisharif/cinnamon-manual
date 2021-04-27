## 1. Bootstrapping OS and dependency installation using Singularity Container 

**Pre-requisite**: You need to have [singularity](https://sylabs.io/guides/3.0/user-guide/quick_start.html) installed on your machine. 

### Option 1: Download pre-built singulairty image:
For singularity 2.x versions: 
```
singularity pull --name cinn.simg shub://mahwisharif/singularity-cinnamon
singularity shell cinn.simg
```
For singularity 3.x versions: 
```
singularity pull --name cinn.sif shub://mahwisharif/singularity-cinnamon
singularity shell cinn.sif
```

### Option 2: Build singularity image from the recipe: 
You will need to have root access on the machine to use this option: 

```
git clone https://github.com/mahwisharif/singularity-cinnamon.git`
cd singularity-cinnamon
sudo singularity build cinn.simg Singularity    //or cinn.sif for singularity 3.x versions
sudo singularity shell cinn.simg
```


## 2.Configuration 
Once inside the singularity shell. You will need to set the following configurations. 

```
ln -s /usr/bin/clang++-3.8 /usr/bin/clang++
ln -s /usr/bin/clang-3.8 /usr/bin/clang
```
[`Cinnamon`](https://github.com/CompArchCam/Cinnamon) can currently target three different binary frameworks; [`Janus`](https://github.com/mahwisharif/Janus/tree/cinnamon), [`Intel Pin`](https://software.intel.com/content/www/us/en/develop/articles/pin-a-dynamic-binary-instrumentation-tool.html) and [`DynInst`](https://dyninst.org/dyninst).

These were downloaded/cloned/copied under `/home` during bootstrap. 

`export PATH=/home/scripts:/home/Janus/janus:/home/Cinnamon/Scripts:$PATH`

### 2.1 Set up and configure Janus

`cd /home/Janus`

set `janus_root` in `/home/scripts/janus_build_tool` to `/home/Janus` and then run:

`janus_build_tool`

Note: We are using 'cinnamon' branch of Janus which is checked out during bootstrap of singularity image. You can make sure that you are at the correct branch by running `git branch` in `/home/Janus` directory.

### 2.2 Set up and configure Pin

`cd /home/pin-3.13`

set `pin_root` in `/home/scripts/pin_build_tool` and `pin_run_tool` to `/home/pin-3.13` and then run: 

`pin_build_tool`

### 2.3 Set up and configure Dyninst

`cd /home/dyninst-10.1.0`

Open `instructionAPI/h/InstructionCategories.h` file and add `c_LoadInsn` and `c_StoreInsn` in `enum InsnCategory{}`

```
mkdir build
cmake . -DCMAKE_INSTALL_PREFIX=build
make install -j8
cp -r /home/dyn-cinnamon/MyDSLTool /home/dyninst-10.1.0/examples/
```

set `dyn_root` in `/home/scripts/dyn_build_tool` and `dyn_run_tool` to `/home/dyninst-10.1.0`

**Alternatively**

There might be issues building `Dyninst` from within the container such as `make install -j8 ` command taking prohibitbly long time. If you see any such issue,  you can try to download and build `Dyninst` outside the container. 

Get Dyninst v10.1.0

``wget https://github.com/dyninst/dyninst/archive/v10.1.0.tar.gz``

 ``tar xzvf v10.1.0.tar.gz``
 
 You can get the relevant files for `Dyninst`  with placeholders, templates and utility libraries for `Cinnamon` from https://github.com/mahwisharif/dyn-cinnamon.git
 
 ``git clone https://github.com/mahwisharif/dyn-cinnamon``
 
 Copy ``MyDSLTool`` folder from ``dyn-cinnamon`` under ``your-path-to-dyninst-10.1.0-root-dir/examples``
 
`` cp -r your-path-to-dyn-cinnamon/MyDSLTool your-path-to-dyninst-10.1.0-root-dir/examples/``

Rest of the instructions to build or run would remain the same. You will just need to set the paths in relevant files and places accordingly. 

### 2.4 Set up and configure Cinnamon

`cd /home/Cinnamon`

in `Scripts/compileToJanus.py` set `JanusPATH=/home/Janus`, set `ParserPATH=/home/Cinnamon`

in `Scripts/compileToPin.py` set `PinPATH=/home/pin-3.13`, set `ParserPATH=/home/Cinnamon`

in `Scripts/compileToDyn.py` set `DynPATH=/home/dyninst-10.1.0`, set `ParserPATH=/home/Cinnamon`


## 3. Building and running tools with Cinnamon

### 3.1 Build & run a tool with cinnamon targetting Janus
```
cd /home/Cinnamon
make TARGET=janus
compileToJanus <tests/ins.dsl>
janus_build_tool
jdsl_run <target_binary>
```

If using spec benchmarks, add this line in your `.cfg` file

`submit = jdsl_run $command`

### 3.2 Build & run a tool with cinnamon targetting Pin
```
cd /home/Cinnamon
make TARGET=pin
compileToPin <tests/ins.dsl>
pin_build_tool
pin_run_tool <target_binary>
```



If using spec benchmarks, add this line in your `.cfg` file

`submit = pin_run_tool $command`


### 3.3 Build and run a tool with cinnamon targetting Dyninst
```
cd /home/Cinnamon
make TARGET=dyn
compileToDyn <tests/ins.dsl>
dyn_build_tool
dyn_run_tool <target_binary>
```

If using spec benchmarks, add this line in your `.cfg` file

`submit = dyn_run_tool $command`



**NOTE**: If you just want to compile the cinnamon program e.g. `ins.dsl` and not yet intergate in target framework, run the following command from `/home/Cinnamon` after you have run the  build script for the respective target. This will generate a number of different files containing relevant code for the target framework:
```
$ ./bdc tests/ins.dsl
```
