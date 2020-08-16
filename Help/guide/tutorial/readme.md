CMake 教程
**************

   .. 内容::

教程提供了一个逐步指导，涵盖了常见的CMake构建系统需要的帮助。各种各样的话题在一个示例项目中一起工作是非常有帮助的。教程示例的文档和源代码可以在``Help/guide/tutorial`` CMake源代码树的目录。每一步都有它自己的子目录包含可以用作起点的代码。这个教程示例是渐进式的，因此每个步骤都提供完整的上一步的解决方案

基础起点 (Step 1)
===============================

本的项目是从源代码文件构建的可执行文件。对于简单的项目，只需一个三行代码的``CMakeLists.txt文件``文件。这将是我们教程的起点。创建
`CMakeLists.txt文件``“Step1”目录中的文件，如下所示：

```

  cmake_minimum_required(VERSION 3.10)

  # set the project name
  project(Tutorial)

  # add the executable
  add_executable(Tutorial tutorial.cxx)
```

请注意，此示例在``CMakeLists.txt文件``文件。CMake支持大写、小写和混合大小写命令。代码``tutorial.cxx``在“Step1”目录中提供，并且可以用来计算一个数的平方根

添加版本号和一个配置文件
--------------------------------------------------

我们将添加的第一个特性是为可执行文件和项目提供版本号。虽然我们可以只在源代码中实现，但用``CMakeLists.txt文件``可提供更大的灵活性。

首先，修改``CMakeLists.txt文件``使用`project`命令设置项目名称和版本号。

```
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)
```
其次，配置一个头文件把版本传入源文件。
```
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

为了使配置文件写入二进制树，我们必须添加其路径到搜索路径列表中，以便能探索到该头文件。添加以下行到``CMakeLists.txt``。

```
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

用你的编辑器编辑 ``TutorialConfig.h.in`` 文件，包含以下内容：
```
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

当Cmake配置这个头文件时，参数``@Tutorial_VERSION_MAJOR@`` 和 ``@Tutorial_VERSION_MINOR@`` 将被加入.

接着，修改 ``tutorial.cxx`` 以包含此头文件,``TutorialConfig.h``.

最后，我们可以在 ``tutorial.cxx`` 中打印出版本号：
follows:

```
  if (argc < 2) {
    // report version
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
```

指定 C++ 标准
-------------------------

接下来让我们增加 支持C++11 标准到我们的工程， 通过修改 ``atof`` with``std::stod`` in ``tutorial.cxx``. 同时，移除``#include <cstdlib>``.

```
const double inputValue = std::stod(argv[1]);
```

我们需要在CMake中显式地声明它应该使用正确的标志。最简单的设置支持某个C++标准是修改``CMakeLists.txt文件``，配置`CMAKE_CXX_STANDARD`及`CMAKE_CXX_STANDARD_REQUIRED`的值。

```
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

构建和测试
--------------

命令行运行`cmake`  或运行图形化的cmake程序`cmake-gui`用你选择的构建工具配置项目并构建。

例如，我们从命令行进入``Help/guide/tutorial`` 项目， 并运行以下命令：

```
mkdir Step1_build
cd Step1_build
cmake ../Step1
cmake --build .
```

导航到构建Tutorial的目录（可能是make目录或Debug或Release build configuration子目录）并运行以下命令：


``` 
 Tutorial 4294967296
  Tutorial 10
  Tutorial
```

添加一个库 (Step 2)
=========================

现在我们将在我们的项目中添加一个库。这个库包含我们自己的计算一个数的平方根的实现。可执行文件可以可以选择使用此库，而不是编译器的。

此教程中，我们把库放入名为 ``MathFunctions``的子目录中. 其中包含一个头文件``MathFunctions.h``, 和一个源文件 ``mysqrt.cxx``. 源文件有一函数 ``mysqrt`` 提供类似编译器的函数 ``sqrt`` 的功能.

``MathFunctions``目录中添加 ``CMakeLists.txt`` 文件，其包含一行代码：
```
add_library(MathFunctions mysqrt.cxx)
```

我们在顶层 ``CMakeLists.txt`` 中添加一条命令`add_subdirectory`以确保库会加入我们的运行程序，同时包含目录添加 ``MathFunctions`` 路径以确保 ``mqsqrt.h``头文件能被找到. 最后，顶层的 ``CMakeLists.txt`` 如下所示:

```
# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC MathFunctions)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                          "${PROJECT_BINARY_DIR}"
                          "${PROJECT_SOURCE_DIR}/MathFunctions"
                          )
