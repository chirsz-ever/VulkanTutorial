在本章中，我们将设置用于开发Vulkan应用程序的环境，并安装一些有用的库。除了编译器之外，我们将使用的所有工具都与Windows、Linux和MacOS兼容，但安装它们的步骤有点不同，所以我们会在这里分别介绍。

## Windows

如果您是为Windows开发的，那么我会假设您使用的是Visual Studio来编译代码。要获得完整的C++17支持，您需要使用Visual Studio 2017或2019。下面概述的步骤是为VS 2017编写的。

### Vulkan SDK

开发Vulkan应用程序所需的最重要的组件是SDK。它包括Vulkan函数的标头、标准验证层、调试工具和加载程序。加载程序在运行时查找驱动程序中的函数，类似于OpenGL的GLEW，如果你熟悉的话。

SDK可以从[LunarG网站](https://vulkan.lunarg.com/)下载，使用页面底部的按钮。您不必创建帐户，但它可以提供一些可能对您有用的附加文档。

![](/images/vulkan_sdk_download_buttons.png)

继续进行安装，并注意SDK的安装位置。我们要做的第一件事是验证您的显卡和驱动程序是否正确支持Vulkan。转到安装SDK的目录，打开`Bin`目录并运行`vkcube.exe`演示。您应该看到以下内容：

![](/images/cube_demo.png)

如果您收到错误消息，请确保您的驱动程序是最新的，包括Vulkan Runtime，并且您的显卡是受支持的。有关主要供应商驱动程序的链接，请参阅[简介章节](!zh/Introduction)。

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

这些说明将针对Ubuntu、Fedora和Arch Linux用户，但您可以将包管理器特定的命令更改为适合您的命令。您应该有一个支持C++17（GCC 7+或Clang 5+）的编译器。您还需要 `make`。

### Vulkan Packages

在Linux上开发Vulkan应用程序所需的最重要组件是Vulkan加载程序、验证层和几个命令行实用程序，用于测试您的机器是否支持Vulkan：

* `sudo apt install vulkan-tools` 或 `sudo dnf install vulkan-tools`：命令行实用程序，最重要的是`vulkaninfo`和`vkcube`。运行这些以确认您的机器支持Vulkan。

* `sudo apt install libvulkan-dev` 或 `sudo dnf install vulkan-loader-devel`：安装Vulkan加载程序。加载程序在运行时查找驱动程序中的函数，类似于OpenGL的GLEW,如果你熟悉的话。

* `sudo apt install vulkan-validationlayers-dev spirv-tools` 或 `sudo dnf install mesa-vulkan-devel vulkan-validation-layers-devel`：安装标准验证层和所需的SPIR-V工具。这些在调试Vulkan应用程序时至关重要，我们将在下一章中讨论它们。

在Arch Linux上，您可以运行 `sudo pacman -S vulkan-devel` 来安装上面所需的所有工具。

如果安装成功，您应该已经做好了Vulkan部分的准备。请记住运行`vkcube`，并确保您在窗口中看到以下弹出窗口：

![](/images/cube_demo_nowindow.png)

如果您收到错误消息，请确保您的驱动程序是最新的，包括Vulkan Runtime，并且您的显卡是受支持的。有关主要供应商驱动程序的链接，请参阅[简介章节](!zh/Introduction)。

### GLFW

如前所述，Vulkan本身是一个平台无关的API，不包括用于创建窗口以显示渲染结果的工具。为了利于Vulkan的跨平台优势并避免Win32的繁琐，我们将使用[GLFW库](http://www.glfw.org/)创建一个支持Windows、Linux和MacOS的窗口。还有其他库可用于此目的，如[SDL](https://www.libsdl.org/)，但GLFW的优势在于，除了创建窗口之外，它还抽象了Vulkan中其他一些特定于平台的东西。

我们将通过以下命令安装GLFW：

```bash
sudo apt install libglfw3-dev
```
或者
```bash
sudo dnf install glfw-devel
```
或者
```bash
sudo pacman -S glfw-wayland # glfw-x11 for X11 users
```

### GLM
与DirectX 12不同，Vulkan不包括用于线性代数运算的库，所以我们必须下载一个。[GLM](http://glm.g-truc.net/)是一个很好的库，设计用于图形API，也常用于OpenGL。

GLM是一个只有头文件的库，它可以从 `libglm-dev` 或者 `glm-devel` 包安装:

```bash
sudo apt install libglm-dev
```
或者
```bash
sudo dnf install glm-devel
```
或者
```bash
sudo pacman -S glm
```

### 着色器编译

我们几乎已经有了我们所需要的一切，但是我们还需要一个将人类可读的编译着色器语音[GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language)翻译到二进制文件的程序。

两个流行的着色器编译器是Khronos组织的`glslangValidator`和Google的`glslc`。后者有一个熟悉的类似GCC和Clang的用法，所以我们将继续使用它。在Ubuntu上，下载谷歌的[非官方二进制文件](https://github.com/google/shaderc/blob/main/downloads.md)并将`glslc`复制到您的`/usr/local/bin`中。请注意，根据您的权限，您可能需要`sudo`。在Fedora上使用`sudo dnf install glslc`，而在Arch Linux上运行`sudo pacman-S shaderc`。要进行测试，请运行`glslc`，它应该会理所当然地抱怨我们没有传递任何着色器进行编译：

`glslc: error: no input files`

我们将在[着色器模块](!en/Drawing_a_triangle/Graphics_pipeline_basics/Shader_modules)一章中深入介绍`glslc`。


### 设置Makefile项目

现在您已经安装了所有的依赖项，我们可以为Vulkan设置一个基本的makefile项目，并编写一些代码来确保一切正常。

在方便的位置创建一个新目录，像是`VulkanTest`。创建一个名为`main.cpp`的源文件，并插入以下代码。不要担心现在就试图理解它；我们只是确保您可以编译和运行Vulkan应用程序。我们将在下一章从头开始。

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

接下来，我们将编写一个makefile来编译和运行这个基本的Vulkan代码。创建一个名为`Makefile`的新空文件。我假设您已经对makefile有了一些基本的经验，比如变量和规则是如何工作的。如果没有，你可以通过[本教程](https://makefiletutorial.com/)很快跟上进度。

我们将首先定义几个变量来简化文件的其余部分。定义一个`FLAGS`变量，该变量将指定基本编译器标志：

```make
CFLAGS = -std=c++17 -O2
```

我们将使用现代C++（`-std=C++17`），并将优化级别设置为O2。我们可以删除-O2以更快地编译程序，但我们应该记住将其放回发布版本。

类似地，在`LDFLAGS`变量中定义链接器标志：

```make
LDFLAGS = -lglfw -lvulkan -ldl -lpthread -lX11 -lXxf86vm -lXrandr -lXi
```

标志`-lglfw`用于GLFW，`-lvulkan`链接Vulkan函数加载程序，其余标志为GLFW所需的低级系统库。剩下的标志是GLFW本身的依赖项：线程和窗口管理。

有可能 `Xxf68vm` 和 `Xi` 库尚未安装在您的系统上。您可以在以下软件包中找到它们：

```bash
sudo apt install libxxf86vm-dev libxi-dev
```
或者
```bash
sudo dnf install libXi-devel libXxf86vm-devel
```
或者
```bash
sudo pacman -S libxi libxxf86vm
```

现在，指定要编译`VulkanTest`的规则很简单。请确保使用制表符（Tabs）而不是空格进行缩进。

```make
VulkanTest: main.cpp
	g++ $(CFLAGS) -o VulkanTest main.cpp $(LDFLAGS)
```

通过保存makefile并在带有`main.cpp`和`makefile`的目录中运行`make`，验证此规则是否有效。这应该会产生一个`VulkanTest`可执行文件。

现在我们将再定义两个规则，`test`和`clean`，前者将运行可执行文件，后者将删除已构建的可执行文件：

```make
.PHONY: test clean

test: VulkanTest
	./VulkanTest

clean:
	rm -f VulkanTest
```

运行`make-test`应显示程序成功运行，并显示Vulkan扩展的数量。关闭空窗口时，应用程序应退出，并返回成功返回代码（`0`）。现在，您应该有一个完整的生成文件，类似于以下内容：

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

现在，您可以将此目录用作Vulkan项目的模板。制作一个副本，将其重命名为`HelloTriangle`，并删除`main.cpp`中的所有代码。

现在，您已经为[真正的冒险](!zh/Drawing_a_triangle/Setup/Base_code)做好了准备。

## MacOS

这些说明将假设您正在使用Xcode和[Homebrew package manager](https://brew.sh/). 此外，请记住，您至少需要10.11版MacOS，并且您的设备需要支持[Metal API](https://en.wikipedia.org/wiki/Metal_(API)#Supported_GPUs)。

### Vulkan SDK

开发Vulkan应用程序所需的最重要的组件是SDK。它包括Vulkan函数的标头、标准验证层、调试工具和加载程序。加载程序在运行时查找驱动程序中的函数，类似于OpenGL的GLEW，如果你熟悉的话。

SDK可以从[LunarG网站](https://vulkan.lunarg.com/)下载，使用页面底部的按钮。您不必创建帐户，但它可以提供一些可能对您有用的附加文档。

![](/images/vulkan_sdk_download_buttons.png)


MacOS的SDK版本内部使用[MoltenVK](https://moltengl.com/). MacOS上没有对Vulkan的本机支持，因此MoltenVK实际上充当了一个层，将Vulkan-API调用转换为苹果的Metal图形框架。有了它，您可以充分利用Apple的Metal框架的调试和性能优势。

下载后，只需将内容提取到您选择的文件夹中（请记住，在Xcode上创建项目时需要引用它）。在提取的文件夹中，在`Applications`文件夹中，您应该有一些可执行文件，这些文件将使用SDK运行一些演示。运行`vkcube`可执行文件，您将看到以下内容：

![](/images/cube_demo_mac.png)

### GLFW

如前所述，Vulkan本身是一个平台无关的API，不包括用于创建窗口以显示渲染结果的工具。为了利于Vulkan的跨平台优势并避免Win32的繁琐，我们将使用[GLFW库](http://www.glfw.org/)创建一个支持Windows、Linux和MacOS的窗口。还有其他库可用于此目的，如[SDL](https://www.libsdl.org/)，但GLFW的优势在于，除了创建窗口之外，它还抽象了Vulkan中其他一些特定于平台的东西。

要在MacOS上安装GLFW，我们将使用Homebrew软件包管理器来获取`glfw`软件包：

```bash
brew install glfw
```

### GLM

与DirectX 12不同，Vulkan不包括用于线性代数运算的库，所以我们必须下载一个。[GLM](http://glm.g-truc.net/)是一个很好的库，设计用于图形API，也常用于OpenGL。

GLM是一个只有头文件的库，它可以从 `glm` 包安装:

```bash
brew install glm
```

### 设置 Xcode

现在所有的依赖项都安装好了，我们可以为Vulkan设置一个基本的Xcode项目。这里的大多数说明本质上都是很多“管道（plumbing）”，所以我们可以将所有依赖项链接到项目。此外，请记住，在以下说明中，每当我们提到文件夹`vulkansdk`时，我们指的是您提取Vulkan SDK的文件夹。

启动Xcode并创建一个新的Xcode项目。在将打开的窗口中，选择 Application > Command Line Tool。

![](/images/xcode_new_project.png)

选择 `Next`, 给项目起个名字并选择 `Language` 为 `C++`.

![](/images/xcode_new_project_2.png)

按`下一步`，项目应该已经创建。现在，让我们将生成的`main.cpp`文件中的代码更改为以下代码：

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

请记住，您还不需要了解所有这些代码，我们只是设置了一些API调用，以确保一切正常。

Xcode应该已经显示了一些错误，比如它找不到的库。我们现在将开始配置项目以消除这些错误。在*项目导航器* 面板上，选择您的项目。打开*生成设置*选项卡，然后：

* 找到 **Header Search Paths** 字段，并添加一个指向`/usr/local/include`的链接（这是Homebrew安装Header的地方，所以glm和glfw3头文件应该在那里），以及一个指向Vulkan头的`vulkansdk/macOS/include`的链接。
* 找到 **库搜索路径** 字段，并添加到`/usr/local/lib`的链接（同样，这是Homebrew安装库的地方，所以glm和glfw3 lib文件应该在那里）和到`vulkansdk/macOS/lib`的链接。

它应该是这样的（很明显，路径会根据您在文件上的位置而有所不同）：

![](/images/xcode_paths.png)

现在，在 *Build Phases* 选项卡中，在 **Link Binary With Libraries** 上，我们将添加`glfw3`和`vulkan`框架。为了让事情变得更容易，我们将在项目中添加动态库（如果你想使用静态框架，你可以查看这些库的文档）。

* 对于glfw，打开文件夹`/usr/local/lib`，你会发现一个类似于`libglfw.3.x.dylib`的文件名（“x”是库的版本号，它可能会有所不同，这取决于你从Homebrew下载包的时间）。只需将该文件拖动到Xcode上的链接框架和库选项卡即可。
* 对于vulkan，请转到`vulkansdk/macOS/lib`。对`libvulkan.1.dylib`和`libvulcan.1.x.xx.dylib`这两个文件执行相同操作（其中“x”将是您下载的SDK的版本号）。

添加这些库后，在 **Copy Files** 的同一选项卡中将`Destination`更改为"Frameworks"，清除子路径并取消选择"Copy only when installing"。单击"+"号，并在此处添加所有这三个框架。

您的Xcode配置应该如下所示：

![](/images/xcode_frameworks.png)

最后需要设置的是几个环境变量。在Xcode工具栏上，转到`Product` > `Scheme` > `Edit Scheme...`，并在`Arguments`选项卡中添加以下两个环境变量：

* VK_ICD_FILENAMES = `vulkansdk/macOS/share/vulkan/icd.d/MoltenVK_icd.json`
* VK_LAYER_PATH = `vulkansdk/macOS/share/vulkan/explicit_layer.d`

看起来应该是这样的：

![](/images/xcode_variables.png)

最后，你应该准备好了！现在，如果您运行项目（记住根据您选择的配置将生成配置设置为Debug或Release），您应该看到以下内容：

![](/images/xcode_output.png)

扩展的数量应为非零。其他日志来自库，根据您的配置，您可能会收到不同的消息。

现在，您已经为[真正的冒险](!zh/Drawing_a_triangle/Setup/Base_code)做好了准备
