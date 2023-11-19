# meta-ide-sdk

This is a temporary preview for the devtool ide-sdk plugin.
This will either find its way upstream into poky or move to its own layer in <https://github.com/siemens>

## Bootstrap the eSDK

```sh
git clone --recurse-submodules git@github.com:afreof/meta-ide-sdk.git
cd meta-ide-sdk
. oe-init-build-env
```

## Working on a recipe's source code

### Create the workspace for the recipe and build the SDK

devtool modify cmake-example
devtool modify meson-example

devtool ide-sdk cmake-example meson-example core-image-minimal -c --debug-build-config

### Start Qemu

```sh
runqemu nographic
```

### Working with VSCode

The default IDE is VSCode.
Open VSCode in the workspace folder of the recipe: ``$BUILDIR/workspace/sources/cmake-example``

- To work with CMake press ``Ctrl + Shift + p``, type ``cmake``.
  This will show some possible commands like selecting a CMake preset, compiling or running CTest.
  Cross-compiled unit tests are executed transparently with qemu-user.

- To work with Meson press ``Ctrl + Shift + p``, type ``meson``.
  This will show some possible commands like compiling or executing the unit tests.

- For the deployment to the target device, just press ``Ctrl + Shift + p``, type ``task``.
  Select the ``install & deploy task``.

- For remote debugging, switch to the debugging view by pressing the "play" button with the ``bug icon`` on the left side.
  This will provide a green play button with a drop-down list where a debug configuration can be selected.
  After selecting one of the generated configurations, press the "play" button.

  Starting a remote debugging sesssion automatically initiates the deployment to the target device.
  If this is not desired, the ``"dependsOn": ["install && deploy-target...]`` parameter of the tasks
  with ``"label": "gdbserver start...`` can be removed from the ``tasks.json`` file.

### Using another IDE

Additionally ``--ide=none`` is supported.
With the none IDE parameter some generic configurations files like ``.gdbinit`` files and some helper scripts
starting the gdbserver remotely on the target device as well as the gdb client on the host are generated.

## Working on source code without using a recipe

For some special recipes and use cases a per-recipe sysroot based SDK is not suitable.
Therefore ``devtool ide-sdk`` also supports setting up the shared sysroots environment and generating
IDE configurations referring to the shared sysroots. Recipes leading to a shared sysroot
are for example ``meta-ide-support`` or ``shared-sysroots``.
Also passing ``none`` as a recipe name leads to a shared sysroot SDK:

### Build the SDK

```sh
devtool ide-sdk none core-image-minimal
```

### Working with VSCode and CMake

For VSCode the cross-tool-chain is exposed as a CMake kit. CMake kits are defined in
``~/.local/share/CMakeTools/cmake-tools-kits.json``.
The following example shows how the cross-toolchain can be selected in VSCode.
Fist of all we need a folder containing a CMake project.
For this example lets create a CMake project and start VSCode::

```sh
mkdir kit-test
echo "project(foo VERSION 1.0)" > kit-test/CMakeLists.txt
code kit-test
```

If there is a CMake project in the workspace cross-compilation is supported:

- Press ``Ctrl + Shift + P``, type ``CMake: Scan for Kits``
- Press ``Ctrl + Shift + P``, type ``CMake: Select a Kit``

Finally most of the features provided by CMake and the IDE should be available.

### Using another IDE without a recipe

For other IDEs than VSCode ``devtool ide-sdk none ...`` is just a simple wrapper for the
setup of the extensible SDK as described in :ref:`setting_up_ext_sdk_in_build`.

```sh
. oe-init-build-env
devtool modify cmake-example
devtool ide-sdk cmake-example core-image-minimal -c --debug-build-config --ide=none
runqemu nographic
```

```sh
# Find cmake-native and save path into variable
grep cmakeExecutable CMakeUserPresets.json

CMAKE_NATIVE="$BUILDDIR/tmp/work/cortexa57-poky-linux/cmake-example/1.0/recipe-sysroot-native/usr/bin/cmake"
"$CMAKE_NATIVE" --list-presets
"$CMAKE_NATIVE" --build --preset cmake-example-cortexa57
"$CMAKE_NATIVE" --build --preset cmake-example-cortexa57 --target clean
"$CMAKE_NATIVE" --build --preset cmake-example-cortexa57 --target all
"$CMAKE_NATIVE" --build --preset cmake-example-cortexa57 --target test
```

Deploy application and start gdbserver on the remote target device

```sh
../../temp/cmake-example/install_and_deploy_cmake-example-cortexa57
../../temp/cmake-example/gdbserver_start_1234_-usr-bin-cmake-example_m
```

Start gdb on the host

```sh
GDB_NATIVE=/home/adrian/projets/oss/meta-sdk-ide/build/tmp/work/x86_64-linux/gdb-cross-aarch64/13.2/recipe-sysroot-native/usr/bin/aarch64-poky-linux/aarch64-poky-linux-gdb
"$GDB_NATIVE" -x gdbinit-usr-bin-cmake-example
run
break main
run
step
stepi
continue
```