```

现在我们制作一个使用``MathFunctions``库的选项，在教程中添加系统自检确实没有必要这样做，对于更大的项目来说，这是很常见的事情。
第一步是向顶层添加一个选项 ``CMakeLists.txt文件``文件。
```
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

这个选项会显示在`cmake-gui ` 和`ccmake`中，缺省值为ON，且可以被用户修改。设置会存储在cache中，用户不必每次都去设置它。
另一个变动是创建build和linking``MathFunctions``库的条件。我们修改顶层``CMakeLists.txt``：
```
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
  list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()
```
# 添加执行程序
```
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           ${EXTRA_INCLUDES}
                           )
```

注意我们使用变量 ``EXTRA_LIBS`` 收集所有选项库，最后把它链接到可执行文件，变量
``EXTRA_INCLUDES`` 用于采集选项库的头文件. 这是一个当处理许多可选组件时，我们将介绍
下一步的现代方法。

源码的相应改变很直接. 
首先,在 ``tutorial.cxx``, 包含 ``MathFunctions.h`` :

```
#ifdef USE_MYMATH
#  include "MathFunctions.h"
#endif
```

接着, 同一文件中, make ``USE_MYMATH`` 由选项控件选择哪个函数:

```
#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif
```

源码现在需要 ``USE_MYMATH`` 我们把它加到
``TutorialConfig.h.in`` :

```
#cmakedefine USE_MYMATH
```

**运用**: 为什么在使用选项``USE_MYMATH``后配置 ``TutorialConfig.h.in``很重要?

运行``cmake`` 或``cmake-gui`` 配置并build项目. 并运行例程程序.

使用 ``ccmake`` 或 ``cmake-gui`` 更新变量``USE_MYMATH``. 重新build和运行示例程序. 哪个函数会给出更好的结果?, sqrt 还是 mysqrt?

添加库的使用需求 (Step 3)
==============================================

使用需求允许更好地控制库或可执行文件链接和包含（有些命令是库自需求，有些是调用者需求，我们需要区分它们），同时给予CMake内的目标传递更多控制信息。以下是使用率较高的几个命令：

  - `target_compile_definitions()`
  - `target_compile_options()`
  - `target_include_directories()`
  - `target_link_libraries()`

我们利用Cmake的新方法重新实现第二步 `添加一个库 (Step 2)`. 首先，任何要使用
``MathFunctions``库 必须包含其源码路径, 但
`MathFunctions`库本身不需要这个路径。 itself doesn't. 因此这将成为一个 ``INTERFACE`` 使用需求.

记住``INTERFACE``是指消费者需要的东西，但生产者不需要.在``MathFunctions/CMakeLists.txt``末尾添加:

```
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
```
上面代码，库本身并不需要，调用者才需要，所以用``INTERFACE``
注：cmake target_link_libraries() 等命令中<PUBLIC|PRIVATE|INTERFACE> 的区别
如果你有一个需要重用的目标（通常是库），这些模式是非常有用的。 PRIVATE 定义仅适用于库目标，但不适用于使用此库的其他目标。 INTERFACE 定义仅适用于依赖目标，但不适用于库本身。 PUBLIC 定义适用于库目标以及依赖目标。
  
现在我们指定了对`MathFunctions`的使用需求，现在我们可以从顶层``CMakeLists.txt``安全移除 ``EXTRA_INCLUDES`` 变量，顶层``CMakeLists.txt``做了以下两个改动：

```
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()
```

还有这:

```
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )
```

这些做完后, 运行`cmake` 或`cmake-gui` 用你选择的build工具 ``cmake --build .``配置和build项目。

安装和测试 (Step 4)
===============================

现在为我们的项目添加安装规则和测试支持.

安装规则
-------------

安装规则相当简单：对于MathFunctions，我们希望安装
库和头文件，对于应用程序需要安装可执行文件和配置头文件。

在 ``MathFunctions/CMakeLists.txt`` 我们增加:

