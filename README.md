sudo singularity pull --name cinn.simg shub://mahwisharif/singularity-cinnamon
sudo singularity shell cinn.simg


once inside
ln -s /usr/bin/clang++-3.8 /usr/bin/clang++
ln -s /usr/bin/clang-3.8 /usr/bin/clang

all directories copied under
cd /home

cd /home/Janus

set janus_root in /home/scripts/janus_build_tool to /home/Janus and then run:

/home/scripts/janus_build_tool

cd /home/pin-3.13
set path in /home/scripts/pin_build_tool and pin_run_tool for pin_root to /home/pin-3.13 and then run: 
/home/scripts/pin_build_tool

cd /home/dyninst-10.1.0
c_LoadInsn and c_StoreInsn in instructionAPI/h/InstructionCategories.h in enum InsnCategory
mkdir build
cmake . -DCMAKE_INSTALL_PREFIX=build
make install -j8
cp -r /home/dyn-cinnamon/MyDSLTool /home/dyninst-10.1.0/examples/
set dyn_root in /home/scripts/dyn_build_tool and dyn_run_tool to /home/dyninst-10.1.0

cd /home/Cinnamon

in compileToJanus.py set JANUS_PATH=/home/Janus
in compileToPin.py set PIN_PATH=/home/pin-3.13
in compileToDyn.py set DYNINST_PATH=/home/dyninst-10.1.0

export PATH=/home/scripts:/home/Janus/janus:$PATH

make TARGET=janus
./compileToJanus <test.dsl>
janus_build_tool
/home/Janus/janus/jdsl_run <target_binary>

make TARGET=pin
./compileToPin <test.dsl>
pin_build_tool
pin_run_tool <target_binary>

make TARGET=dyn
./compileToDyn <test.dsl>
dyn_build_tool
dyn_run_tool <target_binary>

If using spec benchmarks: 
use 
submit = pin_run_tool $command 
