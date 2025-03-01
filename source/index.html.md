---
title: API Reference

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

search: true

code_clipboard: true
---

# Introduction

In this tutorial you will learn how to fuzz test software that runs in an emulator.

If you are unfamiliar with mutation-based fuzzing, I would recommend that you read section 2.1 of the paper located here: /home/user/Exercises/paper.pdf to get an idea of what fuzzing is.

Prerequisites: Beginner-level experience with C/C++ and the Unix shell.

All following paths are relative to the base directory: ~/Exercises

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
<li>Write a simple Arduino program that prints 'Hello World' via Serial. You can also use the example program on the right.</li>
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

We can also run unit tests via PlatformIO's Unit Testing Framework. Compared to arduino-cli, PlatformIO uses a different workflow to compile source code. 

This exercise shows you an example PlatformIO project setup, how PlatformIO-based projects are compiled, and how PlatformIO-based unit tests are run.

<ol>
<li>Look at the files in the 'TestPrograms/exercise_unit_test/' directory and its subdirectories.
<li>Can you already see how PlatformIO-based Unit Tests are compiled and emulated? Compile and run the unit tests in the emulator.</li>
<li>Exit the emulation via &lt;CTRL&gt; + C</li>
</ol>

<aside class="notice">
The links below may help you. You can click on the 'Hint ...' buttons below to display hints.
</aside>

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
<p style="margin: 10px">Emulate an ELF file via: 'emu PATH/TO/FILE.elf'</p>
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


<ol>
<li>Look at the src/main.cpp and test/test.cpp files in the 'TestPrograms/exercise_jump_foo/' directory. Get a quick overview of what the program is doing.</li>
<li>After that, compile and run the unit test in the emulator.</li>
<li>Exit the emulation via &lt;CTRL&gt; + C.</li>
</ol>

The emulator provides a C API where a user can specify that the emulator should execute additional user-specified code when the emulator is about to emulate a particular function. This C API is accessed via C code. Open a new terminal and 'cd' into the '~/Exercises/patches' directory. In this directory, there is a file called 'example1.c'. 

With example1.c, when the emulated program calls the setup() function, the emulator calls the print_hello() function. The print_hello() function runs 'outside' of the emulation. Take a quick look at the example1.c code. For these exercises, you don't need to understand how the `patch_function()` function works.

<details style="margin: 20px; width: 48%">
  <summary>Optional reading - Click to show how more information about the C API</summary>
<p style="margin: 20px">Understanding how the functions from the C API work is optional. The reason why we need this API will also only make sense after the next exercise 'Introduction to Fuzzing'. <br>

The API to specify user-specified code is via the patch_function function: <br>

`patch_function(char *function_name, void *function_pointer, void *function_argument, avr_t *avr);` <br></p>

<table style="width: 98%;">
<tbody>
<tr>
<td>&nbsp;Argument</td>
<td>&nbsp;Type</td>
<td>&nbsp;Description</td>
</tr>
<tr>
<td>&nbsp;function_name</td>
<td>&nbsp;char*</td>
<td>&nbsp;A function of the program that the emulator is emulating. For example, the setup() or the loop() function.</td>
</tr>
<tr>
<td>&nbsp;function_pointer</td>
<td>&nbsp;void*</td>
<td>&nbsp;Prior to emulating the call of the function 'function_name', the emulator calls the function pointed to by the 'function_pointer' argument.</td>
</tr>
<tr>
<td>&nbsp;function_argument</td>
<td>&nbsp;void*</td>
<td>&nbsp;Thereby the 'function_argument' argument is passed as a parameter to the 'function_pointer' function.</td>
</tr>
<tr>
<td>&nbsp;avr</td>
<td>&nbsp;avr_t*</td>
<td>&nbsp;This is always the 'avr' struct pointer that is passed to the setup_patches function as an argument. The patch_function implementation requires this.</td>
</tr>
</tbody>
</table>

<p style="margin: 20px">The 'avr' struct includes references (i.e. pointers) to emulator-internal data structures
such as the (virtual) RAM of the emulated program and to fuzzer-internal data structures
such as the generated input.<br>
</details>

