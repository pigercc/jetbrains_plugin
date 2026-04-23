这是一个 JetBrains IntelliJ Platform 插件工程，主要技术栈是：

- Kotlin/JVM，源码在 src/main/kotlin/dev/sweep/assistant
- Gradle Kotlin DSL：build.gradle.kts
- JetBrains 插件配置：src/main/resources/META-INF/plugin.xml
- Protobuf：src/main/proto
- Gradle Wrapper：gradlew.bat / gradlew
- 当前插件 ID：dev.sweep.assistant
- 当前版本：1.29.3
- 构建目标 IDE：主要是 IntelliJ IDEA Community 2025.1
- 兼容范围：sinceBuild=241，untilBuild=253.*
- JDK 要求：17，我已确认本机是 Corretto 17，满足要求

如何编译生成插件

在 Windows PowerShell 里执行：

cd C:\Users\95497\IdeaProjects\jetbrains_plugin
.\gradlew.bat clean buildPlugin

生成后的插件 zip 通常在：

build\distributions\sweepai-1.29.3.zip

然后在 JetBrains IDE 中安装：

1. 打开 IDE
2. Settings / Plugins
3. 齿轮图标
4. Install Plugin from Disk...
5. 选择 build\distributions\*.zip
6. 重启 IDE

开发调试运行插件：

.\gradlew.bat runIde

我本地验证时发现的问题

我尝试运行了 Gradle。tasks 配置阶段能过，但 buildPlugin 两次长时间未完成，并且 Windows 报过：

The paging file is too small for this operation to complete

这个项目的 gradle.properties 内存配置比较激进：

kotlin.daemon.jvmargs=-Xmx8g
org.gradle.jvmargs=-Xmx4g -Dfile.encoding=UTF-8
org.gradle.workers.max=8
org.gradle.parallel=true

如果你机器内存或 pagefile 不够，建议先改成：

kotlin.daemon.jvmargs=-Xmx2g
org.gradle.jvmargs=-Xmx2g -Dfile.encoding=UTF-8
org.gradle.workers.max=2
org.gradle.parallel=false

然后再跑：

.\gradlew.bat --stop
.\gradlew.bat clean buildPlugin --no-configuration-cache --console=plain

另一个配置问题

settings.gradle.kts 里有：

include("untitled")

但当前工程没有 untitled 目录。Gradle 8.11 只是警告，Gradle 9 会失败。建议删除这一行，除非你确实需要这个子模块。

另外，bin/install 是 Bash 脚本，并且安装路径写的是 macOS 的 JetBrains 目录；在 Windows 上不要用它，直接用 Install Plugin from Disk... 安装 zip 更稳。