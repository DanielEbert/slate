---
title: API Reference

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

# TODO: include errors when e.g. elf symbol cant be found

search: true

code_clipboard: true
---

# Introduction

In this tutorial you will learn how to fuzz test software that runs in an emulator.

If you are unfamiliar with fuzzing, you can read section 2.1 of this TODO paper.

Prerequisites: Beginner-level experience with C/C++ and the Unix shell.

TODO: check service up and running?

TODO: reference relevant sections of my paper

All following paths are relative to the base directory: TODO

# Exercises 

## Hello World 

~~~bash
cd TestPrograms/hello_world/
cat src/src.ino
~~~
~~~c
void setup() {
  Serial.begin(9600);
  Serial.println("Hello World");
}

void loop(){}
~~~

> Compile the program above:

~~~
arduino-cli compile -b arduino:avr:mega:cpu=atmega2560 ./src --output-dir .
~~~

> In the previous command, './src' is a directory that contains the source code file 'src.ino'. '--output-dir .' specifies that the compiled executable 'src.ino.elf' is created in the current working directory. <br>
> Emulate the compiled ELF executable:

~~~
emu src.ino.elf
~~~

> You should see the following output on the terminal:

~~~
Could not connect to 127.0.0.1:8123
Using no patches.
Hello World..
~~~

This exercise introduces you to the process of compiling and emulating. 

<ol>
<li>Write a simple arduino program that prints 'Hello World' via Serial. You can also use the example program on the right.</li>
<li>Compile the program. One option is to compile via the arduino-cli. Compile for the 'atmega2560 board'. The emulator uses the 'atmega2560' board.</li>
<li>Emulate the program. 'emu' is an alias for starting the emulator. The only required command line argument is the path to an ELF file that you want to emulate.</li>
<li>You should see 'Hello World..' printed to the terminal.</li>
<li>Arduino programs run forever. The 'loop()' function is repeatedly called. Exit the emulation via &lt;CTRL&gt; + C</li>
</ol>


## PlatformIO Unit Tests

~~~bash
cd TestPrograms/exercise_unit_test/
~~~

> 'tree' shows the contents of the current directory as a tree. The output includes hidden files.

~~~bash
tree . -a -C
ls -la
~~~

> Running the unit tests should produce an output like this:

~~~bash
test/test.cpp:12:test_add:PASS
-----------------------
1 Tests 0 Failures 0 Ignored
OK
~~~

We can also run unit tests via PlatformIO's Unit Testing Framework. Compared to arduino-cli, PlatformIO uses a different workflow to compile binaries. 

This exercise shows you an example PlatformIO project setup, how PlatformIO-based projects are compiled, and how PlatformIO-based unit tests are run.

<ol>
<li>Take a look at the files in the 'TestPrograms/exercise_unit_test/' directory and its subdirectories.
<li>Can you already see how PlatformIO-based Unit Tests are compiled and emulated? Compile and run the unit tests in the emulator. You do NOT have to write source code. The links below may help you. You can click on the 'Hint ...' buttons below to display hints.</li>
<li>Exit the emulation via &lt;CTRL&gt; + C</li>
</ol>