<ol>
<li>In the patches directory, run 'make compile=example1.c' to compile the example1.c file to a shared object 'example1.c.so'.</li>
<li>'Enable' this file via the LD_PRELOAD environment variable like this: In the TestPrograms/exercise_jump_foo/ directory: 'LD_PRELOAD=../../patches/example1.c.so emu .pio/build/megaatmega2560/firmware.elf'. The output should include the text 'Hello'.</li>
<li>Exit the emulation via &lt;CTRL&gt; + C</li>
</ol>


## Introduction to Fuzzing

> Remove SUT code that overwrites fuzz_input and fuzz_input_length, then recompile:

~~~bash
cd TestPrograms/exercise_jump_foo
make
~~~

> Compile the shared object:

~~~bash
cd patches
make compile=exercise_parse.c
~~~

> Run the fuzz test in the emulator (with the shared object enabled):

~~~bash
cd TestPrograms/exercise_jump_foo
LD_PRELOAD=../../patches/exercise_parse.c.so emu .pio/build/megaatmega2560/firmware.elf
~~~

The goal of this exercise is to convert the unit test from the previous exercise to a fuzz test and fuzz the program in the emulator.

In fuzzing, the software under test (SUT) is **repeatedly** executed with **different inputs from the fuzzer**. In its current state, the SUT (i.e. the `parse(char *input, uint16_t input_length)` function) is executed **once** with an **input from the unit test** (i.e. the string "Hi?"). Thus, we need to:
<ol>
<li>Replace the "Hi?" string in the fuzz_input variable and the fuzz_input_length value with the input and input length from the fuzzer.</li>
<li>Repeatedly call the parse function.</li>
</ol>

The fuzzer runs outside of the emulation. We must specify where the fuzzer should write the input and its length. To do this, we can use the C API call `patch_function("setup", write_fuzz_input_global, avr, avr);`. Compared to the patch_function() call from the previous exercise, the write_fuzz_input_global() function is called instead of the print_hello() function. write_fuzz_input_global() is provided by the C API and when called it will override the content of the fuzz_input and fuzz_input_length variables with the generated input and its length from the fuzzer.

The unit test calls the parse function once. We want to repeatedly call the parse function. The user API function `fuzz_reset` restarts the emulated program and generates a new input. We can reset the SUT after the test_parse function returns, for example when the UnityConcludeTest function is called. UnityConcludeTest() is called shortly after each unit test returns. The C API call for this is `patch_function("UnityConcludeTest", fuzz_reset, avr, avr);`.

In addition, we need to remove the SUT code that would otherwise overwrite the fuzz_input and fuzz_input_length variables. This is your task: 

<ol>
<li>Delete all lines from 'fuzz_input[0] = 'H';' to 'fuzz_input_length = 3;' in the 'TestPrograms/exercise_jump_foo/test/test.cpp' file.</li>
<li>Recompile the unit test source code to update the ELF executable.</li>
<li>There is already a C API file with the two patch_function calls shown above in 'patches/exercise_parse.c'. Examine this file, compile this file to a shared object, and run the fuzz test in the emulator with this shared object enabled.
<li>If that was successful, exit the emulation via &lt;CTRL&gt; + C</li>
</ol>

## Fuzzing Statistics

> Open a new Terminal and run the following to start the UI server:

~~~bash
cd server
./main_loop
~~~

> Start the emulator with the exercise_parse.c.so shared object and the TestPrograms/exercise_jump_foo program with a maximum input length of 6.

~~~bash
cd TestPrograms/exercise_jump_foo
LD_PRELOAD=../../patches/exercise_parse.c.so emu --max_input_length 6 .pio/build/megaatmega2560/firmware.elf
~~~

> Examine the files in the current\_run directory.

The fuzzer can send updates about the current fuzzing campaign to a UI server. Open a new Terminal and run 'cd ~/Exercises/server/ && ./main_loop' to start this UI server.

Start the emulator with the exercise_parse.c.so shared object and the TestPrograms/exercise_jump_foo program. Additionally, specify a maximum input length of 6 via the 'emu' command line argument --max_input_length like so: 'LD_PRELOAD=../../patches/exercise_parse.c.so emu --max_input_length 6 .pio/build/megaatmega2560/firmware.elf'. The default maximum input length is 128. In this exercise, setting this maximum to 6 increases the code coverage faster.