```
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

在顶层 ``CMakeLists.txt`` 末尾增加:

```
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
```

这就是我们为示例创建的一个基本的本地install所做的所有工作。

运行`cmake` 或`cmake-gui` 用你选择的配置工具，配置和build项目. 使用``install``选项执行`cmake ` 命令 (在 3.15,或更老的CMake版本中，须在命令行使用 ``make install``) 或在IDE中执行
``INSTALL``. 这将安装需要的头文件，库，可执行程序。

CMake 变量`CMAKE_INSTALL_PREFIX` 用于指明文件要安装的的根目录，如果使用 ``cmake --install`` ，使用参数 ``--prefix`` 可指定安装路径. 对于
multi-configuration 工具, 使用 ``--config`` 参数实现配置.

Verify that the installed Tutorial runs.

测试支持
---------------

接下来，我们测试我们的应用. 在顶层 ``CMakeLists.txt``的末尾，
我们使能测试并加入一系列基本测试，检查应用是否正常工作。

```
enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# define a function to simplify adding tests
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction(do_test)

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

第一个测试只是验证应用程序是否运行，而不是segfault或
崩溃，返回值为零。这是的基本的
CTest测试。

The next test makes use of the :prop_test:`PASS_REGULAR_EXPRESSION` test
property to verify that the output of the test contains certain strings. In
this case, verifying that the usage message is printed when an incorrect number
of arguments are provided.
下一个测试使用`PASS_REGULAR_EXPRESSION`测试
属性验证测试的输出是否包含某些字符串。在
在这种情况下，使用消息打印验证是否在错误的参数

Lastly, we have a function called ``do_test`` that runs the application and
verifies that the computed square root is correct for given input. For each
invocation of ``do_test``, another test is added to the project with a name,
input, and expected results based on the passed arguments.

Rebuild the application and then cd to the binary directory and run the
:manual:`ctest <ctest(1)>` executable: ``ctest -N`` and ``ctest -VV``. For
multi-config generators (e.g. Visual Studio), the configuration type must be
specified. To run tests in Debug mode, for example, use ``ctest -C Debug -VV``
from the build directory (not the Debug subdirectory!). Alternatively, build
the ``RUN_TESTS`` target from the IDE.
最后，我们有一个名为“do\u test”的函数，它运行应用程序并验证计算的平方根对于给定输入是否正确。为每一个调用“do_test”时，会将另一个名为的测试添加到项目中，输入，以及基于传递参数的预期结果。重新生成应用程序，然后cd到二进制目录并运行`ctest`可执行文件：``ctest-N``和`ctest-VV`。为多配置生成器（例如Visual Studio），配置类型必须为明确规定。例如，要在调试模式下运行测试，请使用“ctest-cdebug-VV”``从生成目录（而不是调试子目录！）。或者，构建IDE中的“运行测试”目标。

添加系统自检 (Step 5)
====================================

Let us consider adding some code to our project that depends on features the
target platform may not have. For this example, we will add some code that
depends on whether or not the target platform has the ``log`` and ``exp``
functions. Of course almost every platform has these functions but for this
tutorial assume that they are not common.
让我们考虑在项目中添加一些依赖于目标平台可能没有的特性。对于这个例子，我们将添加一些代码取决于目标平台是否具有“log”和“exp”``功能。当然，当然，几乎每个平台都有这些功能，但我们的教程假设它们并不常见。

If the platform has ``log`` and ``exp`` then we will use them to compute the
square root in the ``mysqrt`` function. We first test for the availability of
these functions using the :module:`CheckSymbolExists` module in the top-level
``CMakeLists.txt``. On some platforms, we will need to link to the m library.
If ``log`` and ``exp`` are not initially found, require the m library and try
again.

We're going to use the new defines in ``TutorialConfig.h.in``, so be sure to
set them before that file is configured.

.. literalinclude:: Step6/MathFunctions/CMakeLists.txt
  :language: cmake
  :start-after: # does this system provide the log and exp functions?
  :end-before: # add compile definitions

Now let's add these defines to ``TutorialConfig.h.in`` so that we can use them
from ``mysqrt.cxx``:

.. code-block:: console

  // does the platform provide exp and log functions?
  #cmakedefine HAVE_LOG
  #cmakedefine HAVE_EXP

