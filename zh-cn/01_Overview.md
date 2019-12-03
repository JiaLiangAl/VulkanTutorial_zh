本章将从介绍Vulkan及其解决的问题开始。在那之后，我们要看看第一个三角形所需的要素。这将为您提供一个大的画面，以覆盖后面的每个章节。最后，我们将介绍Vulkan API的结构和一般使用模式。


## Vulkan之起源

就像以前的图形api一样，Vulkan被设计成一个基于[GPUs](https://en.wikipedia.org/wiki/Graphics_processing_unit)的跨平台抽象。大多数这些api的问题是，在它们被设计为主要限于可配置的固定功能的特色图形硬件的时代。程序员必须以标准格式提供顶点数据，并且在光照和阴影选项方面受GPU制造商的支配。

随着显卡架构的成熟，它们开始提供越来越多的可编程功能。所有这些新功能都必须以某种方式与现有的api集成。这导致了不太理想的抽象和图形驱动程序方面的大量猜测，以将程序员的意图映射到现代图形架构。这就是为什么有这么多的驱动程序更新来提高游戏的性能，有时会有很大的提高。由于这些驱动程序的复杂性，应用程序开发人员还需要处理供应商之间的不一致，比如[shaders](https://en.wikipedia.org/wiki/Shader)接受的语法。除了这些新功能外，过去十年还出现了大量具有强大图形硬件的移动设备。这些移动gpu根据它们的能量和空间需求有不同的架构。一个这样的例子是[tiled rendering](https://en.wikipedia.org/wiki/Tiled_rendering)，它可以通过为程序员提供对该功能的更多控制来提高性能。这些api时代的另一个限制是对多线程的支持有限，这可能导致CPU端的瓶颈。

Vulkan通过为现代图形架构从零开始设计来解决这些问题。它允许程序员使用更详细的API明确地指定他们的意图，并允许多个线程并行地创建和提交命令，从而减少了驱动程序开销。它通过使用单个编译器切换到标准化的字节码格式来减少着色器编译中的不一致性。最后，通过将图形和计算功能统一到一个API中，它认可了现代图形卡的通用处理功能。


## 画一个三角形需要什么

现在我们来看看在一个表现良好的Vulkan程序中渲染三角形所需要的所有步骤。这里介绍的所有概念都将在下一章中详细阐述。这只是给你们一个大的图景把所有的单独的组成部分联系起来。

### 步骤 1 -  实例和物理设备选择

Vulkan应用程序首先通过`VkInstance`设置Vulkan API。实例是通过描述您的应用程序和您将使用的任何API扩展来创建的。创建实例之后，你可以查询支持Vulkan的硬件，并选择一个或更多`VkPhysicalDevice` 进行操作。您可以查询诸如VRAM大小之类的属性和设备功能，以选择所需的设备，例如选择使用专用的图形卡。


### 步骤 2 - 逻辑设备和队列族

选择要使用的正解硬件设备后，您需要创建一个 `VkDevice` (逻辑设备)，在这里您需要更具体地描述将要使用的`VkPhysicalDeviceFeatures`，比如多视口渲染和64位浮点数。你还需要指定你要使用的队列族。像绘制命令和内存操作，大多数Vulkan完成操作，都是将它们提交到`VkQueue`来异步执行。队列是从队列族分配的，其中每个队列族在其队列中支持一组特定的操作。例如，可以为图形、计算和内存传输操作创建单独的队列族。队列族的可用性也可以作为物理设备选择中的一个区别因素。支持Vulkan的设备可能不提供任何图形功能，但是所有支持Vulkan的显卡一般都会支持我们感兴趣的所有队列操作。

### 步骤 3 - 窗口表面和交换链

除非你只对离屏渲染感兴趣，否则你需要创建一个窗口来呈现渲染的图像。窗口可以被本地平台API或者像[GLFW](http://www.glfw.org/)和[SDL](https://www.libsdl.org/)的库创建。在本教程中我们将使用GLFW，但更多关于它的信息在下一章。

我们需要另外两个组件来实际渲染到窗口：一个窗口表面(VkSurfaceKHR)和交换链(VkSwapchainKHR)。注意`KHR`后缀，意味着这些对象是Vulkan扩展的一部分。Vulkan API本身是完全平台无关的，这就是为什么我们需要使用标准化的WSI(窗口系统接口)扩展来与窗口管理器交互。表面是一个要呈现给窗口的跨平台抽象，通常通过提供对本机窗口句柄的引用来实例化，例如Windows上的` HWND `。幸运的是，GLFW库有一个内置函数来处理平台特定的细节。

交换链是渲染目标的集合。它的基本目的是确保我们当前渲染的图像不同于当前在屏幕上的。确保只显示完成的图像是重要的。每当我们想要绘制一帧时，我们必须要求交换链为我们提供一个要渲染的图像。当我们完成一帧的绘制后，图像将返回到交换链，以便在某个时刻显示到屏幕上。渲染目标的数量和向屏幕显示已完成图像的条件取决于当前模式。目前常见的模式是双缓冲(vsync)和三缓冲。我们将在交换链创建一章中对此进行研究。

一些平台允许您通过`VK_KHR_display`和`VK_KHR_display_swapchain`扩展，不用与任务窗口管理器交互，直接渲染到显示。例如，这允许您创建一个表示整个屏幕的表面，并可用于实现自己的窗口管理器。


### 步骤 4 - 图像视图和帧缓冲区

要绘制从交换链获取的图像，我们必须将其包装到VkImageView和VkFramebuffer中。图像视图引用要使用的图像的特定部分，帧缓冲区引用要用于颜色、深度和模板目标的图像视图。因为在交换链中可能有许多不同的图像，我们将先为它们每个创建一个图像视图和帧缓冲区，并在绘制时选择正确的一个。

### 步骤 5 - 渲染通道

Vulkan中的渲染通道描述了渲染操作中使用的图像类型、如何使用它们以及如何处理它们的内容。在我们初始的三角形渲染程序中，我们将告诉Vulkan我们将使用单个图像作为颜色目标，并且我们想在绘制操作前将其清理成纯色。渲染通道只描述图像的类型，而VkFramebuffer实际上将特定的图像绑定到这些插槽。

### 步骤 6 - 图形管线

Vulkan中的图形管线是通过创建VkPipeline对象来设置的。它使用VkShaderModule对象描述像视口尺寸、深度缓存操作和可编程状态这些图形卡上可配置的状态，VkShaderModule对象着色器字节码创建的。驱动程序还需要知道哪些渲染目标将在管道中使用，我们通过引用渲染通道来指定。

与现有api相比，Vulkan最独特的特性之一是几乎所有图形管道的配置都需要提前设置。

这意味着如果你想切换到一个不同的着色器或者稍微改变一下顶点布局，那么你需要完全重新创建图形管道。这意味着您必须为呈现操作所需的所有不同组合预先创建许多VkPipeline对象。只有像视口尺寸和清理颜色这些基本的配置可以动态的改变。所有的状态也需要明确地描述，比如没有默认的颜色混合状态。

好消息是，因为您所做的工作相当于提前编译和即时编译，所以驱动程序有更多的优化机会，运行时性能也更容易预测，因为像切换到不同的图形管道这样的大型状态变化会变得非常明确。

### 步骤 7 - 命令池和命令缓冲区

就像之前提到的，在Vulkan中我们想执行的许多操作，像绘制操作，都需要提交到一个队列。在提交之前，这些操作首先需要记录到VkCommandBuffer中。这些命令缓存是由一个关联到特定的队列族的`VkCommandPool`分配。我们需要记录一个具有以下操作的命令缓冲来绘制一个简单的三角形：
* 开始渲染管道
* 绑定图形管线
* 绘制三个顶点
* 结束渲染管道

因为帧缓存中的图像依赖交换链将提供给我们的特定图像，我们需要为每个可能的图像记录一个命令缓存并且在绘制时选择正确的一个。另一种方法是每帧都重新记录一次命令缓冲区，这样效率就不高了。

### 步骤 8 - 主循环

现在绘制命令已经交换到一个命令缓存中，主循环是非常简单的。我们首先用vkAcquireNextImageKHR从交换链中获取一个图像。然后，我们可以为该映像选择适当的命令缓冲区，并使用vkQueueSubmit执行它。最后，我们使用vkQueuePresentKHR将图像返回到交换链，以便显示到屏幕上。

提交到队列的操作是异步执行的。因此，我们必须使用像信号量这样的同步对象来确保正确的执行顺序。绘制命令缓冲区的执行必须设置为等待图像获取完成，否则可能会出现这样的情况:我们开始呈现一个仍在被读取以在屏幕上显示的图像。vkQueuePresentKHR调用需要等待渲染完成，为此我们将使用第二个信号量，在渲染完成后发出信号。

### 总结

这次旋风之旅将使您对绘制第一个三角形的工作有一个基本的了解。真实的程序包含事多的步骤，像分配顶点缓冲区，创建一致性缓冲区和上传纹理图像将会在后续章节涉及到。但我们将从简单的开始，因为Vulkan已经有了足够陡峭的学习曲线。注意，我们将通过最初在顶点着色器中嵌入顶点坐标而不是使用顶点缓冲区来作弊。这是因为管理顶点缓冲区需要先熟悉一些命令缓冲区。

所以简而言之，绘制第一个三角形，我们需要：
* 创建一个VkInstance
* 选择一个被支持的图形卡(VkPhysicalDevice)
* 为绘制和呈现创建一个VkDevice和VkQueue
* 创建一个窗口，窗口表面和交换链
* 将交换链图像包装到VkImageView中
* 创建一个指定渲染目标和渲染用法的渲染通道
* 为渲染通道创建帧缓冲
* 设置图形管线
* 使用绘制命令为每个可能的交换链映像分配和记录一个命令缓冲区
* 通过获取图像、提交正确的绘制命令缓冲区并将图像返回到交换链来绘制帧


有很多步骤，但是每个步骤的目的在接下来的章节中会变得非常简单和清晰。如果您对单个步骤与整个程序之间的关系感到困惑，那么您应该返回到本章。

## API 概念

本章最后将简要概述Vulkan API在较低级别上是如何构建的。

### 代码编写约定

所有的Vulkan函数，枚举量和结构都定义在`vulkan.h`头文件，这包含在由LunarG开发的[Vulkan SDK](https://lunarg.com/vulkan-sdk/)中。我们将在下一章详细研究安装这个SDK。

函数有小写的`vk`前缀，像枚举和结构体这样的类型是`Vk`前缀，枚举值以`VK_`为前缀。API大量使用结构来为函数提供参数。例如，对象创建通常遵循以下模式:

```c++
VkXXXCreateInfo createInfo = {};
createInfo.sType = VK_STRUCTURE_TYPE_XXX_CREATE_INFO;
createInfo.pNext = nullptr;
createInfo.foo = ...;
createInfo.bar = ...;

VkXXX object;
if (vkCreateXXX(&createInfo, nullptr, &object) != VK_SUCCESS) {
    std::cerr << "failed to create object" << std::endl;
    return false;
}
```

Vulkan中的许多结构要求您在`sType`成员中显式地指定结构类型。`pNext`成员可以指向一个扩展结构，在本教程中总是`nullptr`。创建或销毁对象的函数将有一个VkAllocationCallbacks参数，该参数允许您为驱动程序内存使用自定义分配器，在本教程中也将保留`nullptr`。

几乎所有的函数返回一个`VK_SUCCESS`或者其他码的VkResult。规范描述了每个函数能返回的错误码以及它们的含义。

### 验证层

前面提到，Vulkan是为高性能和低驱动开销而设计的。因此，默认情况下，它将包含非常有限的错误检查和调试功能。如果你做错了什么，驱动程序通常会崩溃，而不是返回一个错误代码，或者更糟的是，它似乎可以在你显卡上工作，而在其他显卡上完全失败。

Vulkan允许您通过一个称为*验证层*的特性来进行广泛的检查。验证层是可以插入到API和图形驱动程序之间的代码片段，用于执行额外的功能参数检查和跟踪内存管理问题。好的方面是，您可以在开发期间启用它们，然后在发布应用程序时完全禁用它们，开销为零。任何人都可以编写自己的验证层，但是LunarG的Vulkan SDK提供了一组标准的验证层，我们将在本教程中使用它们。您还需要注册一个回调函数来接收来自各层的调试消息。

因为Vulkan对每个操作都很明确，而且验证层非常广泛，所以与OpenGL和Direct3D相比，找出为什么屏幕是黑色的实际上要容易得多!

在我们开始编写代码之前，还有一个步骤，那就是[设置开发环境](!zh-cn/Development_environment)。
