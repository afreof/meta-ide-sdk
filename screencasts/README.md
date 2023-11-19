# Screencasts

## devtool-ide-sdk-recipe-debug

Demo of a remote debug session.

1. Setup the eSDK
2. Start another VSCode instance in the workspace of the modified recipe
3. Run a remote debug session
4. Change a variable
5. Re-compile, deploy
6. Run a remote debug session with modified source code

## devtool-ide-sdk-recipe-unit-test

Demo of executing Unit Tests which are cross-compiled for a different CPU architecture.
The result is integrated with the Test View of VSCode.
CMake uses Qemu from native sysroot to execute the test binaries transparently.

## devtool-ide-sdk-recipe-ide-none

Demo of devtool ide-sdk with a recipe and ide=none.
Remote debiggung from command line.