**Links** <br>
[platformio.ini](https://docs.platformio.org/en/latest/projectconf/index.html) <br>
[Makefile](https://en.wikipedia.org/wiki/Make_%28software%29#Behavior) <br>
[src Directory](https://docs.platformio.org/en/latest/projectconf/section_platformio.html#src-dir) <br>
[test Directory](https://docs.platformio.org/en/latest/projectconf/section_platformio.html#projectconf-pio-test-dir) <br>
[Directory for compiled binaries](https://docs.platformio.org/en/latest/projectconf/section_platformio.html#projectconf-pio-workspace-dir) <br>
[PlatformIO Unit Testing Workflow](https://docs.platformio.org/en/latest/plus/unit-testing.html#workflow) <br>

<details style="margin: 20px; width: 48%">
  <summary>Hint 1 for compiling</summary>
<p style="margin: 10px">In the previous Hello World exercise, we used 'arduino-cli compile...' to compile the source code to a ELF binary. PlatformIO uses a different command line tool to do this. PlatformIO's tool is executed via 'pio'. The pio command to compile the unit tests program is in one of the files in the 'exercise_unit_test/' directory.</p>
</details>

<details style="margin: 20px; width: 48%">
  <summary>Hint 2 for compiling</summary>
<p style="margin: 10px">The Makefile includes the command to compile the project.</p>
</details>

<details style="margin: 20px; width: 48%">
  <summary>Hint 3 for compiling</summary>
<p style="margin: 10px">You can run 'make' in the 'exercise_unit_test' directory. 'make' will run the command to compile.</p>
</details>

<details style="margin: 20px; width: 48%">
  <summary>Hint 1 for emulating</summary>
<p style="margin: 10px">The compiled binary ELF file is in a hidden directory.</p>
</details>

<details style="margin: 20px; width: 48%">
  <summary>Hint 2 for emulating</summary>
<p style="margin: 10px">The path to the compiled binary ELF file is <br>'exercise_unit_test/.pio/build/megaatmega2560/firmware.elf'</p>
</details>

<details style="margin: 20px; width: 48%">
  <summary>Hint 3 for emulating</summary>
<p style="margin: 10px">Emulate an ELF file via: 'emu path/to/ELFfile'</p>
</details>

<details style="margin: 20px; width: 48%">
  <summary>Click here for a possible solution</summary>
<p style="margin: 10px">To compile the unit tests, run 'pio test --without-uploading --without-testing -v' in the 'exercise_unit_test' directory. Alternatively, we can also run 'make' in this directory. make will run the same 'pio test ...' command. By default, 'pio test' will upload the compiled machine code to a microcontroller board and run the unit tests on this board. We do not want this to happen. Thus, we disable this default behavior with the '--without-uploading --without-testing' command line flags. The '-v' command line flag enables verbose output. This output also includes the location of the compiled binary file, which is '.pio/build/megaatmega2560/firmware.elf'. We emulate this program with 'emu .pio/build/megaatmega2560/firmware.elf'.</p>
</details>

## Patches

~~~bash
cd TestPrograms/exercise_jump_foo/
make
emu .pio/build/megaatmega2560/firmware.elf
~~~

> Open a second Terminal.

~~~bash
cd patches
make compile=example1.c
~~~

> (Back in the first Terminal) Specify via C API to print 'Hello' before the SUT calls the setup() function: 

~~~bash
LD_PRELOAD=../../patches/example1.c.so emu .pio/build/megaatmega2560/firmware.elf
~~~

> Expected output (Note the 'Hello' in the second line):

~~~
filename: .pio/build/megaatmega2560/firmware.elf
Hello
test/test.cpp:19:test_parse:PASS.
.
-----------------------.
1 Tests 0 Failures 0 Ignored .
OK.
~~~


Take a look at the src/main.cpp and test/test.cpp files in the 'TestPrograms/exercise_jump_foo/' directory. Get a quick overview of what the program is doing. After that, compile and run the unit test in the emulator. Exit the emulation via &lt;CTRL&gt; + C.

The emulator provides a C API to execute user-specified code when the SUT is about to execute a particular instruction.
This C API is accessed via C code. Open a new terminal and 'cd' into the 'patches' directory. In this directory, there is a file called 'example1.c'. Read the content of this file.

The API to specify user-specified code is patch_instruction(uint32_t address, void *function_pointer, void *function_argument);.
The 'address' argument is the address of an instruction in the SUT's program space. 
Prior to emulating this instruction, the emulator calls the function pointed to by 
the 'function_pointer' argument. Thereby the 'function_argument' argument is passed 
as a parameter to the 'function_pointer' function.
The get_symbol_address() function returns the address of an ELF symbol with a given name.
The 'avr' struct includes references (i.e. pointers) to emulator-internal data structures
such as the (virual) RAM of the emulated program and to fuzzer-internal data structures
such as the generated input.
In this example, it is the ELF symbol for the 'setup()' function.
The print_hello() function is called before the emulator emulates the setup() function.

In the patches directory, run 'make compile=example1.c' to compile the file to a shared object 'example1.c.so'. 'Enable' this file via the LD_PRELOAD environment variable like this: In the TestPrograms/exercise_jump_foo/ directory: 'LD_PRELOAD=../../patches/example1.c.so emu .pio/build/megaatmega2560/firmware.elf'. The output should include the text 'Hello'.


## Introduction to Fuzzing

> Compile the shared object:

~~~bash
cd patches
make compile=exercise_parse.c
~~~

> Remove SUT code that overwrites fuzz_input and fuzz_input_length, then recompile and run:

~~~bash
cd TestPrograms/exercise_jump_foo
make
LD_PRELOAD=../../patches/exercise_parse.c.so emu .pio/build/megaatmega2560/firmware.elf
~~~

The goal of this exercise is to convert the unit test from the previous exercise to a fuzz test and fuzz the program in the emulator.

At the moment, the parse(char &#x2A;input, uint16_t input_length) function is tested with the string 'Hi?'. In this exercise, we want to test this parse function with 'random' inputs. The goal is to find an input that crashes the software under test (SUT) via fuzzing.

The fuzzer can generate inputs for the parse function. We need to replace the fuzz_input string and the fuzz_input_length of this input with the input and input length data from the fuzzer. The fuzzer runs 'outside' of the emulation. We must specify where the fuzzer should write the input. To enable this, the fuzzer provides a C API which we can use to override the content of the fuzz_input and fuzz_input_length variables with the generated input from the fuzzer at runtime.

We could write our own function that copies the input and length from the fuzzer to the fuzz_input and fuzz_input_length variables of the SUT. However, there is a function from the C API to do exactly that: write_fuzz_input_global. This function requires the 'avr' argument via function parameters. For example, if we want to write the fuzz_input and fuzz_input_length variables when the SUT is about to call the setup function, we can call the patch_instruction function like this: 'patch_instruction(get_symbol_address("setup", avr), write_fuzz_input_global, avr);'. If you are interested in the internals of these functions, you find the implementation for the write_fuzz_input_global function in the 'simavr/simavr/sim/fuzz_patch_instructions.c' file.

The unit test calls the parse function once. We want to repeatetly call the parse function. The user API function fuzz_reset restarts the emulated program and generates a new input. We can reset the SUT after the RUN_TEST(test_parse); function returns, i.e. when UNITY_END() is called. UNITY_END is a compiler macro for UnityEnd ('#define UNITY_END() UnityEnd()'). This means there is no 'UNITY_END' ELF symbol. We must use the 'UnityEnd' ELF symbol instead to get the symbol address.

In addition, we need to remove the SUT code that writes to the fuzz_input and fuzz_input_length variables. To do this, delete all lines from 'fuzz_input[0] = 'H';' to 'fuzz_input_length = 3;' in the 'TestPrograms/exercise_jump_foo/test/test.cpp' file. Recompile the unit test source code to update the ELF executable.

A C API file for the parse function with the write_fuzz_input_global and fuzz_reset patches is 'patches/exercise_parse.c'. Compile this file to a shared object and run the unit test in the emulator with this shared object enabled. If that was successful, quit the emulator and move on to the next exercise below.

## Fuzzing Statistics

> Open a new Terminal and run the following to start the UI server:

~~~bash
cd server
./main_loop
~~~

> Start the emulator with exercise_parse.c.so shared object and the TestPrograms/exercise_jump_foo program.

~~~bash
cd TestPrograms/exercise_jump_foo
LD_PRELOAD=../../patches/exercise_parse.c.so emu .pio/build/megaatmega2560/firmware.elf
~~~

> Examine the files in the current\_run directory.

The fuzzer can send updates about the current fuzzing campaign to a UI server. Open a new Terminal and run 'server/main_loop' to start this UI server.

Start the emulator with exercise_parse.c.so shared object and the TestPrograms/exercise_jump_foo program.

The UI server creates the following directory structure:

Path | Description
- | -
current\_run/ | Information about the current or most recent fuzzing run.
current\_run/crashing\_inputs/ | Whenever a sanitizer or emulator detects a bug or crash, two files are created in this directory. The name of two files are for example 'stack\_buffer\_overflow\_200c' and 'stack\_buffer\_overflow\_200c\_info'. 'stack\_buffer\_overflow\_200c' contains the input that crashed the SUT. '\*\_info' contains the stack trace at the time when the crash or bug is detected.
current\_run/coverage.html | Coverage Explorer that show the SUT source code. Open this file in your browser. Executed lines have a green background. Have a look at the 'src/main.cpp' file (current_run/coverage.home_user_EFF_TestPrograms_delme2_src_main.cpp.html).
current\_run/coverage\_over\_time\_plot.png | Edge coverage over time graph.
current\_run/fuzer\_stats | Statistics about the fuzzing run, e.g. fuzzer runtime and number of bugs found.
current\_run/previous\_interesting\_inputs/ | Directory that contains all inputs that increased the code coverage.
previous\_runs/ | If you restart the emulator, the current\_run directory is moved into the previous\_runs/ directory.

# Emulator/Fuzzer Command Line Arguments and Environment Variables

Names that start with '--' are command line arguments. Otherwise, it is an environment variable. 

Name | Description
- | - | - | -
LD\_PRELOAD | Path to your compiled software under test configuration file.
--seeds | Path to a directory. For every file in this directory, the content of this file is one seed.
--run\_once\_with | Path to a file. Runs the emulator once/ The input is the content of the specified file.
--timeout | One input times out when more than the specified 'timeout' number of clock cycles has passed since the emulator started or reset.
--dont\_report\_timeouts | This is a flag. If this flag is not set, a timeout is treated as a bug, which means that the input that timed out is stored. If this flag is set, timeouts are not treated as bugs.
--mutator\_so\_path | When you want to use another mutator. The default mutator is the mutator from libFuzzer.