The UI server creates the following directory structure:

Path | Description
- | -
current\_run/ | Information about the current or most recent fuzzing run.
current\_run/crashing\_inputs/ | Whenever a sanitizer or emulator detects a bug or crash, two files are created in this directory. The name of two files is for example 'stack\_buffer\_overflow\_200c' and 'stack\_buffer\_overflow\_200c\_info'. 'stack\_buffer\_overflow\_200c' contains the input that crashed the SUT. '\*\_info' contains the stack trace at the time when the crash or bug is detected.
current\_run/coverage.html | Coverage Explorer that show the SUT source code. Open this file in your browser. Executed lines have a green background. Have a look at the 'src/main.cpp' file (current_run/coverage.home_user_EFF_TestPrograms_delme2_src_main.cpp.html).
current\_run/coverage\_over\_time\_plot.png | Edge coverage over time graph.
current\_run/fuzer\_stats | Statistics about the fuzzing run, e.g. fuzzer runtime and number of bugs found.
current\_run/previous\_interesting\_inputs/ | Directory that contains all inputs that increased the code coverage.
previous\_runs/ | If you restart the emulator, the current\_run directory is moved into the previous\_runs/ directory.

You don't have to do this, but if you were tasked to fuzz test a large code base, one goal is to reach a high code coverage. The primary goal of the fuzzer is to find bugs. But if your fuzzer did not find any bugs, how do you know how the fuzzer is doing? Code coverage can be an indication for how well the fuzzer is doing. High code coverage is beneficial in fuzzing, because a fuzzer can only find bugs in code that is executed and because there is strong empirical evidence that a fuzzer with higher code coverage discovers more bugs in the SUT in a given time window. If for example the fuzzer has not increased the code coverage in some time, a fuzz tester can help the fuzzer in reaching a higher code coverage faster. Different fuzzers provide different ways of how this can be achieved. Typical options include for example creating seeds for the mutator (we'll do this later) and updating or adding a new entrypoint for the inputs from the fuzzer (e.g. a new 'unit' test that calls a function that previously was not covered/tested by the fuzzer). 

Due to the random nature of fuzzing, the fuzzer can find an input that crashes the SUT sometimes in 2 minutes and sometimes it takes 20 minutes. If your fuzzer instance did not find the bug by now, its OK. Quit the emulator and move on to the next exercise.

<aside class="notice">
You do NOT have to restart the UI server. If you stop and start a new emulator process, the UI server will create a new current_run directory for you. The UI server supports one concurrent connection. You can run multiple emulator processes concurrently, but the UI server will only create the current_run directory for the emulator process that was started first. Because the fuzzer does not stop, if you are finished with an exercise, you should manually stop the emulator process so that the fuzzer run for the following exercise updates the current_run directory.
</aside>

## Fuzzing a SUT with strcpy

Your task is to fuzz test the process_input function in the TestPrograms/exercise_strcpy project. The fuzzer should quickly trigger a buffer overflow.

<details style="margin: 20px; width: 48%">
  <summary>Hint 1</summary>
<p style="margin: 10px">We need two things: 1) A test for the process_input function in the test/test.cpp file. 2) A C API shared object file to 'inject' the input from the fuzzer and to reset the emulator when the test completed. Optionally you can also specify command line arguments for 'emu' such as the --max_input_length. The default --max_input_length is 128, which is a good value for this exercise. The fuzzer should trigger the bug in less than 30 seconds.</p>
</details>

<details style="margin: 20px; width: 48%">
  <summary>Hint 2</summary>
<p style="margin: 10px">Compare the (incomplete) source code from this exercise to the (complete) source code from the previous exercise. What is the same and what is different? We can reuse code from the previous exercise, for example the shared object for the C API and more.</p>
</details>

<details style="margin: 20px; width: 48%">
  <summary>Possible Solution</summary>
