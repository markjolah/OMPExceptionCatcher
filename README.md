# OMPExceptionCatcher
A lightweight class for managing C++ exception handling strategies in OpenMP code.

## Motivation
OpenMP code must catch any exceptions that may have been thrown before exiting the OpenMP block.  Allowing an unhanded exception to escape an OpenMP block, normally leads to immediate program termination.  Properly functioning OpenMP code should never throw exceptions, however sometimes things go wrong.  As a mechanism for error reporting in exceptional circumstances, the C++ exception implementation poses no overhead to normally functioning code.

The goal of the OMPExceptionCatcher class is to
 * allow easy handling of exceptions throw in exceptional circumstances even if in OpenMP code blocks
 * maintain the un-measurably small overhead of the normal C++ exception handling mechanism even in parallel code.

The OMPExceptionCatcher acts as lightweight wrapper that allows an arbitrary function or lambda expression to be run safely and efficiently in OMP even if it might throw exceptions.  One of four possible strategies my be employed as determined By the `omp_exception_catcher::Strategies` enum.
 
 ## Exception Catching Strategy's
 * `Strategies::DoNotTry` - Don't even try,  this is a null-op to completely disable this class's effect.  Exceptions are not handled and will escape if thrown.
 * `Strategies::Continue` - Catch un-handled exceptions, but completely ignore them and keep going.
 * `Strategies::Abort`    - Catch un-handled exceptions, and immediately abort on the first exception.
 * `Strategies::RethrowFirst`  - Catch un-handled exceptions, keep only the first exception thrown.  Subsequent exceptions are ignored.  Stored exceptions may be re-thrown later in single-threaded code with `OMPExceptionCatcher::rethrow()`.

## Features
 * **Low overhead** -  Locks are only acquired if an un-handled exception is actually caught.  Hopefully this never happens in normally functioning code.  Non-throwing code will see no
 performance penalty, and the compiler optimizer should inline the `OMPExceptionCatcher::run()` call.
 * **Easy to add** - OMPExceptionCatcher is header only.  It can be directly included in any OpenMP project.
 * **Flexible usage** - The `run()` member function is called from parallel code blocks. It takes in any function or lambda and optionally a variadic list of arguments, and prevents any exceptions escaping that function call.
 
 ## Including OMPExceptionCatcher in a project
Since OMPExceptionCatcher is header-only, the easiest way to use it is via the [git subrepo](https://github.com/ingydotnet/git-subrepo) plugin.  Unlike the traditional `git submodule` command, `git subrepo` is transparent to other users of your repository, and solves several issues prevalent with `git submodule` useage.
 * Follow the [git subrepo install guide](https://github.com/ingydotnet/git-subrepo#installation-instructions) to install on a development machine.

```.sh
cd $MY_PROJ_REPOS
git subrepo pull https://github.com/markjolah/OMPExceptionCatcher include/MyProj/OMPExceptionCactcher
```
 
 ## Example usage:
 ~~~.cxx
 #include "MyProj/OMPExceptionCatcher/OMPExceptionCatcher.h"
 omp_exception_catcher::OMPExceptionCatcher catcher();
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
