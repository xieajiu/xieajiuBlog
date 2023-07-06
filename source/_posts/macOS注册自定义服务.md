---
title: macOS注册自定义服务
date: 2023-07-05 17:07:36
tags:
  - MacOS
categories:
  - xieajiu
description: "macOS注册自定义服务"
---

### macOS注册自定义服务

在 macOS 上，您可以使用 `plist` 文件来注册和管理自定义服务。 `plist` 文件是一个 XML 格式的文件，用于描述和配置系统服务和应用程序的行为。

下面是在 macOS 上注册自定义服务的步骤：

1. 创建 `plist` 文件，用于描述您的自定义服务的行为。以下是一个示例 `plist` 文件的内容：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
   <plist version="1.0">
   <dict>
       <key>Label</key>
       <string>com.example.my-service</string>
       <key>ProgramArguments</key>
       <array>
           <string>/path/to/my-service</string>
           <string>--config=/path/to/config-file</string>
       </array>
       <key>RunAtLoad</key>
       <true/>
       <key>KeepAlive</key>
       <true/>
   </dict>
   </plist>
   ```

   在上面的示例中，`Label` 指定了服务的标识符，`ProgramArguments` 指定了要运行的命令和参数，`RunAtLoad` 指定服务是否在启动时自动启动，`KeepAlive` 指定服务是否应该在异常情况下自动重启。

2. 将 `plist` 文件复制到 `/Library/LaunchDaemons` 目录下，这是系统级别的服务目录。例如：

   ```bash
   sudo cp /path/to/my-service.plist /Library/LaunchDaemons/
   ```

   注意，由于 `LaunchDaemons` 目录是系统级别的目录，因此需要使用 `sudo` 命令以管理员权限进行复制。

3. 加载并启动服务，执行以下命令：

   ```bash
   sudo launchctl load /Library/LaunchDaemons/my-service.plist
   sudo launchctl start com.example.my-service
   ```

   以上命令将加载和启动您的自定义服务。

现在，您的自定义服务已经成功注册到 macOS 系统中，并可以使用 `launchctl` 命令来管理服务的状态，例如启动、停止、重启等操作。

### launchctl相关命令

`launchctl` 是 macOS 下用于管理守护进程（daemon）和代理（agent）的命令行工具。下面是一些常用的 `launchctl` 命令：

1. `launchctl list`：列出当前系统中所有的守护进程和代理。加上 `-x` 参数可以显示详细信息。
2. `launchctl load /path/to/plist`：加载指定的守护进程或代理。`/path/to/plist` 是守护进程或代理的属性列表文件（plist）的路径。
3. `launchctl unload /path/to/plist`：卸载指定的守护进程或代理。
4. `launchctl start com.example.daemon`：启动指定的守护进程或代理。`com.example.daemon` 是守护进程或代理的标识符（identifier）。
5. `launchctl stop com.example.daemon`：停止指定的守护进程或代理。
6. `launchctl status com.example.daemon`：查看指定的守护进程或代理的状态。

其中，`com.example.daemon` 这种标识符是守护进程或代理的唯一标识符，类似于 Unix 中的进程 ID（PID），可以在守护进程或代理的 plist 文件中找到。在使用 `launchctl` 命令时，需要使用这个标识符来指定要操作的守护进程或代理。