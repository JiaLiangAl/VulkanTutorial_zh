## 关于

这个教程将教你使用[Vulkan](https://www.khronos.org/vulkan/)图形和计算API的基础知识。Vulkan是由[Khronos group](https://www.khronos.org/)(因OpenGL闻名)开发的新API，它提供了一个更好的现代图形卡抽象。这新的接口允许你更好的描述你程序的意图，与[OpenGL](https://en.wikipedia.org/wiki/OpenGL)和[Direct3D](https://en.wikipedia.org/wiki/Direct3D)等现有api相比，这可以带来更好的性能和更少的令人惊讶的驱动程序行为。Vulkan背后的理念与[Direct3D 12](https://en.wikipedia.org/wiki/Direct3D#Direct3D_12)和[Metal](https://en.wikipedia.org/wiki/Metal_(API))类似，但是Vulkan的优势在于它是完全跨平台的，允许你同时针对Windows、Linux和Android进行开发。

然而，您为这些好处所付出的代价是必须使用非常冗长的API。与图形API相关的每个细节都需要由应用程序从头开始设置，包括初始帧缓冲区的创建和对象(如缓冲区和纹理图像)的内存管理。图形驱动程序将减少很多的手动操作，这意味着您必须在应用程序中做更多的工作来确保正确的行为。

这里传达的信息是，Vulkan并不适合所有人。它的目标是那些对高性能计算机图形有热情并愿意投入一些工作的程序员。如果你更感兴趣的是游戏开发，而不是计算机图形，那么你可能会希望坚持使用OpenGL或Direct3D，它们不会很快被Vulkan取代。另外一种选择是使用像[Unreal Engine](https://en.wikipedia.org/wiki/Unreal_Engine#Unreal_Engine_4)或[Unity](https://en.wikipedia.org/wiki/Unity_(game_engine))这样的引擎，它将能够使用Vulkan,同时向您展示一个更高级别的API。


既然这样，让我们来谈谈学习本教程的一些先决条件:

* 一个与Vulkan兼容的显卡和驱动程序([NVIDIA](https://developer.nvidia.com/vulkan-driver), [AMD](http://www.amd.com/en-us/innovations/software-technologies/technologies-gaming/vulkan), [Intel](https://software.intel.com/en-us/blogs/2016/03/14/new-intel-vulkan-beta-1540204404-graphics-driver-for-windows-78110-1540))
* C++ 的经验(熟悉RAII，初始化列表)
* 一个百分之百支持C++17特性的编译(Visual Studio 2017+, GCC 7+, Or Clang 5+)
* 有3D计算机图形经验

本教程不假设您了解OpenGL或Direct3D概念,但是要求你了解3D计算机图形的基本知识。它将不会解译透视投影背后的数学知识，例如，有关计算机图形学概念的详细介绍，请参照[这本书](https://paroj.github.io/gltut/)。其他一些优秀的计算机图形资源也是如此：
* [Ray tracing in one weekend](https://github.com/petershirley/raytracinginoneweekend)
* [Physically Based Rendering book](http://www.pbr-book.org/)
* Vulkan在真正的开源引擎中使用[Quake](https://github.com/Novum/vkQuake) and [DOOM 3](https://github.com/DustinHLand/vkDOOM3)

你可以按你的意愿用C代替C++，但是你将不得不用一个不同的线性代数库而且你将在代码结构方面独立。我们将使用类和RAII等c++特性来组织逻辑和资源生存期。这里同样有一个为Rust开发者提供的[不同的版本的教程](https://github.com/bwasty/vulkan-tutorial-rs)

为了让使用其他编程语言的开发人员更容易理解，并获得一些使用基本API的经验，我们将使用原始的C API来使用Vulkan。但是，如果您使用的是c++，您可能更喜欢使用较新的[Vulkan-Hpp](https://github.com/KhronosGroup/Vulkan-Hpp)封装，该封装可以抽象一些没人愿意干的事，并有助于防止某些类型的错误。

## 电子书

如果你倾向于以电子书的形式阅读本教程，你可以在这里下载一个EPUB或者PDF版本：
* [EPUB](https://raw.githubusercontent.com/Overv/VulkanTutorial/master/ebook/Vulkan%20Tutorial%20en.epub)
* [PDF](https://raw.githubusercontent.com/Overv/VulkanTutorial/master/ebook/Vulkan%20Tutorial%20en.pdf)

## 教程结构

我们将首先概述Vulkan是如何工作的，以及如何在屏幕上显示第一个三角形。在你理解了它们在整个过程中的基本作用之后，所有小步骤的目的将变得更有意义。下一步，我们用[Vulkan SDK](https://lunarg.com/vulkan-sdk/),线性代数操作库[GLM library](http://glm.g-truc.net/)和窗口创建库[GLFW](http://www.glfw.org/)来搭建开发环境。本教程将介绍如何在Windows上用Visual Studio，以及在Ubuntu上用GCC来搭建开发环境。

之后，我们将实现渲染第一个三角形的Vulkan程序所必需的所有基本组件。每一章将大致遵循以下的结构：
* 介绍新的概念以及其目的
* 使用所有相关的API调用将其集成到程序中
* 将其抽象为辅助函数

虽然每一章都是前一章的后续，但也可以将每一章作为介绍Vulkan特性的独立文章来阅读。这意味着本网站也可以作为参考。所有的Vulkan函数和类型都链接到规范中，因此您可以单击它们来了解更多信息。Vulkan是非常新的API，因此，规范本身可能存在一些缺陷。我们鼓励您向[这个Khronos储存库](https://github.com/KhronosGroup/Vulkan-Docs)提交反馈。

如前所述，Vulkan API有一个非常详细的API，其中包含许多参数，可以最大限度地控制图形硬件。这导致创建纹理之类的基本操作每次都要重复很多步骤。因此，我们将创建自己的helper集合函数贯穿整个教程。

每一章最后都会有一个完整代码列表的链接。如果您对代码的结构有任何疑问，或者正在处理bug并希望进行比较，您可以参考它。所有的代码文件都已经在多个供应商的显卡上测试过，以验证其正确性。每一章的结尾都有一个评论部分，在那里你可以问任何与特定主题相关的问题。请指定您的平台、驱动程序版本、源代码、期望行为和实际行为来帮助我们帮助您。

本教程是社区的成果。Vulkan仍然是一个非常新的API，而且最佳实践还没有真正建立起来。如果您对教程和站点本身有任何类型的反馈，那么请毫不犹豫地提交问题或将请求拉到[GitHub repository](https://github.com/Overv/VulkanTutorial)。您可以*watch*本教程更新通知的存储库。

在你完成了在屏幕上绘制第一个Vulkan三角形的步骤后，我们将开始扩展程序，包括线性变换、纹理和3D模型。

如果您以前接触过图形api，那么您就会知道，在第一个几何图形出现在屏幕上之前，可能需要执行很多步骤。在Vulkan中有许多这样的初始步骤，但是您将看到每个单独的步骤都很容易理解，而且不会觉得多余。同样重要的是要记住，一旦你有了那个看起来很无聊的三角形，绘制完全纹理的3D模型并不需要很多额外的工作，每一步都比那个点更有意义。

如果您在学习本教程时遇到任何问题，那么首先检查FAQ，看看您的问题及其解决方案是否已经在那里列出。如果你在那之后仍然被卡住，那么请在最近相关章节的评论部分寻求帮助。

准备好深入研究高性能图形api的未来了吗?[我们走!](!zh-cn/Overview)
