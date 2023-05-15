在本章中，我们将设置用于开发Vulkan应用程序的环境，并安装一些有用的库。除了编译器之外，我们将使用的所有工具都与Windows、Linux和MacOS兼容，但安装它们的步骤有点不同，所以我们会在这里分别介绍。

## Windows

如果您是为Windows开发的，那么我会假设您使用的是Visual Studio来编译代码。要获得完整的C++17支持，您需要使用Visual Studio 2017或2019。下面概述的步骤是为VS 2017编写的。

### Vulkan SDK

开发Vulkan应用程序所需的最重要的组件是SDK。它包括Vulkan函数的标头、标准验证层、调试工具和加载程序。加载程序在运行时查找驱动程序中的函数，类似于OpenGL的GLEW，如果你熟悉的话。

SDK可以从[LunarG网站](https://vulkan.lunarg.com/)下载，使用页面底部的按钮。您不必创建帐户，但它可以提供一些可能对您有用的附加文档。

![](/images/vulkan_sdk_download_buttons.png)

继续进行安装，并注意SDK的安装位置。我们要做的第一件事是验证您的显卡和驱动程序是否正确支持Vulkan。转到安装SDK的目录，打开`Bin`目录并运行`vkcube.exe`演示。您应该看到以下内容：

![](/images/cube_demo.png)

如果您收到错误消息，请确保您的驱动程序是最新的，包括Vulkan运行时，并且您的显卡是受支持的。有关主要供应商驱动程序的链接，请参阅[简介章节](!zh/Introduction)。

这个目录中还有另一个对开发有用的程序。`glslangValidator.exe`和`glslc.exe`将用于编译人类可读的着色器[GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language)到二进制编码。我们将在[着色器模块](!zh/Drawing_a_triangle/Graphics_pipeline_basics/Shader_modules)一章中对此进行深入介绍。`Bin`目录还包含Vulkan加载程序和验证层的二进制文件，而`Lib`目录包含库。

最后，还有`Include`目录，其中包含Vulkan标头。请随意浏览其他文件，但本篇教程不需要它们。

### GLFW

如前所述，Vulkan本身是一个平台无关的API，不包括用于创建窗口以显示渲染结果的工具。为了利于Vulkan的跨平台优势并避免Win32的繁琐，我们将使用[GLFW库](http://www.glfw.org/)创建一个支持Windows、Linux和MacOS的窗口。还有其他库可用于此目的，如[SDL](https://www.libsdl.org/)，但GLFW的优势在于，除了创建窗口之外，它还抽象了Vulkan中其他一些特定于平台的东西。

您可以在[官网](http://www.glfw.org/download.html)上找到GLFW的最新版本。

在本教程中，我们将使用64位二进制文件，但您当然也可以选择以32位模式构建。在这种情况下，请确保链接到`Lib32`目录中的Vulkan SDK二进制文件，而不是`Lib`目录。下载后，将文件提取到一个方便的位置。我已经选择在文档下的Visual Studio目录中创建一个`Libraries`目录。

![](/images/glfw_directory.png)

### GLM

与DirectX 12不同，Vulkan不包括用于线性代数运算的库，所以我们必须下载一个。[GLM](http://glm.g-truc.net/)是一个很好的库，设计用于图形API，也常用于OpenGL。

GLM是一个只有头文件的库，所以只需下载[最新版本](https://github.com/g-truc/glm/releases)并将其存放在方便的位置。您现在应该有一个类似于以下内容的目录结构：

![](/images/library_directory.png)

### 设置 Visual Studio

既然您已经安装了所有的依赖项，我们就可以为Vulkan设置一个基本的Visual Studio项目，并编写一些代码来确保一切正常。

启动Visual Studio，输入名称并按`确定`以创建一个新的`Windows桌面向导（Windows Desktop Wizard)`项目。

![](/images/vs_new_cpp_project.png)

请确保选择`Console Application (.exe)`作为应用程序类型，以便我们有一个打印调试消息的位置，并选中`空项目（Empty Project）`以防止Visual Studio添加样板代码。

![](/images/vs_application_settings.png)

按`确定`创建项目并添加C++源文件。您应该已经知道如何做到这一点，但为了完整起见，此处包含了这些步骤。

![](/images/vs_new_item.png)

![](/images/vs_new_source_file.png)

现在将以下代码添加到文件中。不要担心现在就试图理解它；我们只是确保您可以编译和运行Vulkan应用程序。我们将在下一章从头开始。

```c++
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>

#define GLM_FORCE_RADIANS
#define GLM_FORCE_DEPTH_ZERO_TO_ONE
#include <glm/vec4.hpp>
#include <glm/mat4x4.hpp>

#include <iostream>

int main() {
    glfwInit();

    glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
    GLFWwindow* window = glfwCreateWindow(800, 600, "Vulkan window", nullptr, nullptr);

    uint32_t extensionCount = 0;
    vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, nullptr);

    std::cout << extensionCount << " extensions supported\n";

    glm::mat4 matrix;
    glm::vec4 vec;
    auto test = matrix * vec;

    while(!glfwWindowShouldClose(window)) {
        glfwPollEvents();
    }

    glfwDestroyWindow(window);

    glfwTerminate();

    return 0;
}
```

现在让我们配置该项目以消除错误。打开项目属性对话框，确保选中`所有配置（All Configurations）`，因为大多数设置都适用于`Debug`和`Release`模式。

![](/images/vs_open_project_properties.png)

![](/images/vs_all_configs.png)

转到`C++ -> General -> Additional Include Directories` ，然后按下拉框中的`<Edit...>`。


![](/images/vs_cpp_general.png)

添加Vulkan、GLFW和GLM的头文件目录：

![](/images/vs_include_dirs.png)

接下来，打开`Linker -> General`下的库目录编辑器：

![](/images/vs_link_settings.png)

并添加Vulkan和GLFW的库文件的位置：

![](/images/vs_link_dirs.png)

打开`Linker -> Input`，然后在`Additional Dependencies`下拉框中按`<Edit...>` ：

![](/images/vs_link_input.png)

输入Vulkan和GLFW库文件：

![](/images/vs_dependencies.png)

最后更改编译器以支持C++17的功能：

![](/images/vs_cpp17.png)

现在可以关闭项目属性对话框。如果你做得很好，那么你就不应该再看到任何错误在代码中被突出显示。最后，请确保您是在64位模式下进行编译：

![](/images/vs_build_mode.png)

按`F5`编译并运行项目，您应该会看到一个命令提示符和一个弹出窗口，如下所示：

![](/images/vs_test_window.png)

The number of extensions should be non-zero. Congratulations, you're all set for
[playing with Vulkan](!en/Drawing_a_triangle/Setup/Base_code)!

扩展的数量应为非零。祝贺你，你已经做好了[playing with Vulkan](!zh/Drawing_a_triangle/Setup/Base_code)的准备。

## Linux

These instructions will be aimed at Ubuntu, Fedora and Arch Linux users, but you may be able to follow
along by changing the package manager-specific commands to the ones that are appropriate for you. You should have a compiler that supports C++17 (GCC 7+ or Clang 5+). You'll also need `make`.

### Vulkan Packages

The most important components you'll need for developing Vulkan applications on Linux are the Vulkan loader, validation layers, and a couple of command-line utilities to test whether your machine is Vulkan-capable:

* `sudo apt install vulkan-tools` or `sudo dnf install vulkan-tools`: Command-line utilities, most importantly `vulkaninfo` and `vkcube`. Run these to confirm your machine supports Vulkan.
* `sudo apt install libvulkan-dev` or `sudo dnf install vulkan-loader-devel` : Installs Vulkan loader. The loader looks up the functions in the driver at runtime, similarly to GLEW for OpenGL - if you're familiar with that.
* `sudo apt install vulkan-validationlayers-dev spirv-tools` or `sudo dnf install mesa-vulkan-devel vulkan-validation-layers-devel`: Installs the standard validation layers and required SPIR-V tools. These are crucial when debugging Vulkan applications, and we'll discuss them in the upcoming chapter.

On Arch Linux, you can run `sudo pacman -S vulkan-devel` to install all the
required tools above.

If installation was successful, you should be all set with the Vulkan portion. Remember to run
 `vkcube` and ensure you see the following pop up in a window:

![](/images/cube_demo_nowindow.png)

If you receive an error message then ensure that your drivers are up-to-date,
include the Vulkan runtime and that your graphics card is supported. See the
[introduction chapter](!en/Introduction) for links to drivers from the major
vendors.

### GLFW

As mentioned before, Vulkan by itself is a platform agnostic API and does not
include tools for creation a window to display the rendered results. To benefit
from the cross-platform advantages of Vulkan and to avoid the horrors of X11,
we'll use the [GLFW library](http://www.glfw.org/) to create a window, which
supports Windows, Linux and MacOS. There are other libraries available for this
purpose, like [SDL](https://www.libsdl.org/), but the advantage of GLFW is that
it also abstracts away some of the other platform-specific things in Vulkan
besides just window creation.

We'll be installing GLFW from the following command:

```bash
sudo apt install libglfw3-dev
```
or
```bash
sudo dnf install glfw-devel
```
or
```bash
sudo pacman -S glfw-wayland # glfw-x11 for X11 users
```

### GLM

Unlike DirectX 12, Vulkan does not include a library for linear algebra
operations, so we'll have to download one. [GLM](http://glm.g-truc.net/) is a
nice library that is designed for use with graphics APIs and is also commonly
used with OpenGL.

It is a header-only library that can be installed from the `libglm-dev` or
`glm-devel` package:

```bash
sudo apt install libglm-dev
```
or
```bash
sudo dnf install glm-devel
```
or
```bash
sudo pacman -S glm
```

### Shader Compiler

We have just about all we need, except we'll want a program to compile shaders from the human-readable  [GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language) to bytecode.

Two popular shader compilers are Khronos Group's `glslangValidator` and Google's `glslc`. The latter has a familiar GCC- and Clang-like usage, so we'll go with that: on Ubuntu, download Google's [unofficial binaries](https://github.com/google/shaderc/blob/main/downloads.md) and copy `glslc` to your `/usr/local/bin`. Note you may need to `sudo` depending on your permissions. On Fedora use `sudo dnf install glslc`, while on Arch Linux run `sudo pacman -S shaderc`.  To test, run `glslc` and it should rightfully complain we didn't pass any shaders to compile:

`glslc: error: no input files`

We'll cover `glslc` in depth in the [shader modules](!en/Drawing_a_triangle/Graphics_pipeline_basics/Shader_modules) chapter.

### Setting up a makefile project

Now that you have installed all of the dependencies, we can set up a basic
makefile project for Vulkan and write a little bit of code to make sure that
everything works.

Create a new directory at a convenient location with a name like `VulkanTest`.
Create a source file called `main.cpp` and insert the following code. Don't
worry about trying to understand it right now; we're just making sure that you
can compile and run Vulkan applications. We'll start from scratch in the next
chapter.

```c++
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>

#define GLM_FORCE_RADIANS
#define GLM_FORCE_DEPTH_ZERO_TO_ONE
#include <glm/vec4.hpp>
#include <glm/mat4x4.hpp>

#include <iostream>

int main() {
    glfwInit();

    glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
    GLFWwindow* window = glfwCreateWindow(800, 600, "Vulkan window", nullptr, nullptr);

    uint32_t extensionCount = 0;
    vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, nullptr);

    std::cout << extensionCount << " extensions supported\n";

    glm::mat4 matrix;
    glm::vec4 vec;
    auto test = matrix * vec;

    while(!glfwWindowShouldClose(window)) {
        glfwPollEvents();
    }

    glfwDestroyWindow(window);

    glfwTerminate();

    return 0;
}
```

Next, we'll write a makefile to compile and run this basic Vulkan code. Create a
new empty file called `Makefile`. I will assume that you already have some basic
experience with makefiles, like how variables and rules work. If not, you can
get up to speed very quickly with [this tutorial](https://makefiletutorial.com/).

We'll first define a couple of variables to simplify the remainder of the file.
Define a `CFLAGS` variable that will specify the basic compiler flags:

```make
CFLAGS = -std=c++17 -O2
```

We're going to use modern C++ (`-std=c++17`), and we'll set optimization level to O2. We can remove -O2 to compile programs faster, but we should remember to place it back for release builds.

Similarly, define the linker flags in a `LDFLAGS` variable:

```make
LDFLAGS = -lglfw -lvulkan -ldl -lpthread -lX11 -lXxf86vm -lXrandr -lXi
```

The flag `-lglfw` is for GLFW, `-lvulkan` links with the Vulkan function loader and the remaining flags are low-level system libraries that GLFW needs. The remaining flags are dependencies of GLFW itself: the threading and window management.

It is possible that the `Xxf68vm` and `Xi` libraries are not yet installed on your system. You can find them in the following packages:

```bash
sudo apt install libxxf86vm-dev libxi-dev
```
or
```bash
sudo dnf install libXi-devel libXxf86vm-devel
```
or
```bash
sudo pacman -S libxi libxxf86vm
```

Specifying the rule to compile `VulkanTest` is straightforward now. Make sure to
use tabs for indentation instead of spaces.

```make
VulkanTest: main.cpp
	g++ $(CFLAGS) -o VulkanTest main.cpp $(LDFLAGS)
```

Verify that this rule works by saving the makefile and running `make` in the
directory with `main.cpp` and `Makefile`. This should result in a `VulkanTest`
executable.

We'll now define two more rules, `test` and `clean`, where the former will
run the executable and the latter will remove a built executable:

```make
.PHONY: test clean

test: VulkanTest
	./VulkanTest

clean:
	rm -f VulkanTest
```

Running `make test` should show the program running successfully, and displaying the number of Vulkan extensions. The application should exit with the success return code (`0`) when you close the empty window. You should now have a complete makefile that resembles the following:

```make
CFLAGS = -std=c++17 -O2
LDFLAGS = -lglfw -lvulkan -ldl -lpthread -lX11 -lXxf86vm -lXrandr -lXi

VulkanTest: main.cpp
	g++ $(CFLAGS) -o VulkanTest main.cpp $(LDFLAGS)

.PHONY: test clean

test: VulkanTest
	./VulkanTest

clean:
	rm -f VulkanTest
```

You can now use this directory as a template for your Vulkan projects. Make a copy, rename it to something like `HelloTriangle` and remove all of the code in `main.cpp`.

You are now all set for [the real adventure](!en/Drawing_a_triangle/Setup/Base_code).

## MacOS

These instructions will assume you are using Xcode and the [Homebrew package manager](https://brew.sh/). Also, keep in mind that you will need at least MacOS version 10.11, and your device needs to support the [Metal API](https://en.wikipedia.org/wiki/Metal_(API)#Supported_GPUs).

### Vulkan SDK

The most important component you'll need for developing Vulkan applications is the SDK. It includes the headers, standard validation layers, debugging tools and a loader for the Vulkan functions. The loader looks up the functions in the driver at runtime, similarly to GLEW for OpenGL - if you're familiar with that.

The SDK can be downloaded from [the LunarG website](https://vulkan.lunarg.com/) using the buttons at the bottom of the page. You don't have to create an account, but it will give you access to some additional documentation that may be useful to you.

![](/images/vulkan_sdk_download_buttons.png)

The SDK version for MacOS internally uses [MoltenVK](https://moltengl.com/). There is no native support for Vulkan on MacOS, so what MoltenVK does is actually act as a layer that translates Vulkan API calls to Apple's Metal graphics framework. With this you can take advantage of debugging and performance benefits of Apple's Metal framework.

After downloading it, simply extract the contents to a folder of your choice (keep in mind you will need to reference it when creating your projects on Xcode). Inside the extracted folder, in the `Applications` folder you should have some executable files that will run a few demos using the SDK. Run the `vkcube` executable and you will see the following:

![](/images/cube_demo_mac.png)

### GLFW

As mentioned before, Vulkan by itself is a platform agnostic API and does not include tools for creation a window to display the rendered results. We'll use the [GLFW library](http://www.glfw.org/) to create a window, which supports Windows, Linux and MacOS. There are other libraries available for this purpose, like [SDL](https://www.libsdl.org/), but the advantage of GLFW is that it also abstracts away some of the other platform-specific things in Vulkan besides just window creation.

To install GLFW on MacOS we will use the Homebrew package manager to get the `glfw` package:

```bash
brew install glfw
```

### GLM

Vulkan does not include a library for linear algebra operations, so we'll have to download one. [GLM](http://glm.g-truc.net/) is a nice library that is designed for use with graphics APIs and is also commonly used with OpenGL.

It is a header-only library that can be installed from the `glm` package:

```bash
brew install glm
```

### Setting up Xcode

Now that all the dependencies are installed we can set up a basic Xcode project for Vulkan. Most of the instructions here are essentially a lot of "plumbing" so we can get all the dependencies linked to the project. Also, keep in mind that during the following instructions whenever we mention the folder `vulkansdk` we are refering to the folder where you extracted the Vulkan SDK.

Start Xcode and create a new Xcode project. On the window that will open select Application > Command Line Tool.

![](/images/xcode_new_project.png)

Select `Next`, write a name for the project and for `Language` select `C++`.

![](/images/xcode_new_project_2.png)

Press `Next` and the project should have been created. Now, let's change the code in the generated `main.cpp` file to the following code:

```c++
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>

#define GLM_FORCE_RADIANS
#define GLM_FORCE_DEPTH_ZERO_TO_ONE
#include <glm/vec4.hpp>
#include <glm/mat4x4.hpp>

#include <iostream>

int main() {
    glfwInit();

    glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
    GLFWwindow* window = glfwCreateWindow(800, 600, "Vulkan window", nullptr, nullptr);

    uint32_t extensionCount = 0;
    vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, nullptr);

    std::cout << extensionCount << " extensions supported\n";

    glm::mat4 matrix;
    glm::vec4 vec;
    auto test = matrix * vec;

    while(!glfwWindowShouldClose(window)) {
        glfwPollEvents();
    }

    glfwDestroyWindow(window);

    glfwTerminate();

    return 0;
}
```

Keep in mind you are not required to understand all this code is doing yet, we are just setting up some API calls to make sure everything is working.

Xcode should already be showing some errors such as libraries it cannot find. We will now start configuring the project to get rid of those errors. On the *Project Navigator* panel select your project. Open the *Build Settings* tab and then:

* Find the **Header Search Paths** field and add a link to `/usr/local/include` (this is where Homebrew installs headers, so the glm and glfw3 header files should be there) and a link to `vulkansdk/macOS/include` for the Vulkan headers.
* Find the **Library Search Paths** field and add a link to `/usr/local/lib` (again, this is where Homebrew installs libraries, so the glm and glfw3 lib files should be there) and a link to `vulkansdk/macOS/lib`.

It should look like so (obviously, paths will be different depending on where you placed on your files):

![](/images/xcode_paths.png)

Now, in the *Build Phases* tab, on **Link Binary With Libraries** we will add both the `glfw3` and the `vulkan` frameworks. To make things easier we will be adding the dynamic libraries in the project (you can check the documentation of these libraries if you want to use the static frameworks).

* For glfw open the folder `/usr/local/lib` and there you will find a file name like `libglfw.3.x.dylib` ("x" is the library's version number, it might be different depending on when you downloaded the package from Homebrew). Simply drag that file to the Linked Frameworks and Libraries tab on Xcode.
* For vulkan, go to `vulkansdk/macOS/lib`. Do the same for the both files `libvulkan.1.dylib` and `libvulkan.1.x.xx.dylib` (where "x" will be the version number of the the SDK you downloaded).

After adding those libraries, in the same tab on **Copy Files** change `Destination` to "Frameworks", clear the subpath and deselect "Copy only when installing". Click on the "+" sign and add all those three frameworks here aswell.

Your Xcode configuration should look like:

![](/images/xcode_frameworks.png)

The last thing you need to setup are a couple of environment variables. On Xcode toolbar go to `Product` > `Scheme` > `Edit Scheme...`, and in the `Arguments` tab add the two following environment variables:

* VK_ICD_FILENAMES = `vulkansdk/macOS/share/vulkan/icd.d/MoltenVK_icd.json`
* VK_LAYER_PATH = `vulkansdk/macOS/share/vulkan/explicit_layer.d`

It should look like so:

![](/images/xcode_variables.png)

Finally, you should be all set! Now if you run the project (remembering to setting the build configuration to Debug or Release depending on the configuration you chose) you should see the following:

![](/images/xcode_output.png)

The number of extensions should be non-zero. The other logs are from the libraries, you might get different messages from those depending on your configuration.

You are now all set for [the real thing](!en/Drawing_a_triangle/Setup/Base_code).
