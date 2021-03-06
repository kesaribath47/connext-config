# RTI Connext DDS Config

A small command-line utility, similar to pkg-config that help supply build information for a given architecture



## Usage

```
RTI Connext DDS Config version 1.0.0
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Usage:
    connext-config -h|--help    Show this help
    connext-config -V|--version Prints version number
    connext-config --list-all   List all platform architectures supported
    connext-config --dump-all   Dump all platforms and all settings (testing only)
    connext-config [modifiers] <what> <targetArch>

Where [modifiers] are:
    --static    use static linking against RTI Connext DDS
    --debug     use debug version of the RTI Connext DDS libraries
    --sh        use shell-like variable expansion (vs. make-like variables)
    --noexpand  do not expand environment variables in output

Required argument <what> is one of:
    --ccomp     output the C compiler to use
    --clink     output the C linker to use
    --cxxcomp   output the C++ compiler to use
    --cxxlink   output the C++ linker to use
    --cflags    output all pre-processor and compiler flags for C compiler
    --cxxflags  output all pre-processor and compiler flags for C++ compiler
    --ldflags   output the linker flags to use when building C programs
    --ldxxflags output the linker flags to use when building C++ programs
    --ldlibs    output the required libraries for C programs
    --ldxxlibs  output the required libraries for C++ programs
    --os        output the OS (i.e. UNIX, ANDROID, IOS, ...)
    --platform  output the Platform (i.e. i86, x64, armv7a, ...)

Required argument <targetArch> is one of the supported target architectures.
Use `--list-all` to output a list of supported architectures
```



The tool can be invoked in two ways:

1. To list all the supported targets (including targets NOT installed). This is equivalent of using:
   `pkg-config --list-all`. No other arguments are required.
2. To obtain the right tool name, flags and libraries for a given architecture. 

In general, when determining the tools required to build a RTI Connext DDS application, you may need to use this tool 5 times. For example, to build a C program:

* Use the `--ccom` to determine the correct C compiler to use
* Use the `--cflags` to determine the right flags for the C compiler 
* Use the `--clink` to determine the correct linker to use
* Use the `--ldflags` and `--libs` to get the flags and libraries for the linker. Place the output of the `--ldflags` at the beginning of the argument list, and the output of `--libs` at the end.



### Examples

Show all the supported Linux 64-bit targets: 

```
$ ./connext-config --list-all | grep x64Linux
x64Linux2.6gcc3.4.5
x64Linux2.6gcc4.1.1
x64Linux2.6gcc4.1.2
...
```



Show the C compiler to use for the target architecture `ppc4xxFPLinux2.6gcc4.5.1`:

```
$ ./connext-config --ccomp ppc4xxFPLinux2.6gcc4.5.1
powerpc-ngc-linux-gcc
```



Shows the libraries to use when statically linking (`--static`) C applications on `x64Linux2.6gcc4.5.1` with debugging information (`--debug`):

```
$ ./connext-config --debug --static --ldlibs x64Linux2.6gcc4.5.1
-L/Users/fabrizio/working/rti_connext_dds-6.0.0/lib/x64Linux2.6gcc4.5.1 -lnddsczd -lnddscorezd -ldl -lnsl -lm -lpthread -lrt
```



### How to use it in projects

Inside the `examples` directory you can find some projects that uses this tool to automatically configure the build system.

* `examples/autoconf-hello`: this is the same `hello_builtin` example that is part RTI workspace area (typically `~/rti_workspace`) under the `examples/connext_dds/c/hello_builtin` directory. This example uses the popular autotools (autoconf, automake) to invoke `connext-config` to determine the correct build settings. 
  Refer to the [README.md](examples/autoconf-hello/README.md) file for additional information on this example.





## Additional Notes

* The tool operates by looking up the requested information from the platform file used by `rtiddsgen`. This file is located under: `$NDDSHOME/resource/app/app_support/rtiddsgen/templates/projectfiles/platforms.vm`.

  where `$NDDSHOME` is the directory where RTI Connext DDS is installed.
  Because of this dependency, `connext-config` needs to be able to locate this directory.

* `connext-config` will first use the environment variable `$NDDSHOME` to locate the platform file above. If `$NDDSHOME` is not defined, it will attempt to determine the location of the platform file from the location of the `connext-config` application (assumes this tool is invoked from the `$NDDSHOME/bin` directory).
  If it cannot locate the `platforms.vm` file, it will report an error.

* All the error messages are printed to `stderr`, while the normal results are printed to `stdout` (help is printed to `stdout`).

* The `--debug` option affects only the name of the required libraries and NOT the C or C++ compiler flags (for example, it does not include `-g` or disable any optimization, nor define any debug-related macros). 

* Many platforms are not supported: Windows, iOS, Integrity, and in general all the platform where a project is normally not built through Makefile. All the *nix-based architectures are supported.

* To build `connext-config` from the source, just run:
  
  ```
  $ autoreconf -ivf
  $ ./configure
  $ make
  ```
  
  You need to have the traditional autoconf, automake, make, buildtools installed in your system.
  
* To install this application on your existing RTI Connext DDS installation:
  
  * Copy the script: `bin/connext-config` under `$NDDSHOME/bin`
  * Copy the compiled version of connext-config under: `$NDDSHOME/resource/app/bin/<hostPlatform>` (where <hostPlatform> is the name of the directory containing platform-specific binaries).
  
  Alternatively just un-tar one of the pre-built binaries (see the [Release page](https://github.com/rticommunity/connext-config/releases))
