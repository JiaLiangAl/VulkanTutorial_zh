## 一般结构
在前一章中，您已经使用所有适当的配置创建了一个Vulkan项目，并使用示例代码对其进行了测试。在本章中，我们将从头开始使用以下代码:

```c++
#include <vulkan/vulkan.h>

#include <iostream>
#include <stdexcept>
#include <functional>
#include <cstdlib>

class HelloTriangleApplication {
public:
    void run() {
        initVulkan();
        mainLoop();
        cleanup();
    }

private:
    void initVulkan() {

    }

    void mainLoop() {

    }

    void cleanup() {

    }
};

int main() {
    HelloTriangleApplication app;

    try {
        app.run();
    } catch (const std::exception& e) {
        std::cerr << e.what() << std::endl;
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```
我们首先包含LunarG SDK 的头文件，它提供函数，结构体和枚举。`stdexcept`和`iostream`头文件包含进来为了报告和传播错误。“functional”头文件将用于资源管理部分中的lambda函数。`cstdlib`头文件提供`EXIT_SUCCESS`和`EXIT_FAILURE`宏。

程序本身被封装到一个类中，在这个类中，我们将把Vulkan对象存储为私有类成员，并添加一些函数来初始化它们，这些函数将从`initVulkan`函数中调用。一旦所有的都准备好了，我们就进入主循环去渲染帧。我们将填充`mainLoop`函数，以包含一个循环，该循环迭代直到窗口在某一时刻关闭。一旦窗口关闭，`mainLoop`返回，我们将要确保在`cleanup`函数中释放我们用过的资源。

如果在执行过程中发生了任何致命错误，那么我们将抛出一个`std::runtime_error` 异常，并附带一个描述性消息，该消息将传播回`main`函数并打印到命令提示符。为了处理各种标准异常类型，我们捕获更一般的`std::exception`。我们将很快处理的一个错误示例是发现某个必需的扩展不受支持。

大致上，后面的每一章都会添加一个新函数，这个函数将从`initVulkan`中调用，一个或多个新的Vulkan对象将被添加到`cleanup`中需要释放的私有类成员中。

## 资源管理
就像分配给`malloc`的每一块内存都需要调用`free`一样，我们创建的每一个Vulkan对象都需要在不再需要时显式销毁。在现代c++代码中，可以通过`<memory>`头中的工具来实现自动的资源管理，但是在本教程中，我选择了对Vulkan对象的分配和回收进行显式说明。毕竟，Vulkan的宗旨是明确每个操作以避免错误，所以明确对象的生命周期有助于了解API的工作方式。

学习完本教程后，您可以通过重载`std::shared_ptr`来实现自动资源管理。使用[RAII](https://en.wikipedia.org/wiki/resource_tion_is_initial化)是应对大型Vulkan程序的推荐方法，但是为了便于学习，最好知道幕后发生了什么。

Vulkan 对象可以通过像`vkCreateXXX`的函数直接创建，也可以通过拥有像`vkAllocateXXX`函数的对象分配。当确认一个对象已经不会在任何地方用到之后，您需要使用对应的`vkDestroyXXX`和`vkFreeXXX`来销毁它。对于不同类型的对象，这些函数的参数通常是不同的，但是有一个参数是它们都共享的:`pAllocator`。这是一个可选参数，允许您为自定义内存分配器指定回调。我们将在教程中忽略这个参数，并始终传递`nullptr`作为参数。

## 集成 GLFW
Vulkan在没有创建窗口的情况下工作得非常好，如果你想在屏幕外使用它的渲染，但是它更令人兴奋的是实际显示一些东西!首先替换`#include <vulkan/vulkan.h>`这一行，用：

```c++
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>
```
这样，GLFW将包含自己的定义，并自动加载Vulkan标头。添加一个`initWindow`函数，并在其他调用之前从`run`函数向其添加一个调用。我们将使用该函数初始化GLFW并创建一个窗口。

```c++
void run() {
    initWindow();
    initVulkan();
    mainLoop();
    cleanup();
}

private:
    void initWindow() {

    }
```
在`initWindow`首先调用的必须是`glfwInit()`，它初始化GLFW库。因为GLFW最初是为了创建OpenGL上下文而设计的，我们需要告诉它不要用后续调用来创建OpenGL上下文:

```c++
glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
```
因为处理调整大小的窗口需要特别小心，我们将在后面讨论，现在用另一个窗口提示调用禁用它:

```c++
glfwWindowHint(GLFW_RESIZABLE, GLFW_FALSE);
```
现在剩下的就是创建这个真正的窗口。添加一个`GLFWwindow* window;`私有成员来保存它的引用，和初始化，用：

```c++
window = glfwCreateWindow(800, 600, "Vulkan", nullptr, nullptr);
```

前三个参数指定窗口的宽度、高度和标题。第四个参数允许您有选择地指定要打开窗口的监视器，最后一个参数只与OpenGL相关。

使用常量而不是硬编码的宽度和高度数是一个好主意，因为以后我们将多次引用这些值。我在`HelloTriangleApplication`类定义上面添加了以下代码:

```c++
const int WIDTH = 800;
const int HEIGHT = 600;
```
替换窗口的创建调用过程，用：

```c++
window = glfwCreateWindow(WIDTH, HEIGHT, "Vulkan", nullptr, nullptr);
```
你现在应该有一个`initWindow`功能，看起来像这样:

```c++
void initWindow() {
    glfwInit();

    glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
    glfwWindowHint(GLFW_RESIZABLE, GLFW_FALSE);

    window = glfwCreateWindow(WIDTH, HEIGHT, "Vulkan", nullptr, nullptr);
}
```
要保持应用程序运行，直到出现错误或窗口关闭，我们需要添加一个事件循环到`mainLoop`函数如下:

To keep the application running until either an error occurs or the window is
closed, we need to add an event loop to the `mainLoop` function as follows:

```c++
void mainLoop() {
    while (!glfwWindowShouldClose(window)) {
        glfwPollEvents();
    }
}
```
这段代码应该相当容易理解。它循环并检查像按X按钮这样的事件，直到用户关闭窗口。这也是我们稍后将调用一个函数来呈现单个帧的循环。

一旦窗口关闭，我们需要通过破坏它和终止GLFW本身来清理资源。这将是我们的第一个`cleanup`代码:

```c++
void cleanup() {
    glfwDestroyWindow(window);

    glfwTerminate();
}
```
当你现在运行程序时，你应该会看到一个名为`Vulkan`的窗口，直到关闭该窗口，应用程序才会终止。现在我们已经有了Vulkan应用程序的框架，让我们[创建第一个Vulkan对象](!zh-cn/Drawing_a_triangle/Setup/Instance)

[C++ code](/code/00_base_code.cpp)

