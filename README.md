`sudo singularity pull --name cinn.simg shub://mahwisharif/singularity-cinnamon`

`sudo singularity shell cinn.simg`

## Configuration 

once inside the singularity shell. You will need to set the following configurations. 

`ln -s /usr/bin/clang++-3.8 /usr/bin/clang++`

`ln -s /usr/bin/clang-3.8 /usr/bin/clang`

all directories were cloned and copied under `/home`. 

`export PATH=/home/scripts:/home/Janus/janus:/home/Cinnamon/Scripts:$PATH`

### Set up and configure Janus

`cd /home/Janus`

set `janus_root` in `/home/scripts/janus_build_tool` to `/home/Janus` and then run:

`janus_build_tool`

### Set up and configure Pin

`cd /home/pin-3.13`

set `pin_root` in `/home/scripts/pin_build_tool` and `pin_run_tool` to `/home/pin-3.13` and then run: 

`pin_build_tool`

### Set up and configure Dyninst
`cd /home/dyninst-10.1.0`

open `instructionAPI/h/InstructionCategories.h` file and add `c_LoadInsn` and `c_StoreInsn` in `enum InsnCategory{}`

`mkdir build`

`cmake . -DCMAKE_INSTALL_PREFIX=build`

`make install -j8`

`cp -r /home/dyn-cinnamon/MyDSLTool /home/dyninst-10.1.0/examples/`

set `dyn_root` in `/home/scripts/dyn_build_tool` and `dyn_run_tool` to `/home/dyninst-10.1.0`


### Set up and configure Cinnamon

`cd /home/Cinnamon`

in `Scripts/compileToJanus.py` set `JANUS_PATH=/home/Janus`

in `Scripts/compileToPin.py` set `PIN_PATH=/home/pin-3.13`

in `Scripts/compileToDyn.py` set `DYNINST_PATH=/home/dyninst-10.1.0`


## Building and running tools with Cinnamon 

### Build & run a tool with cinnamon targetting Janus
`cd /home/Cinnamon`

`make TARGET=janus`

`compileToJanus <test.dsl>`

`janus_build_tool`

`jdsl_run <target_binary>`

If using spec benchmarks, add in your `.cfg` file

`submit = jdsl_run $command`

### Build & run a tool with cinnamon targetting Pin
`cd /home/Cinnamon`

`make TARGET=pin`

`compileToPin <test.dsl>`

`pin_build_tool`

`pin_run_tool <target_binary>`

If using spec benchmarks, add in your `.cfg` file

`submit = pin_run_tool $command`

### Build and run a tool with cinnamon targetting Dyninst
`cd /home/Cinnamon`

`make TARGET=dyn`

`compileToDyn <test.dsl>`

`dyn_build_tool`

`dyn_run_tool <target_binary>`

If using spec benchmarks, add in your `.cfg` file

`submit = dyn_run_tool $command`
