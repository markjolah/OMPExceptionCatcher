# OMPExceptionCatcher
A lightweight class for managing C++ exception handling strategies in OpenMP code.

## Motivation
OpenMP code must catch any exceptions that may have been thrown before exiting the OpenMP block.  Allowing an unhanded exception to escape an OpenMP block normally leads to immediate program termination.  Properly functioning OpenMP code should never throw exceptions, however sometimes things go wrong.  As a mechanism for error reporting in exceptional circumstances, the C++ exception implementation poses no overhead to normally functioning code.

The goals of the OMPExceptionCatcher project are to:
 * Allow OpenMP and other parallel code to safely handle exceptions thrown in exceptional circumstances
 * Maintain a lock free, no-overhead execution path for normally functioning non-throwing code
 * Be minimally intrusive, flexible, and easy to incorporate into existing code
 * Lead to clear, readable, and maintainable OpenMP code

Functionally, the OMPExceptionCatcher acts as lightweight wrapper that allows an arbitrary function or lambda expression with arguments to be run while catching any unhandled exceptions using one of four possible thread-safe strategies as enumerated by `omp_exception_catcher::Strategy`.
 
## Exception Catching Strategies

 * `Strategy::DoNotTry` - Don't even try,  this is a null-op to completely disable any `OMPExceptionCatcher` effect.  Exceptions are not handled and will escape if thrown.
 * `Strategy::Continue` - Catch unhandled exceptions, but completely ignore them and keep going.
 * `Strategy::Abort`    - Catch unhandled exceptions, and immediately abort on the first exception.
 * `Strategy::RethrowFirst`  - Catch unhandled exceptions, keep only the first exception thrown.  Subsequent exceptions are ignored.  Stored exceptions may be re-thrown later in single-threaded code with `OMPExceptionCatcher::rethrow()`.

## Features
* **Low overhead** -  Locks are only acquired if an unhandled exception is actually caught.  Hopefully this never happens in normally functioning code.  Non-throwing code will see no
 performance penalty, and the compiler optimizer should inline the `OMPExceptionCatcher::run()` and any intervening lambda calls.
* **Easy to add** - OMPExceptionCatcher is header only.  It can be directly included in any OpenMP project.
* **Flexible usage** - The `run()` member function is called from parallel code blocks. It takes in any function or lambda and optionally a variadic list of arguments, and prevents any exceptions escaping that function call.
 
 ## Including OMPExceptionCatcher in a project
Since OMPExceptionCatcher is header-only, the easiest way to use it is via the [git subrepo](https://github.com/ingydotnet/git-subrepo) plugin.  Unlike the traditional `git submodule` command, `git subrepo` is transparent to other repository users.
1. Follow the [git subrepo install guide](https://github.com/ingydotnet/git-subrepo#installation-instructions) to install on a development machine.
2. Add OMPExceptionCatcher as a git subrepo:
```.sh
cd $MY_PROJ_REPOS
git subrepo clone https://github.com/markjolah/OMPExceptionCatcher include/MyProj/OMPExceptionCactcher
```
 
## Example usage:
Original code:
 ~~~.cxx
 #pragma omp parallel for
 for(int n=0; n < N; n++)
     my_output(n) = do_my_calculations(args(n));
 ~~~

Code modified to use OMPExceptionCatcher:
 ~~~.cxx
 #include "OMPExceptionCatcher/OMPExceptionCatcher.h"
 omp_exception_catcher::OMPExceptionCatcher catcher(); // Use default: Strategy::RethrowFirst
 #pragma omp parallel for
 for(int n=0; n < N; n++) 
     catcher.run([&]{ my_output(n) = do_my_calculations(args(n)); })
 catcher.rethrow(); //Required only for Strategy::RethrowFirst
 ~~~
 
 
 # License
 * Author: Mark J. Olah
 * Email: (mjo@cs.unm DOT edu)
 * Copyright: 2019
 * LICENSE: Apache 2.0.  See [LICENSE](https://github.com/markjolah/OMPExceptionCatcher/blob/master/LICENSE) file.
