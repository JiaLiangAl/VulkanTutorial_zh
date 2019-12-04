这页面罗列了在开发Vulkan 应用程序时可能会遇到的普遍问题的解决方案。

* **我在核心验证层得到一个访问冲突错误**：确保MSI 的Afterburner / RivaTuner统计服务没有运行，因为它与Vulkan有一些兼容性问题。

* **我看不到来自验证层的任何信息/验证层不可用**：首先确保验证层在程序退出后保持终端打开，从而有机会打印错误。您可以在Visual Studio中按 Ctrl-F5 代替直接按 F5来做到这一点，在Linux中，就在一个终端中执行你的程序。如果仍然没有信息你也不确定验证层是否被开启，那么你应该确保你的Vulkan SDK 是正确地按[这些介绍](https://vulkan.lunarg.com/doc/view/1.1.106.0/windows/getting_started.html#user-content-verify-the-installation)安装的。也要确保你的SDK版本至少在1.1.106.0来支持`VK_LAYER_KHRONOS_validation`层。

* **vkCreateSwapchainKHR 在SteamOverlayVulkanLayer64.dll中引发一个错误**：

* **vkCreateSwapchainKHR triggers an error in SteamOverlayVulkanLayer64.dll**:这像是一个在Steam 测试版客户端中的一个兼容性问题。有几个可能的变通方法：

    * 选择退出Steam测试版程序
    * `DISABLE_VK_LAYER_VALVE_steam_overlay_1`环境变量设置成`1`
    * 删除注册表中`HKEY_LOCAL_MACHINE\SOFTWARE\Khronos\Vulkan\ImplicitLayers`下的Steam在Vulkan层之上的入口。
例如：

![](/images/steam_layers_env.png)