If ``log`` and ``exp`` are available on the system, then we will use them to
compute the square root in the ``mysqrt`` function. Add the following code to
the ``mysqrt`` function in ``MathFunctions/mysqrt.cxx`` (don't forget the
``#endif`` before returning the result!):

.. literalinclude:: Step6/MathFunctions/mysqrt.cxx
  :language: c++
  :start-after: // if we have both log and exp then use them
  :end-before: // do ten iterations

We will also need to modify ``mysqrt.cxx`` to include ``cmath``.

.. literalinclude:: Step6/MathFunctions/mysqrt.cxx
  :language: c++
  :end-before: #include <iostream>

Run the :manual:`cmake  <cmake(1)>` executable or the
:manual:`cmake-gui <cmake-gui(1)>` to configure the project and then build it
with your chosen build tool and run the Tutorial executable.

You will notice that we're not using ``log`` and ``exp``, even if we think they
should be available. We should realize quickly that we have forgotten to
include ``TutorialConfig.h`` in ``mysqrt.cxx``.

We will also need to update ``MathFunctions/CMakeLists.txt`` so ``mysqrt.cxx``
knows where this file is located:

.. code-block:: cmake

  target_include_directories(MathFunctions
            INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
            PRIVATE ${CMAKE_BINARY_DIR}
            )

After making this update, go ahead and build the project again and run the
built Tutorial executable. If ``log`` and ``exp`` are still not being used,
open the generated ``TutorialConfig.h`` file from the build directory. Maybe
they aren't available on the current system?

Which function gives better results now, sqrt or mysqrt?

特殊的编译Definition
--------------------------

Is there a better place for us to save the ``HAVE_LOG`` and ``HAVE_EXP`` values
other than in ``TutorialConfig.h``? Let's try to use
:command:`target_compile_definitions`.

First, remove the defines from ``TutorialConfig.h.in``. We no longer need to
include ``TutorialConfig.h`` from ``mysqrt.cxx`` or the extra include in
``MathFunctions/CMakeLists.txt``.

Next, we can move the check for ``HAVE_LOG`` and ``HAVE_EXP`` to
``MathFunctions/CMakeLists.txt`` and then specify those values as ``PRIVATE``
compile definitions.

.. literalinclude:: Step6/MathFunctions/CMakeLists.txt
  :language: cmake
  :start-after: # does this system provide the log and exp functions?
  :end-before: # install rules

After making these updates, go ahead and build the project again. Run the
built Tutorial executable and verify that the results are same as earlier in
this step.

Adding a Custom Command and Generated File (Step 6)
===================================================

Suppose, for the purpose of this tutorial, we decide that we never want to use
the platform ``log`` and ``exp`` functions and instead would like to
generate a table of precomputed values to use in the ``mysqrt`` function.
In this section, we will create the table as part of the build process,
and then compile that table into our application.

First, let's remove the check for the ``log`` and ``exp`` functions in
``MathFunctions/CMakeLists.txt``. Then remove the check for ``HAVE_LOG`` and
``HAVE_EXP`` from ``mysqrt.cxx``. At the same time, we can remove
:code:`#include <cmath>`.

In the ``MathFunctions`` subdirectory, a new source file named
``MakeTable.cxx`` has been provided to generate the table.

After reviewing the file, we can see that the table is produced as valid C++
code and that the output filename is passed in as an argument.

The next step is to add the appropriate commands to the
``MathFunctions/CMakeLists.txt`` file to build the MakeTable executable and
then run it as part of the build process. A few commands are needed to
accomplish this.

First, at the top of ``MathFunctions/CMakeLists.txt``, the executable for
``MakeTable`` is added as any other executable would be added.

.. literalinclude:: Step7/MathFunctions/CMakeLists.txt
  :language: cmake
  :start-after: # first we add the executable that generates the table
  :end-before: # add the command to generate the source code

Then we add a custom command that specifies how to produce ``Table.h``
by running MakeTable.

.. literalinclude:: Step7/MathFunctions/CMakeLists.txt
  :language: cmake
  :start-after: # add the command to generate the source code
  :end-before: # add the main library

Next we have to let CMake know that ``mysqrt.cxx`` depends on the generated
file ``Table.h``. This is done by adding the generated ``Table.h`` to the list
of sources for the library MathFunctions.

.. literalinclude:: Step7/MathFunctions/CMakeLists.txt
  :language: cmake
  :start-after: # add the main library
  :end-before: # state that anybody linking

We also have to add the current binary directory to the list of include
directories so that ``Table.h`` can be found and included by ``mysqrt.cxx``.

.. literalinclude:: Step7/MathFunctions/CMakeLists.txt
  :start-after: # state that we depend on our bin
  :end-before: # install rules

Now let's use the generated table. First, modify ``mysqrt.cxx`` to include
``Table.h``. Next, we can rewrite the mysqrt function to use the table:

.. literalinclude:: Step7/MathFunctions/mysqrt.cxx
  :language: c++
  :start-after: // a hack square root calculation using simple operations

Run the :manual:`cmake  <cmake(1)>` executable or the
:manual:`cmake-gui <cmake-gui(1)>` to configure the project and then build it
with your chosen build tool.

When this project is built it will first build the ``MakeTable`` executable.
It will then run ``MakeTable`` to produce ``Table.h``. Finally, it will
compile ``mysqrt.cxx`` which includes ``Table.h`` to produce the MathFunctions
library.

Run the Tutorial executable and verify that it is using the table.

创建一个安装器 (Step 7)
==============================

Next suppose that we want to distribute our project to other people so that
they can use it. We want to provide both binary and source distributions on a
variety of platforms. This is a little different from the install we did
previously in `Installing and Testing (Step 4)`_ , where we were
installing the binaries that we had built from the source code. In this
example we will be building installation packages that support binary
installations and package management features. To accomplish this we will use
CPack to create platform specific installers. Specifically we need to add a
few lines to the bottom of our top-level ``CMakeLists.txt`` file.

.. literalinclude:: Step8/CMakeLists.txt
  :language: cmake
  :start-after: # setup installer

That is all there is to it. We start by including
:module:`InstallRequiredSystemLibraries`. This module will include any runtime
libraries that are needed by the project for the current platform. Next we set
some CPack variables to where we have stored the license and version
information for this project. The version information was set earlier in this
tutorial and the ``license.txt`` has been included in the top-level source
directory for this step.

Finally we include the :module:`CPack module <CPack>` which will use these
variables and some other properties of the current system to setup an
installer.

The next step is to build the project in the usual manner and then run the
:manual:`cpack <cpack(1)>` executable. To build a binary distribution, from the
binary directory run:

.. code-block:: console

  cpack

To specify the generator, use the ``-G`` option. For multi-config builds, use
``-C`` to specify the configuration. For example:

.. code-block:: console

  cpack -G ZIP -C Debug

To create a source distribution you would type:

.. code-block:: console

  cpack --config CPackSourceConfig.cmake

Alternatively, run ``make package`` or right click the ``Package`` target and
``Build Project`` from an IDE.

Run the installer found in the binary directory. Then run the installed
executable and verify that it works.

Adding Support for a Dashboard (Step 8)
=======================================

Adding support for submitting our test results to a dashboard is simple. We
already defined a number of tests for our project in `Testing Support`_. Now we
just have to run those tests and submit them to a dashboard. To include support
for dashboards we include the :module:`CTest` module in our top-level
``CMakeLists.txt``.

Replace:

.. code-block:: cmake

  # enable testing
  enable_testing()

With:

.. code-block:: cmake

  # enable dashboard scripting
  include(CTest)

The :module:`CTest` module will automatically call ``enable_testing()``, so we
can remove it from our CMake files.

We will also need to create a ``CTestConfig.cmake`` file in the top-level
directory where we can specify the name of the project and where to submit the
dashboard.

.. literalinclude:: Step9/CTestConfig.cmake
  :language: cmake

The :manual:`ctest <ctest(1)>` executable will read in this file when it runs.
To create a simple dashboard you can run the :manual:`cmake <cmake(1)>`
executable or the :manual:`cmake-gui <cmake-gui(1)>` to configure the project,
but do not build it yet. Instead, change directory to the binary tree, and then
run:

  ctest [-VV] -D Experimental

Remember, for multi-config generators (e.g. Visual Studio), the configuration
type must be specified::

  ctest [-VV] -C Debug -D Experimental

Or, from an IDE, build the ``Experimental`` target.

The :manual:`ctest <ctest(1)>` executable will build and test the project and
submit the results to Kitware's public dashboard:
https://my.cdash.org/index.php?project=CMakeTutorial.

混合静态库和共享库 (Step 9)
=================================

这一节，我们展示变量`BUILD_SHARED_LIBS` 可以用于控制命令`add_library`的一些缺省行为,
允许build时控制库没有明确指明(``STATIC``,``SHARED``, ``MODULE`` or ``OBJECT``) 类型.
这就需要加入变量`BUILD_SHARED_LIBS` 到顶层 ``CMakeLists.txt``. 我们使用`option` 命令 as it allows
users to optionally select if the value should be ON or OFF.

下一步，我们重构 MathFunctions 成为一个实时库，其封装了使用 ``mysqrt`` 还是 ``sqrt``, 以替代之前的实现方式. 这意味着t ``USE_MYMATH`` 不再用于控制` MathFunctions`, 取而代之是控制这个库的行为.

第一步，更新顶层``CMakeLists.txt`` :

```
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# control where the static and shared libraries are built so that on windows
# we don't need to tinker with the path to run the executable
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)

# configure a header file to pass the version number only
configure_file(TutorialConfig.h.in TutorialConfig.h)

# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions)
```

这样我们使库`MathFunctions` 处于一直使用状态, 我们需要更新库的实现逻辑. 因此，在库 ``MathFunctions/CMakeLists.txt`` we need to
create a SqrtLibrary that will conditionally be built when ``USE_MYMATH`` is
enabled. Now, since this is a tutorial, we are going to explicitly require
that SqrtLibrary is built statically.

最后 ``MathFunctions/CMakeLists.txt`` 如下:

```
# add the library that runs
add_library(MathFunctions MathFunctions.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
                           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                           )

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if(USE_MYMATH)

  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")

  # first we add the executable that generates the table
  add_executable(MakeTable MakeTable.cxx)

  # add the command to generate the source code
  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    DEPENDS MakeTable
    )

  # library that just does sqrt
  add_library(SqrtLibrary STATIC
              mysqrt.cxx
              ${CMAKE_CURRENT_BINARY_DIR}/Table.h
              )

  # state that we depend on our binary dir to find Table.h
  target_include_directories(SqrtLibrary PRIVATE
                             ${CMAKE_CURRENT_BINARY_DIR}
                             )

  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()

# define the symbol stating we are using the declspec(dllexport) when
# building on windows
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")

# install rules
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

Next, update ``MathFunctions/mysqrt.cxx`` to use the ``mathfunctions`` and
``detail`` namespaces:

.. literalinclude:: Step10/MathFunctions/mysqrt.cxx
  :language: c++

We also need to make some changes in ``tutorial.cxx``, so that it no longer
uses ``USE_MYMATH``:

#. Always include ``MathFunctions.h``
#. Always use ``mathfunctions::sqrt``
#. Don't include cmath

Finally, update ``MathFunctions/MathFunctions.h`` to use dll export defines:

.. literalinclude:: Step10/MathFunctions/MathFunctions.h
  :language: c++

At this point, if you build everything, you will notice that linking fails
as we are combining a static library without position independent code with a
library that has position independent code. The solution to this is to
explicitly set the :prop_tgt:`POSITION_INDEPENDENT_CODE` target property of
SqrtLibrary to be True no matter the build type.

.. literalinclude:: Step10/MathFunctions/CMakeLists.txt
  :language: cmake
  :lines: 37-42

**Exercise**: We modified ``MathFunctions.h`` to use dll export defines.
Using CMake documentation can you find a helper module to simplify this?


Adding Generator Expressions (Step 10)
======================================

:manual:`Generator expressions <cmake-generator-expressions(7)>` are evaluated
during build system generation to produce information specific to each build
configuration.

:manual:`Generator expressions <cmake-generator-expressions(7)>` are allowed in
the context of many target properties, such as :prop_tgt:`LINK_LIBRARIES`,
:prop_tgt:`INCLUDE_DIRECTORIES`, :prop_tgt:`COMPILE_DEFINITIONS` and others.
They may also be used when using commands to populate those properties, such as
:command:`target_link_libraries`, :command:`target_include_directories`,
:command:`target_compile_definitions` and others.

:manual:`Generator expressions <cmake-generator-expressions(7)>`  may be used
to enable conditional linking, conditional definitions used when compiling,
conditional include directories and more. The conditions may be based on the
build configuration, target properties, platform information or any other
queryable information.

There are different types of
:manual:`generator expressions <cmake-generator-expressions(7)>` including
Logical, Informational, and Output expressions.

Logical expressions are used to create conditional output. The basic
expressions are the 0 and 1 expressions. A ``$<0:...>`` results in the empty
string, and ``<1:...>`` results in the content of "...".  They can also be
nested.

A common usage of
:manual:`generator expressions <cmake-generator-expressions(7)>` is to
conditionally add compiler flags, such as those for language levels or
warnings. A nice pattern is to associate this information to an ``INTERFACE``
target allowing this information to propagate. Lets start by constructing an
``INTERFACE`` target and specifying the required C++ standard level of ``11``
instead of using :variable:`CMAKE_CXX_STANDARD`.

So the following code:

.. literalinclude:: Step10/CMakeLists.txt
  :language: cmake
  :start-after: project(Tutorial VERSION 1.0)
  :end-before: # control where the static and shared libraries are built so that on windows

Would be replaced with:

.. literalinclude:: Step11/CMakeLists.txt
  :language: cmake
  :start-after: project(Tutorial VERSION 1.0)
  :end-before: # add compiler warning flags just when building this project via


Next we add the desired compiler warning flags that we want for our project. As
warning flags vary based on the compiler we use the ``COMPILE_LANG_AND_ID``
generator expression to control which flags to apply given a language and a set
of compiler ids as seen below:

.. literalinclude:: Step11/CMakeLists.txt
  :language: cmake
  :start-after: # the BUILD_INTERFACE genex
  :end-before: # control where the static and shared libraries are built so that on windows

Looking at this we see that the warning flags are encapsulated inside a
``BUILD_INTERFACE`` condition. This is done so that consumers of our installed
project will not inherit our warning flags.


**Exercise**: Modify ``MathFunctions/CMakeLists.txt`` so that all targets have
a :command:`target_link_libraries` call to ``tutorial_compiler_flags``.


Adding Export Configuration (Step 11)
=====================================

During `Installing and Testing (Step 4)`_ of the tutorial we added the ability
for CMake to install the library and headers of the project. During
`Building an Installer (Step 7)`_ we added the ability to package up this
information so it could be distributed to other people.

The next step is to add the necessary information so that other CMake projects
can use our project, be it from a build directory, a local install or when
packaged.

The first step is to update our :command:`install(TARGETS)` commands to not
only specify a ``DESTINATION`` but also an ``EXPORT``. The ``EXPORT`` keyword
generates and installs a CMake file containing code to import all targets
listed in the install command from the installation tree. So let's go ahead and
explicitly ``EXPORT`` the MathFunctions library by updating the ``install``
command in ``MathFunctions/CMakeLists.txt`` to look like:

.. literalinclude:: Complete/MathFunctions/CMakeLists.txt
  :language: cmake
  :start-after: # install rules

Now that we have MathFunctions being exported, we also need to explicitly
install the generated ``MathFunctionsTargets.cmake`` file. This is done by
adding the following to the bottom of the top-level ``CMakeLists.txt``:

.. literalinclude:: Complete/CMakeLists.txt
  :language: cmake
  :start-after: # install the configuration targets
  :end-before: include(CMakePackageConfigHelpers)

At this point you should try and run CMake. If everything is setup properly
you will see that CMake will generate an error that looks like:

.. code-block:: console

  Target "MathFunctions" INTERFACE_INCLUDE_DIRECTORIES property contains
  path:

    "/Users/robert/Documents/CMakeClass/Tutorial/Step11/MathFunctions"

  which is prefixed in the source directory.

What CMake is trying to say is that during generating the export information
it will export a path that is intrinsically tied to the current machine and
will not be valid on other machines. The solution to this is to update the
MathFunctions :command:`target_include_directories` to understand that it needs
different ``INTERFACE`` locations when being used from within the build
directory and from an install / package. This means converting the
:command:`target_include_directories` call for MathFunctions to look like:

.. literalinclude:: Step12/MathFunctions/CMakeLists.txt
  :language: cmake
  :start-after: # to find MathFunctions.h, while we don't.
  :end-before: # should we use our own math functions

Once this has been updated, we can re-run CMake and verify that it doesn't
warn anymore.

At this point, we have CMake properly packaging the target information that is
required but we will still need to generate a ``MathFunctionsConfig.cmake`` so
that the CMake :command:`find_package` command can find our project. So let's go
ahead and add a new file to the top-level of the project called
``Config.cmake.in`` with the following contents:

.. literalinclude:: Step12/Config.cmake.in

Then, to properly configure and install that file, add the following to the
bottom of the top-level ``CMakeLists.txt``:

.. literalinclude:: Step12/CMakeLists.txt
  :language: cmake
  :start-after: # install the configuration targets
  :end-before: # generate the export

At this point, we have generated a relocatable CMake Configuration for our
project that can be used after the project has been installed or packaged. If
we want our project to also be used from a build directory we only have to add
the following to the bottom of the top level ``CMakeLists.txt``:

.. literalinclude:: Step12/CMakeLists.txt
  :language: cmake
  :start-after: # needs to be after the install(TARGETS ) command

With this export call we now generate a ``Targets.cmake``, allowing the
configured ``MathFunctionsConfig.cmake`` in the build directory to be used by
other projects, without needing it to be installed.

打包 Debug 和 Release (Step 12)
=====================================

**Note:** This example is valid for single-configuration generators and will
not work for multi-configuration generators (e.g. Visual Studio).

By default, CMake's model is that a build directory only contains a single
configuration, be it Debug, Release, MinSizeRel, or RelWithDebInfo. It is
possible, however, to setup CPack to bundle multiple build directories and
construct a package that contains multiple configurations of the same project.

First, we want to ensure that the debug and release builds use different names
for the executables and libraries that will be installed. Let's use `d` as the
postfix for the debug executable and libraries.

Set :variable:`CMAKE_DEBUG_POSTFIX` near the beginning of the top-level
``CMakeLists.txt`` file:

.. literalinclude:: Complete/CMakeLists.txt
  :language: cmake
  :start-after: project(Tutorial VERSION 1.0)
  :end-before: target_compile_features(tutorial_compiler_flags

And the :prop_tgt:`DEBUG_POSTFIX` property on the tutorial executable:

.. literalinclude:: Complete/CMakeLists.txt
  :language: cmake
  :start-after: # add the executable
  :end-before: # add the binary tree to the search path for include files

Let's also add version numbering to the MathFunctions library. In
``MathFunctions/CMakeLists.txt``, set the :prop_tgt:`VERSION` and
:prop_tgt:`SOVERSION` properties:

.. literalinclude:: Complete/MathFunctions/CMakeLists.txt
  :language: cmake
  :start-after: # setup the version numbering
  :end-before: # install rules

From the ``Step12`` directory, create ``debug`` and ``release``
subbdirectories. The layout will look like:

.. code-block:: none

  - Step12
     - debug
     - release

Now we need to setup debug and release builds. We can use
:variable:`CMAKE_BUILD_TYPE` to set the configuration type:

.. code-block:: console

  cd debug
  cmake -DCMAKE_BUILD_TYPE=Debug ..
  cmake --build .
  cd ../release
  cmake -DCMAKE_BUILD_TYPE=Release ..
  cmake --build .

Now that both the debug and release builds are complete, we can use a custom
configuration file to package both builds into a single release. In the
``Step12`` directory, create a file called ``MultiCPackConfig.cmake``. In this
file, first include the default configuration file that was created by the
:manual:`cmake  <cmake(1)>` executable.

Next, use the ``CPACK_INSTALL_CMAKE_PROJECTS`` variable to specify which
projects to install. In this case, we want to install both debug and release.

.. literalinclude:: Complete/MultiCPackConfig.cmake
  :language: cmake

From the ``Step12`` directory, run :manual:`cpack <cpack(1)>` specifying our
custom configuration file with the ``config`` option:

.. code-block:: console

  cpack --config MultiCPackConfig.cmake