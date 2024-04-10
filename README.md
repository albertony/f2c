# About

This repository contains the source code and a Visual Studio 2015 / Visual C++ 14.0
project (easily upgradable to newer versions) for building the Fortran77 to C source
code translator **f2c**.

See also my [libf2c](https://github.com/albertony/libf2c) project,
for building a static library containing the necessary runtime
components for programs converted using f2c.

The f2c utility is still actively maintained and is available at
<http://www.netlib.org/f2c/>. Changelog can be found at <http://www.netlib.org/f2c/changes>.
Read more in <http://www.netlib.org/f2c/README>.

The contribution of this repository is the Visual Studio project, the source
code is the original f2c source code available at netlib.org - with
some very small necessary fixes, described [below](#how-to-use).

## Changelog

See <http://www.netlib.org/f2c/changes> for history of changes in the netlib f2c source,
and run `f2c.exe --version` on an existing build to see which version it was built from.

10 Apr 2024
- Updated to latest version of netlib f2c source, version 20240312.
- This includes the fix in file proc.c that I previously did manually in my source.
- Released as version 1.2, with release builds of f2c.exe in 32-bit
and 64-bit using Visual Studio 2022 (17.9.6) / Visual C++ 14.3 and
Windows SDK 10.0.22621.0.

13 Jan 2022
- Updated to latest version of netlib f2c source, version 20200916.
   - The newer 20210928 entry in [changes](http://www.netlib.org/f2c/changes)
     is an update to readme only, i.e. does not change the version number
     of the f2c program.
- Fixes broken x64 build.
   - The netlib sources contains several relevant fixes in this area
     since my last version.
   - **Note:** There is one additional change to the source code that is needed to fix
     a memory access related crash in the x64 build of the program. Edit: This was later
     fixed in netlib sources version 20230428, see [changes](http://www.netlib.org/f2c/changes)
     for details.
- Released as version 1.1, with release builds of f2c.exe in 32-bit
and 64-bit using Visual Studio 2022 / Visual C++ 14.3.

22 Mar 2017
- Initial version, based on netlib f2c source version 20100827.
- Released as version 1.0, with release builds of f2c.exe in 32-bit
and 64-bit using Visual Studio 2015 Update 3 / Visual C++ 14.0.

# Building

This repository includes the original source code of the f2c utility as provided
by netlib.org, with some very small modifications described below, and an added
Visual Studio project. So all you have to do is use Visual Studio to build the supplied
project **f2c.vcxproj**. This produces the the console application **f2c.exe**.

The project file is for the older Visual Studio 2015, but you can open it with newest
Visual Studio 2022, and also upgrade it to use the newest versions of Platform Toolset
and Windows SDK.

If you want to use your own copy of the f2c source code you just have to
get the project file from this repository, and then download the latest version
of from <http://www.netlib.org/f2c/> (source archive direct download:
<http://www.netlib.org/f2c/src.tgz>). Extract the subdirectory named "src",
which contains all the relevant source files. Actually you only need the source
(.c) and header (.h) files from the src directory, so you can delete all others.

# How the Visual Studio project was created

The following steps describe how the Visual Studio project was created, just
for documentation but also such that you can followe similar steps to do it
all yourself without using any files from this repository.
The steps are based on the official VC makefile (makefile.vc) included with
the source code, and then a substantial portion of my own trial and error.

1. Prepare the source code:
   1. Get the latest version from <http://www.netlib.org/f2c/>.
      Archive direct download link: <http://www.netlib.org/f2c/src.tgz>.
   2. Extract the src.tgz (using 7-Zip for example) to a temporary directory.
   3. Copy all C source (*.c) and header (*.h) files from subdirectory "src" to
      a subdirectory "src" of the location of the project file you are creating.

2. Create a new project "f2c" of type console application, without precompiled header.

3. Add all source files, extension .c, to the project, except the following:
   * malloc.c
   * memset.c
   * sysdeptest.c
   * xsum.c

4. Add all header files, files with extension .h, to the project.

5. Modify the file sysdep.c as follows:

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

6. Create file tokdefs.h. This was included in previous versions, but with version
   20240312 it was not - however still needed to build. The version from previous versions
   can still be used, or it can be created from the file tokens included in the netlib
   sources.
   
   The original tokens file lists 100 keywords such as this:
   ```
   SEOS
   SCOMMENT
   SLABEL
   ...
   ```
   The needed tokdefs.h needs to define them in different format:
   ```c
   #define SEOS 1
   #define SCOMMENT 2
   #define SLABEL 3
   ...
   ```

7. Modify the Character Set option, set it to "Use Multi-Byte Character Set" instead of Unicode.
   This because there is a call to the Windows API function GetVolumeInformation which is
   supplying ANSI strings (char*) as arguments, and therefore expects it to be the ANSI
   version GetVolumeInformationA and not the Unicode version GetVolumeInformationW which
   expects LPCWSTR (wchar_t) type of string arguments.

8. Add the following preprocessor directives on project level:

   ```c
   STRICT;WIN32_LEAN_AND_MEAN;NOMINMAX
   ```

   This is not an important step, it is just for a cleaner include of the Windows headers
   which are included only to be able to call the GetVolumeInformation function mentioned above.
   * STRICT is to enable strict type checking in Windows headers.
   * WIN32_LEAN_AND_MEAN is to speed the build process exclude rarely-used services from Windows headers.
   * NOMINMAX is to exclude min/max macros from Windows header, which may interfere with standard template versions.

9. Make sure the SDL Checks option is turned off. If you create a new console project using the
   wizard in VS2015 it will have these enabled in Debug configuration, but that results in the
   many uses of deprecated functions being treated as build errors.

10. Set warning level to 1 or 2. With level 3 you will get a lot of anoying warnings about unsafe sprintf,
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
but since we have a built-in type `long long` in Visual C++ 2015 and newer we don't
need to set it in our project.

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
