# Attempting C++ Exception Support for the Target System

This document outlines the efforts made to port C++ exception handling to the `x86_64-elf` target system within this
development environment. The approach was based on the information provided in the OSDev wiki article on [C++ Exception Support](https://wiki.osdev.org/C%2B%2B_Exception_Support).

## Porting Newlib
"Newlib is a C standard library implementation intended for use on embedded systems." One of the main attractions of Newlib is that it's fairly easy to port to any new system, however, it is not a complete implementation and lacks networking and dynamic linking.

After researching and starting the porting process, we talked to Dr. Karim who informed us that we have an almost complete implementation of stdc, and that it makes no sense (in our case) to port an incomplete library. We eventually dropped the porting process in favor of moving faster towards C++ Exceptions.

## Initial Approach to Exceptions

The initial strategy involved the following steps based on the OSDev article [C++ Exception Support](https://wiki.osdev.org/C%2B%2B_Exception_Support) :

1.  **Porting `libcxxrt`:** This library provides the runtime support necessary for C++ features, including exceptions.
2.  **Porting `libgcc_eh`:** This library from GCC handles the low-level details of exception propagation and stack
    unwinding.

## Challenges Encountered

### Lack of a Fully Functioning `stdc`

A significant obstacle was the absence of a complete and functional standard C library (`stdc`) for the target
environment. The C++ standard library (`libcxx`) and its runtime dependencies often rely on features provided by `stdc`.

### Stubbing for `libcxxrt` Compilation

To overcome the missing `stdc` dependencies and allow `libcxxrt` to compile, it became necessary to add stubs for
certain C library functions. These stubs provided minimal or no actual functionality but were sufficient to satisfy the
compiler and linker during the build process of `libcxxrt`.

### Linker Script Modifications

The default linker scripts for the target environment were not configured to properly handle the object files and
libraries produced during the `libcxxrt` build. Modifications to the linker scripts were required to ensure that the
`libcxxrt` library could be linked successfully.

## Partial Success: `libcxxrt` Compilation

Despite the challenges, we were able to successfully compile the `libcxxrt` library for the `x86_64-elf` target after
adding the necessary stubs and adjusting the linker scripts. This indicated progress in establishing the foundational
runtime support for C++.

## Setback: Porting `libgcc_eh`

The second part of the endeavor focused on porting `libgcc_eh`, which is crucial for the actual mechanism of throwing
and catching exceptions. This proved to be significantly more complex than porting `libcxxrt`. Mainly due to missing system calls.

## Second approach (Itanium C++ ABI)
Other than the way presented in the OSDev article, the [Itanium C++ API](https://itanium-cxx-abi.github.io/cxx-abi/abi-eh.html) is a definition of C++ Exception handling API at three levels. It is a more "bear metal" approach to implementing exceptions, where we do not rely on any porting of libraries or external implementation. The second approach, although constitutes more work, seems to be more appropriate for our use case with NileOS.

## Conclusion

While the initial attempt to port C++ exceptions to the `x86_64-elf` target resulted in the successful compilation of
`libcxxrt` (with the aid of stubs and linker modifications), the subsequent effort to port `libgcc_eh` was unsuccessful.
The lack of a complete `stdc` and the inherent complexities of adapting low-level exception handling mechanisms like
those in `libgcc_eh` to a custom environment presented significant hurdles that could not be overcome at this time.

As such, it makes sense to continue with the second approach with implementing the low level details manually, instead of investing more time on trying to port the libraries.