<p style="margin: 10px">A possible solution is in the 'TestPrograms/exercise_strcpy_solution' directory. A test was added in the test/test.cpp file that calls the process_input function with the global variable fuzz_input. You can reuse the shared object 'patches/exercise_parse.c.so' from the previous exercise. We need to let a --max_input_length greater than 40 to trigger the bug, so the default value of 128 is good. The command to run the emulator in this case is: 'LD_PRELOAD=../../simavr/simavr/sim/patches/exercise_parse.c.so emu .pio/build/megaatmega2560/firmware.elf'.</p>
<p style="margin: 10px">An alternative solution is to not use the unit testing framework. We could modify the src/main.cpp code and add a setup function that calls the process_input function. In this case the pio workflow changes, because we are no longer using the Unit Test Framework with 'pio test ...'. In larger projects where you maybe do not know the code base very well, it is often easier to 'convert' a unit test to a fuzz test, because the unit test describes the context in which a given function is typically called. For example, the unit test code may set up the environment so that the function that you want to test does not immediatetly return because some state is not setup or because the SUT developers made heavy use of <a href="https://en.wikipedia.org/wiki/Design_by_contract">design-by-contract</a>.</p>
</details>

## [Optional] Investigate bugs with a debugger 

> Run the emulator once with the content of file crashing_input as the input from the fuzzer:

~~~bash
cd TestPrograms/exercise_strcpy
LD_PRELOAD=../../patches/exercise_parse.c.so emu --run_once_with ../exercise_strcpy_solution/previous_run/crashing_inputs/stack_buffer_overflow_f50 .pio/build/megaatmega2560/firmware.elf
~~~

> Additionally, start the GDB server and wait for a remote to connect:

~~~bash
cd TestPrograms/exercise_strcpy
LD_PRELOAD=../../patches/exercise_parse.c.so emu --run_once_with ../exercise_strcpy_solution/previous_run/crashing_inputs/stack_buffer_overflow_f50 --gdb .pio/build/megaatmega2560/firmware.elf
~~~

> Start GDB

~~~bash
avr-gdb .pio/build/megaatmega2560/firmware.elf
~~~

> In avr-gdb, run the following to connect to the remote:

~~~bash
target remote :1234
break *0xf50
continue
info registers
x/10i $pc-10
~~~

If the fuzzer finds a bug and you cannot identify the reason for this bug via e.g. the stack trace, you may have to investigate the bug with a debugger. The emulator has support for the GDB debugger. In this exercise we will investigate the reason for the buffer overflow from the previous exercise.

The current\_run/crashing\_inputs/ from the previous exercise should include an input that triggered a buffer overflow in the TestPrograms/exercise_strcpy project. If the fuzzer did not find the buffer overflow, use the file stack_buffer_overflow_f50 in the TestPrograms/exercise_strcpy_solution/previous_run/crashing_inputs directory instead. This file contains an input that triggers this buffer overflow.

The emulator has a command line argument '--run_once_with X' where X is path to a file. With this option, the emulator runs once. The input is the content of the specified file X.

In addition, the emulator starts a GDB server when the emulator is run with the '--gdb' command line argument. The GDB server waits until a client connects and listens for commands from the client.

Your task: Run the emulator once with the content of file crashing_input as the input from the fuzzer and tell the emulator process to start a GDB server. (The command is on the right side.) The emulator output should include the text 'avr_gdb_init listening on port 1234'. 

You can now connect to the GDB server. First, opening a new terminal. Then, run the command 'avr-gdb X', where X is the path to the ELF executable that the emulator is currently emulating. In avr-gdb, connect via the gdb command 'target remote :1234'.

At this moment, the emulated program is stopped at the first instruction of this emulated program, and you can debug this program as if it is running locally. For example, we could set a breakpoint at the instruction where the crash is detected with 'break \*0xf50'. Then, we could continue the execution with the gdb command 'continue' in the avr-gdb terminal and output more information, for example the content of the registers with 'info registers' or examine the machine code instructions with 'x/10i $pc-10'. These machine code instructions are from the AVR instruction set architecture.

The buffer overflow occurs because strcpy writes more than 40 bytes to a buffer with a size of 40 bytes. The instruction at address 0xf50 overwrites the return address that is stored on the stack. The instruction at this address is 'st X+, r0', which stores the content of register r0 in X. X points to the return address on the stack. Overriding the return address is illegal.

