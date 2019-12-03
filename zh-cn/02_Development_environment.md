本章我们将为开发Vulkan程序设置环境以及安装一些有用的库。除了编译器之外，我们将使用的所有工具都与Windows、Linux和MacOS兼容，但是安装它们的步骤略有不同，这就是为什么在这里分别介绍它们的原因。

## Windows

如果您正在为Windows开发，那么我将假设您正在使用Visual Studio 2017来编译您的代码。您也可以使用Visual Studio 2013或2015，但是步骤可能有些不同。

### Vulkan SDK

开发Vulkan程序您需要的最重要组件就是SDK。它包括头文件，标准验证层，调试工具和一个Vulkan函数加载器。加载器在运行时查找驱动程序中的函数，这类似于针对OpenGL的GLEW—如果您熟悉它的话。

SDK可以使用页面底部的按钮从[the LunarG website](https://vulkan.lunarg.com/)下载。您不必创建帐户，但它将为您提供一些可能对您有用的附加文档。

![](/images/vulkan_sdk_download_buttons.png)

执行完安装过程并且注意SDK的安装路径。要做的第一件事确认您的图形卡和驱动程序能够正确地支持Vulkan。转到您安装SDK的目录，打开`Bin`目录，运行`cube.exe`演示。您应该可以看到如下：

![](/images/cube_demo.png)

如果你得到一个错误信息，那么确认你的驱动是最新的，包括Vulkan运行时并且你的图形卡被支持。参照[introduction chapter](!zh-cn/Introduction)链接到主要供应商的驱动程序。

在这个目录下有另外一个对开发有用的程序。`glslangValidator.exe`和`glslc.exe`将用来编译着色器脚本从可读的[GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language)到字节码。我们将在[shader modules](!zh-cn/Drawing_a_triangle/Graphics_pipeline_basics/Shader_modules)章节里面深入介绍。`Bin`目录还包含Vulkan加载程序的二进制文件和验证层，而`Lib`目录包含库。

`Doc`目录包含关于Vulkan SDK的有用信息和一个离线版本的Vulkan的整个详细规范。最后，有一个包含Vulkan头文件的`Include`目录。您可以随意查看其他文件，但是本教程不需要它们。

### GLFW

正如前面提到过的，Vulkan本身是一个与平台无关的API，不包括创建窗口去显示渲染结果的工具。为了从Vulkan的跨平台优势中获益，并避免Win32的可怕之处，我们将用支持Windows, Linux和MacOS的[GLFW 库](http://www.glfw.org/) 来创建窗口。还有其他可用的库能够达到这个目的，像[SDL](https://www.libsdl.org/), 但是GLFW的优点是它还抽象了Vulkan中除窗口创建之外的其他一些特定于平台的东西。

你可以在[official website](http://www.glfw.org/download.html)找到GLFW的最新发布。在本教程中，我们将使用64位的二进制文件，但是当然你也可以选择在32位模式下构建的。那样的话，请确认链接`Lib32`目录下的Vulkan SDK 二进制文件代替`Lib`下的。把它下载下来后，解压到一个方便的位置。我选择在`文档`下的Visual Studio目录中创建一个`Libraries`目录。不要担心没有`libvc-2017`文件夹，`libvc-2015`文件夹是兼容的。

![](/images/glfw_directory.png)

### GLM

不像 DirectX 12, Vulkan 没有包含线性代数操作的库，所以，我们必须下载一个，[GLM](http://glm.g-truc.net/)是一个很好的库，它是为图形api设计的，通常也与OpenGL一起使用。

GLM是一个 header-only 的库，所以只需要下载[最新版本](https://github.com/g-truc/glm/releases),保存到一个方便的位置。你应该有一个与以下所示类似的目录结构：

![](/images/library_directory.png)

### 设置 Visual Studio

现在您已经安装了所有的依赖项，我们可以为Vulkan设置一个基本的Visual Studio项目，并编写一些代码来确保一切正常。

启动 Visual Studio,创建一个新的`Windows Desktop Wizard`工程并输出名称，点击`OK`。

![](/images/vs_new_cpp_project.png)

确保`Console Application(.exe)`被选择为应用程序类型，这样我们就有一个地方来打印调试消息，并检查`Empty Project`，以防止Visual Studio添加样板代码。


![](/images/vs_application_settings.png)

点`确定`创建工程和添加C++源文件。你应该已经知道怎么做到了，但是为了完整起见，这里包含了这些步骤。

![](/images/vs_new_item.png)

![](/images/vs_new_source_file.png)

现在添加以下代码到文件中。现在不用担心尝试去弄懂它；我们只是确保您可以编译并运行Vulkan程序。我们将在下一章从零开始。

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

    std::cout << extensionCount << " extensions supported" << std::endl;

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
现在，让我们配置项目以消除错误。打开工程属性对话框，确保`All Configurations`被选中，因为大多数设置同时应用于`Debug`和`Release`模式。

![](/images/vs_open_project_properties.png)

![](/images/vs_all_configs.png)

转到 `C++ -> General -> Additional Include Directories`,在下拉框中点击`<Edit...>`。

![](/images/vs_cpp_general.png)

为Vulkan，GLFW，GLM 添加文件路径：

![](/images/vs_include_dirs.png)

下一步，在`Linker -> General`下打开库路径编辑器：

![](/images/vs_link_settings.png)

然后为Vulkan和GLFW 添加库文件的位置:

![](/images/vs_link_dirs.png)

转到`Linker -> Input`，在 `Additional Dependencies` 下拉框中点击 `<Edit...>`。

![](/images/vs_link_input.png)

输入 Vulkan 和 GLFW 库文件名：

![](/images/vs_dependencies.png)

最后改变编译器支持c++ 17的特性:

![](../images/vs_cpp17.png)

您现在可以关闭工程属性对话模式了。如果你所有的都做对了，那么你应该不会在代码中看到任何高亮的错误信息。

最后，确保你是在64位模式下编译：

![](/images/vs_build_mode.png)

按`F5`来编译和运行工程，你应该会看到一个命令行和一个窗口弹出来，像这样：

![](/images/vs_test_window.png)

扩展的数目应该是非零的。恭喜你，你都准备好了[和 Vulkan 一起玩](!zh-cn/Drawing_a_triangle/Setup/Base_code)!

## Linux

这些说明是针对Ubuntu用户的，但是您可以自己编译LunarG SDK并将`apt`命令更改为适合您的包管理器命令。您应该已经安装了一个支持现代c++(4.8或更高版本)的GCC版本。您还需要CMake和make。

### Vulkan SDK

开发Vulkan应用程序最重要的组件是SDK。它包括头文件、标准验证层、调试工具和Vulkan函数的加载程序。加载器在运行时查找驱动程序中的函数，这类似于针对OpenGL的GLEW—如果您熟悉它的话。

SDK可以使用页面底部的按钮从[the LunarG website](https://vulkan.lunarg.com/)下载。您不必创建帐户，但它将为您提供一些可能对您有用的附加文档。

![](/images/vulkan_sdk_download_buttons.png)

在你下载的`.tar.gz`包文件所在的目录中打开一个终端，然后解压它：

```bash
tar -xzf vulkansdk-linux-x86_64-xxx.tar.gz
```
它将把SDK中的所有文件提取到工作目录中以SDK版本名命名的子目录中。将目录移动到一个方便的位置，并注意它的路径。在SDK的根目录下打开终端，它将包含像 ` build_examples.sh ` 这样的文件

SDK中的示例和稍后将用于程序的一个库依赖于XCB库。这是一个用于与X窗口系统交互的C库。它可以通过`libxcb1-dev`包安装到Ubuntu中。您还需要`xorg-dev`包附带的通用X开发文件。

```bash
sudo apt install libxcb1-dev xorg-dev
```
现在你可以构建SDK中Vulkan例子，执行：

```bash
./build_examples.sh
```
如果编译成功，你将得到一个`./examples/build/vkcube`可执行文件。在`examples/build`目录中运行`./vkcube`,确保你看到以下弹出窗口:

![](/images/cube_demo_nowindow.png)

如果你得到一个错误信息，那么确认你的驱动是最新的，包括Vulkan运行时并且你的图形卡被支持。参照[introduction chapter](!zh-cn/Introduction)链接到主要供应商的驱动程序。


### GLFW

正如前面提到过的，Vulkan本身是一个与平台无关的API，不包括创建窗口去显示渲染结果的工具。为了从Vulkan的跨平台优势中获益，并避免Win32的可怕之处，我们将用支持Windows, Linux和MacOS的[GLFW 库](http://www.glfw.org/) 来创建窗口。还有其他可用的库能够达到这个目的，像[SDL](https://www.libsdl.org/), 但是GLFW的优点是它还抽象了Vulkan中除窗口创建之外的其他一些特定于平台的东西。

我们将从源码安装GLFW而不是用安装包，是因为Vulkan的支持需要最近的版本。你可以在[official website](http://www.glfw.org/)找到源码。

将源码解压到一个方便的目录，在有像`CMakeLists.txt`这个文件的目录打开一个终端：

执行以下命令来生成makefile，并编译GLFW：

```bash
cmake .
make
```
你可能会看到以`Could NOT find Vulkan`开头的警告，但你可以安全的忽略这些信息。如果编译是成功的，那么你可以安装GLFW到系统库，执行：

```bash
sudo make install
```

### GLM

不像 DirectX 12, Vulkan 没有包含线性代数操作的库，所以，我们必须下载一个，[GLM](http://glm.g-truc.net/)是一个很好的库，它是为图形api设计的，通常也与OpenGL一起使用。

它是一个 header-only 库，可以通过`libglm-dev`包来安装：

```bash
sudo apt install libglm-dev
```

### 设置一个 makefile 工程

现在您已经安装了所有的依赖项，我们可以为Vulkan设置一个基本的Visual Studio项目，并编写一些代码来确保一切正常。

在方便的位置创建一个名字像`VulkanTest`的目录。创建一个名为`main.cpp`的源文件，插入如下代码。现在不用担心尝试去弄懂它；我们只是确保您可以编译并运行Vulkan程序。我们将在下一章从零开始。

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

    std::cout << extensionCount << " extensions supported" << std::endl;

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
下一步，我们将写一个makefile来编译和运行这基本的Vulkan代码。创建一个空的文件，命名为`Makefile`。我将假设您对于makefiles已经有一些基本的经验，比如变量和规则如何工作。如果你没有，通过[本教程](https://makefiletutorial.com/)，您可以非常快速地掌握速度。

我们首先定义两个变量来简化文件的其余部分。定义一个`VULKAN_SDK_PATH`变量，该变量指的是‘x86_64’目录在LunarG SDK中的位置，例如:


```make
VULKAN_SDK_PATH = /home/user/VulkanSDK/x.x.x.x/x86_64
```
确保用您自己的用户名代替`user`,用正确的版本号代替`x.x.x.x`。下面，定义一个`CFLAGS`变量，指定基本的编译器标识：

```make
CFLAGS = -std=c++17 -I$(VULKAN_SDK_PATH)/include
```
我们将用现代C++（`-std=c++17`），并且我们要能够定位到LunarG SDK中的`vulkan.h`。类似地，在变量 `LDFLAGS` 中定义链接器标识：

```make
LDFLAGS = -L$(VULKAN_SDK_PATH)/lib `pkg-config --static --libs glfw3` -lvulkan
```
第一个标识表明我们想在LunarG SDK的`x86_64/lib`目录中找到比如`libvulkan.so`这些库。第二个组件调用`pkg-config `来自动检索使用GLFW构建应用程序所需的所有链接器标记。最后，`-lvulkan `链接到LunarG SDK附带的Vulkan函数加载器。

现在可以直接指定编译`VulkanTest`的规则。确保使用制表符而不是空格进行缩进。

```make
VulkanTest: main.cpp
    g++ $(CFLAGS) -o VulkanTest main.cpp $(LDFLAGS)
```
通过保存makefile,在有`main.cpp`和`Makefile`的目录执行`make`来验证规则是否工作。这将得到一个`VulkanTest`的可执行文件。

现在我们定义另外两个规则，`test`和`clean`,前者将运行可执行文件，后者将删除一个构建的可执行文件:

```make
.PHONY: test clean

test: VulkanTest
    ./VulkanTest

clean:
    rm -f VulkanTest
```
你会发现`make clean`工作得非常好，但`make test`最可能失败的错误信息如下:

```text
./VulkanTest: error while loading shared libraries: libvulkan.so.1: cannot open shared object file: No such file or directory
```
那是因为`libvulkan.so`没有作为系统库安装。要缓解这个问题，通过环境变量`LD_LIBRARY_PATH`显式的指定库的加载路径：

```make
test: VulkanTest
    LD_LIBRARY_PATH=$(VULKAN_SDK_PATH)/lib ./VulkanTest
```
程序现在应该可以成功的运行，并且显示Vulkan的扩展数目。当你关闭这个空白窗口时，应用程序应该使用成功返回代码(`0 `)退出。然而，还有更多的变量需要设置。我们将开始使用验证层在Vulkan，你需要告诉Vulkan库从哪里加载这些使用`VK_LAYER_PATH `变量:

```make
test: VulkanTest
    LD_LIBRARY_PATH=$(VULKAN_SDK_PATH)/lib VK_LAYER_PATH=$(VULKAN_SDK_PATH)/etc/vulkan/explicit_layer.d ./VulkanTest
```
您现在应该有一个完整的makdfile，类似如下：

```make
VULKAN_SDK_PATH = /home/user/VulkanSDK/x.x.x.x/x86_64

CFLAGS = -std=c++17 -I$(VULKAN_SDK_PATH)/include
LDFLAGS = -L$(VULKAN_SDK_PATH)/lib `pkg-config --static --libs glfw3` -lvulkan

VulkanTest: main.cpp
    g++ $(CFLAGS) -o VulkanTest main.cpp $(LDFLAGS)

.PHONY: test clean

test: VulkanTest
    LD_LIBRARY_PATH=$(VULKAN_SDK_PATH)/lib VK_LAYER_PATH=$(VULKAN_SDK_PATH)/etc/vulkan/explicit_layer.d ./VulkanTest

clean:
    rm -f VulkanTest
```
您现在可以以这个目录作为你的vulkan工程的模板。做一份拷贝，重命名为比如`HelloTriangle`，删除`main.cpp`里面的所有代码。

在我们开始前，让我们浏览Vulkan SDK更多一些。这里有另外一个对开发非常有用的程序。`x86_64/bin/glslangValidator`和`x86_64/bin/glslc`用来将着色器脚本由可读的[GLSL](https://en.wikipedia.org/wiki/OpenGL_Shading_Language)编译成字节码。我们将在[shader modules](!zh-cn/Drawing_a_triangle/Graphics_pipeline_basics/Shader_modules)章节里面深入介绍。

`Doc`目录包含关于Vulkan SDK的有用信息和一个离线版本的Vulkan的整个详细规范。您可以随意查看其他文件，但是本教程不需要它们。

你现在完全准备好了[真实的冒险](!zh-cn/Drawing_a_triangle/Setup/Base_code)!


## MacOS

这些介绍将假定你正在用Xcode和[Homebrew包管理器](https://brew.sh/).还有，记住您至少需要MacOS 10.11版本，并且您的设置要支持[Metal API](https://en.wikipedia.org/wiki/Metal_(API)#Supported_GPUs)。


### Vulkan SDK

开发Vulkan应用程序最重要的组件是SDK。它包括头文件、标准验证层、调试工具和Vulkan函数的加载程序。加载器在运行时查找驱动程序中的函数，这类似于针对OpenGL的GLEW—如果您熟悉它的话。

SDK可以使用页面底部的按钮从[the LunarG website](https://vulkan.lunarg.com/)下载。您不必创建帐户，但它将为您提供一些可能对您有用的附加文档。

![](/images/vulkan_sdk_download_buttons.png)

MacOS内部的使用SDK版本是[MoltenVK](https://moltengl.com/)。在MacOS上没有对Vulkan本地的支持，所以MoltenVK实际上充当了一个层，将Vulkan API调用转换为苹果的Metal graphics框架。有了它，您可以利用Apple的Metal框架的调试和性能优势。


After downloading it, simply extract the contents to a folder of your choice (keep in mind you will need to reference it when creating your projects on Xcode). Inside the extracted folder, in the `Applications` folder you should have some executable files that will run a few demos using the SDK. Run the `cube` executable and you will see the following:

![](/images/cube_demo_mac.png)

### GLFW

As mentioned before, Vulkan by itself is a platform agnostic API and does not include tools for creation a window to display the rendered results. We'll use the [GLFW library](http://www.glfw.org/) to create a window, which supports Windows, Linux and MacOS. There are other libraries available for this purpose, like [SDL](https://www.libsdl.org/), but the advantage of GLFW is that it also abstracts away some of the other platform-specific things in Vulkan besides just window creation.

To install GLFW on MacOS we will use the Homebrew package manager. Vulkan support for MacOS is still not fully available on the current (at the time of this writing) stable version 3.2.1. Therefore we will install the latest version of the `glfw3` package using:

```bash
brew install glfw3 --HEAD
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

    std::cout << extensionCount << " extensions supported" << std::endl;

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

Now, in the *Build Phases* tab, on **Link Binary With Libraries** we will add both the `glfw3` and the `vulkan` frameworks. To make things easier we will be adding he dynamic libraries in the project (you can check the documentation of these libraries if you want to use the static frameworks).

* For glfw open the folder `/usr/local/lib` and there you will find a file name like `libglfw.3.x.dylib` ("x" is the library's version number, it might be different depending on when you downloaded the package from Homebrew). Simply drag that file to the Linked Frameworks and Libraries tab on Xcode.
* For vulkan, go to `vulkansdk/macOS/lib`. Do the same for the file both files `libvulkan.1.dylib` and `libvulkan.1.x.xx.dylib` (where "x" will be the version number of the the SDK you downloaded).

After adding those libraries, in the same tab on **Copy Files** change `Destination` to "Frameworks", clear the subpath and deselect "Copy only when installing". Click on the "+" sign and add all those three frameworks here aswell.

Your Xcode configuration should look like:

![](/images/xcode_frameworks.png)

The last thing you need to setup are a couple of environment variables. On Xcode toolbar go to `Product` > `Scheme` > `Edit Scheme...`, and in the `Arguments` tab add the two following environment variables:

* VK_ICD_FILENAMES = `vulkansdk/macOS/etc/vulkan/icd.d/MoltenVK_icd.json`
* VK_LAYER_PATH = `vulkansdk/macOS/etc/vulkan/explicit_layer.d`

It should look like so:

![](/images/xcode_variables.png)

Finally, you should be all set! Now if you run the project (remembering to setting the build configuration to Debug or Release depending on the configuration you chose) you should see the following:

![](/images/xcode_output.png)

The number of extensions should be non-zero. The other logs are from the libraries, you might get different messages from those depending on your configuration.

You are now all set for [the real thing](!zh-cn/Drawing_a_triangle/Setup/Base_code).
