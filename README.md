# About

This repository contains the source code and a Visual Studio 2015 project
for building the Fortran77 to C source code translator **f2c**.

See also the **libf2c** project, for building a static library containing
the necessary runtime components for programs converted using f2c.

The f2c utility is still actively maintained and is available at
<http://www.netlib.org/f2c/>. Read more at <http://www.netlib.org/f2c/README>.

The contribution of this repository is the Visual Studio project, the source
code is the original f2c source code available at netlib.org.

# How to use

The repository includes the original source code of the f2c utility as provided
by netlib.org, with a small modification to avoid a compiler warning (see below).
So all you have to do is use Visual Studio 2015 to build the supplied project
**f2c.vcxproj**. This produces the the console application **f2c.exe**.

If you want to use your own copy of the f2c source code you just have to
get the project file from this repository, and then download the latest version
of from <http://www.netlib.org/f2c/>. Extract the subdirectory with name "src",
which contains all the relevant source files. Actually you only need the source
(.c) and header (.h) files from the src directory, so you can delete all others.

## Code modification in sysdep.c

When building the unchanged original source code you get two warnings about
*inconsistent dll linkage* (warning C4273) on the declarations of the functions
unlink and getpid in file sysdep.c. I am not entirely sure how much of a problem
this is, but it is easy to avoid them since you can just comment out the
declarations from sysdep.c, as they are already available from standard libraries
from existing includes. The same is true for declarations of fork and wait
in the same lines. So comment out the following two lines in **sysdep.c**:

```c
    Cextern int unlink Argdcl((const char *));
    Cextern int fork Argdcl((void)), getpid Argdcl((void)), wait Argdcl((int*));
```

# How the Visual Studio project was created

The following steps describe how the Visual Studio project was created, so by
following them you can do it all yourself without using any files from this repository.
The steps are based on the VC makefile (makefile.vc) included with the source code,
and then a substantial portion of my own trial and error.

1. Prepare the source code:
   1. Get the latest version from <http://www.netlib.org/f2c/>. Follow the
      instructions given to get access. Or for convenience I found it easier
      to download an archive file available from a mirror:
      <http://netlib.sandia.gov/cgi-bin/netlib/netlibfiles.tar?filename=netlib/f2c>.
   2. Extract the netlibfiles.tar (using 7-Zip for example) to a temporary directory.
   3. Copy all C source (*.c) and header (*.h) files from subdirectory "src" to
      a subdirectory "src" of the location of the project file you are creating.

2. Create a new project "f2c" of type console application, without precompiled header.

3. Add all source files, extension .c, to the project, except the following:
   * malloc.c
   * memset.c
   * sysdeptest.c
   * xsum.c

4. Add all header files, files with extension .h, to the project.

5. Modify the file sysdep.c as described above (comment out the two lines).

6. Modify the Character Set option, set it to "Use Multi-Byte Character Set" instead of Unicode.
   This because there is a call to the Windows API function GetVolumeInformation which is
   supplying ANSI strings (char*) as arguments, and therefore expects it to be the ANSI
   version GetVolumeInformationA and not the Unicode version GetVolumeInformationW which
   expects LPCWSTR (wchar_t) type of string arguments.

7. Add the following preprocessor directives on project level:

   ```c
   STRICT;WIN32_LEAN_AND_MEAN;NOMINMAX
   ```

   This is not an important step, it is just for a cleaner include of the Windows headers
   which are included only to be able to call the GetVolumeInformation function mentioned above.
   * STRICT is to enable strict type checking in Windows headers.
   * WIN32_LEAN_AND_MEAN is to speed the build process exclude rarely-used services from Windows headers.
   * NOMINMAX is to exclude min/max macros from Windows header, which may interfere with standard template versions.

8. Make sure the SDL Checks option is turned off. If you create a new console project using the
   wizard in VS2015 it will have these enabled in Debug configuration, but that results in the
   many uses of deprecated functions being treated as build errors.

9. Set warning level to 1 or 2. With level 3 you will get a lot of anoying warnings about unsafe sprintf,
   deprecated POSIX name for unlink etc. Of course you could improve the code instead, that would be even
   better, but for building the original source code (almost) untouched you can turn the warnings off.

# For the extra curious

## Checksum

The original makefile contains a separate make target for verifying the validity of the source files.
The target compiles the included source file `xsum.c` into a console application `xsum.exe`.
This is a very simple checksumming tool which can calculate CRC checksum of files. This tool is then
executed with a list of all source files as argument, and the result (a list of file names, CRC-
checksums and file sizes) are piped into a file `xsum1.out`. The original source code contains
a file `xsum0.out` containing the same information, but with the correct check sums as generated
when the source code was released. The final step of the make target is to run the Windows command
FC to compare the generated xsum1.out with the included xsum0.out. The result of the comparison
is shown directly to the user on the console, with an added message telling the user to check if
the files was reported as identical, and in that case telling the user to rename the file xsum1.out
to xsum.out (replace any existing). The final message says that "once you are happy that your source is OK"
you should trigger the build of the build target "f2c.exe" for building the actual application.

## About the use of Windows.h

As mentioned above, the Windows API header file `windows.h` is included to get access to the
function GetVolumeInformation. This only happens when preprocessor \_WIN32 is defined, which it
normally is for x86 builds. The function GetVolumeInformation is only used a single place in
the f2c source code: In the function set_tmp_names in the file sysdep.c. The set_tmp_names functions
sets up a set of pre-defined names of temporary files that will be used later. When the directive
\_WIN32 is defined the current process ID is used as a prefix of the name of each of the temporary
files. A call to the Windows function GetVolumeInformation is then added just to check that the
generated filename prefix is not resulting in a file name longer then what is supported by the
underlying system (it uses the output "maximum length of of a file name component that a
specified file system supports" of the call to GetVolumeInformation). If the temporary prefix
is too long, or if building without the \_WIN32 directive, then a simple fixed prefiks "f2c_" is used.

So for an even clear build you could remove the include of windows.h and the code block
calling the GetVolumeInformation function from systep.c. Then you also don't need the
preprocessor directives `STRICT;WIN32_LEAN_AND_MEAN;NOMINMAX` described above.


## About long long

The original make file for Visual C sets the preprocessor directive `NO_LONG_LONG`,
but since we have a built-in type `long long` in Visual C++ 2015 we don't need to set it
in our project.



# License

The libf2c source code is under copyright by AT&T, Lucent Technologies and Bellcore,
but it's license permits use, copy, modify, and distribute for any purpose and
without fee, provided that the copyright notice is included, and that the
permission notice and warranty disclaimer appear in supporting documentation.
See [LICENSE.md](LICENSE.md).

The Visual Studio project file and other supporting files, including this documention,
is also provided for free use.

# Authors

Albertony