## Fuzzing ArduinoJson

> Fuzzing with seeds example:

~~~bash
mkdir seeds
echo 'A' > seeds/1
LD_PRELOAD=../../patches/exercise_parse.c.so emu --seeds seeds .pio/build/megaatmega2560/firmware.elf
~~~

Examine the files in the TestPrograms/exercise_arduinojson directory. Write a fuzz test for the ArduinoJson library. You can, for example, convert the unit test in exercise_arduinojson/test/test.cpp file to a fuzz test.

<aside class="notice">
ArduinoJson is a widely used and well tested library. You will very likely not find bugs.
</aside>

<details style="margin: 20px; width: 48%">
  <summary>Hint 1</summary>
<p style="margin: 10px">If the input is randomly generated, assertions like 'TEST_ASSERT(ret == DeserializationError::Ok);' will not hold anymore for all possible inputs. With the LD_PRELOAD file exercise_parse.c.so the fuzzer does not report failed assertions. For this reason, failed assertions do not matter in this case. In practice, you would remove assertions in your unit test that do not hold for all possible generated inputs, and you would report inputs that trigger failed assertions.</p>
</details>

<details style="margin: 20px; width: 48%">
  <summary>Click here for a possible solution to the fuzz test</summary>
<p style="margin: 10px">One possible solution (many solutions are valid) is in the TestPrograms/exercise_arduinojson_solution directory. The input variable is replaced with a global fuzz_input variable. fuzz_input must be a global variable so that there is an ELF symbol for this variable. Via the ELF symbol, the emulator knows the constant address of the fuzz_input variable. strlen(input) is replaced with fuzz_input_length. We probably do not want to use strlen(fuzz_input) instead, because fuzz_input may not contain a '\0' character (you can fix this problem by explicitly setting the last character in the fuzz_input buffer to '\0') and calling strlen decreases the performance. This is less of a problem with maximum input lengths of 128 bytes, but consider fuzzing a desktop application that processes audio or video files, which can get very large. For LD_PRELOAD, the patches/exercise_parse.c.so shared object can be resued.</p>
</details>

[Optional] You can specify seeds for the fuzzer with the emulator command line flag '--seeds X' where X is the path to a directory. For every file in this directory, the content of this file is one seed. The mutator/fuzzer will modify seeds to generate new input. Good seeds will result in higher code coverage faster. Good seeds are often inputs that reach a higher code coverage. Think about where you can get seeds from.

<details style="margin: 20px; width: 48%">
  <summary>Solution - Click here to show possible sources for seeds</summary>
<ol>
<li>Google example inputs. For example there are Github repositories like <a href="https://github.com/dvyukov/go-fuzz-corpus/tree/master/json/corpus">this</a> that have seeds for fuzzing.</li>
<li>Reuse interesting inputs that the fuzzer found in a previous test of the same or a similar program. These are in the current_run/previous_interesting_inputs/ directory.</li>
<li>You can manually review the code and identify roadblocks. Roadblocks are input checks that no input from the fuzzer has passed yet. You can add inputs to the seeds that pass this roadblock.</li>
<li>There are also tools that generate inputs. For example concolic execution tools such as KLEE can output inputs that reach deeper paths in a program. There are fuzzers that have a built-in concolic execution tool. These are called hybrid fuzzers.</li>
</details>

# Emulator/Fuzzer Command Line Arguments and Environment Variables

Names that start with '--' are command line arguments. Otherwise, it is an environment variable. 

Name | Description
- | - | - | -
LD\_PRELOAD | Path to your compiled software under test configuration file.
--run\_once\_with | Path to a file. Runs the emulator once. The input is the content of the specified file.
--gdb | Enable built-in support for the GDB debugger.
--seeds | Path to a directory. For every file in this directory, the content of this file is one seed.
--timeout | One input times out when more than the specified 'timeout' number of clock cycles has passed since the emulator started or reset.
--dont\_report\_timeouts | This is a flag. If this flag is not set, a timeout is treated as a bug, which means that the input that timed out is stored to disk by the UI server. If this flag is set, timeouts are not treated as bugs.
--mutator\_so\_path | When you want to use another mutator. The default mutator is the mutator from libFuzzer.
