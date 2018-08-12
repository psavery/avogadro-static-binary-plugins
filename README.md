# Creating Static Binary Plugins for Avogadro2

## Advantages over Linking
  - Avoids Licensing Issues
    - Only programs with non-copyleft open source licenses may be linked with Avogadro2
      - Non-copyleft examples (**can** link with Avogadro2): BSD, Apache, LGPL
      - Copyleft examples (**cannot** link with Avogadro2): GPL (any version)
  - Can Use Compilers Avogadro2 does not Currently Use
    - Fortran, for instance
    - Extra compilers may be added, but they can make the build process much more difficult
  - May Reduce Number of Segmentation Faults in Avogadro2
    - If the plugin is linked to and it segmentation faults, Avogadro2 will crash as well
    - If the plugin is not linked, Avogadro2 will only report an error
## Disadvantages over Linking
  - All desired functionality must be available in the binary via `stdin` and `stdout` or some other i/o
  - Extra steps must be taken to make sure the binary will work on all target operating systems
  - Will use more disk space than linked plugins
    - For instance, each plugin might have its own copy of the language's standard library

## Requirements
  - All desired functionality must be available in the binary via `stdin` and `stdout` or some other i/o
    - Open Babel is a great example
    - One can also write their own wrapper for a program that uses `stdin` and `stdout` and distribute that. [GenXrdPattern](https://github.com/psavery/genxrdpattern) is an example of this.
    
## Guide
### Windows
- Confirm the Binary is Static
  - Visual Studio's `dumpbin.exe` Program Works
    - Start up Visual Studio CMD environment and run `dumpbin /dependents` on the executable
    - Desired output (or something similar):
      - ```
        Image has the following dependencies:
          KERNEL32.dll
        ```
      - Note that the binary may depend on msvc runtime libraries, but it must be done with caution. It can limit compatibility among windows versions.
### Mac OS X
- Things to Consider
  - For `AppleClang`, Apple does not allow static linking to the standard libraries, but static linking to other libraries is allowed
  - Need to choose a specific Mac OS X version as the minimum version to support
- Compiler Flags
  - Choose a Mac OS X version as the Minimum Version
    - `-mmacosx-version-min=10.x`
      - Set this flag in the following environment variables: CFLAGS, CXXFLAGS, and LDFLAGS
    - CMake Flags
      - `-DCMAKE_OSX_DEPLOYMENT_TARGET:STRING=10.x`
        - Uses `-mmacosx-version-min=10x` underneath
  - `-static` for `AppleClang` is not supported and will fail to compile
    - `-static-libgcc` and `-static-libgfortran` are possibilities, though
- Confirm the Binary is Static
  - Apple's `otool` Program Works
    - Start up terminal and run `otool -L` on the executable
    - Desired output (or something similar):
      - ```
        /System/Library/Frameworks/Accelerate.framework/Versions/A/Accelerate (compatibility version 1.0.0, current version 4.0.0)
        /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1213.0.0)
        ```
      - Note that the binary may depend on c++ (and others such as gfortran) runtime libraries, but it must be done with caution. It can limit compatibility among OS X versions. Prefer compiling on an older operating system if this route is taken.
### Linux
- Things to Consider
  - Prefer Build on Older Operating System (Virtual Machine is Possible)
    - Binaries built with new CPU instruction sets will not run on older machines (illegal instruction error)
    - The program can dynamically link to an old system library, and the library should be available on newer OS's
    - CMake builds its static binaries on a Debian 6 virtual machine, for instance
- Compiler Flags
  - `-static`
- Confirm the Binary is Static
  - Linux's `ldd` Program Works
    - Start up terminal and run `ldd` on the executable
    - Desired output: `	not a dynamic executable``
    - Note that the binary may depend on c++ (and others such as gfortran) runtime libraries, but it must be done with caution. Prefer compiling on an older operating system if this route is taken.

## Examples
### [YAeHMOP](https://github.com/greglandrum/yaehmop)
- Reason for static binary: Difficult to build within Avogadro due to needed fortran compiler or `f2c` library.
  - It is also preferred, for speed, to build it linked with `blas` and `lapack`, which would add extra dependencies to Avogadro if an external binary were not used.
### [GenXRDPattern](https://github.com/psavery/genxrdpattern)
- Reason for static binary: Links to [ObjCryst++](https://github.com/vincefn/objcryst), which has a GPLv2 license.

## Alternatives
- Python/Java
  - Pros:
    - One plugin should work for all operating systems
  - Cons:
    - Requires user to have python/java installed, or packaging python/java with the program.
- Web API
  - Pros:
    - Works for all operating systems
    - No extra user installations required
  - Cons:
    - Requires users to have an internet connection
    - Requires an active server that may need to be maintained
- Docker
  - Pros:
    - One docker image works for all operating sytems
  - Cons:
    - Requires users to have docker installed on their computer
    - Currently requires admin privileges for clients 
    - May be slow on some operating systems (such as Mac OS X)
