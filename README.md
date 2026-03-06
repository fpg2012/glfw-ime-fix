# GLFW IME Fix

解决glfw在MacOS和Linux输入文本时的退格错误。

原版在输入法中按下退格，不仅会删除预编辑的文本，还会删除已经完成输入的文本。经过修正后，如果文本框中有预编辑文本，则不会把事件传递给glfw，避免退格键被触发两次。

## Compiling

克隆这个仓库，然后进入仓库目录。参考[glfw的编译指南](https://www.glfw.org/docs/latest/compile.html)。

```
mkdir build
cd build
cmake .. -DBUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=release
```

如果您正在使用wayland，minecraft可能会优先调用wayland 选项，但是目前并没有wayland支持，因此只能在编译时禁用wayland支持强制让游戏使用xwayland

使用以下指令来生成禁用wayland的编译模板：

```
cmake .. -DBUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=release -DGLFW_BUILD_WAYLAND=0
```

## For Minecraft

下面将minecraft安装文件夹记为 `${MINECRAFT}`。在Linux下一般是 `~/.minecraft`，在MacOS下一般是`~/Library/Application\ Support/minecraft`。

下面假定lwjql的版本为3.3.3（适用于mc 1.21.10）。如果不是这个版本，自行替换一下版本号。

在开始前，备份一下`${MINECRAFT}/libraries/org/lwjgl/lwjgl-glfw/3.3.3/`中的文件。

然后根据你的系统选择后续步骤：

### MacOS (Apple Silicon)

```bash
cd ${MINECRAFT}/libraries/org/lwjgl/lwjgl-glfw/3.3.3/
jar xf lwjgl-glfw-3.3.3-natives-macos-arm64.jar
mkdir repack
```

将解压出来的文件移动到刚创建的repack文件夹里（后续操作全部在repack文件夹下执行）。

```
mv macos META-INF -t repack
```

进入repack文件夹，用刚刚本项目编译出的dylib替换macos文件夹下的libglfw.dylib（文件名保持libglfw.dylib不变）。

```
mv <替换为 libglfw.3.4.dylib 的实际路径> macos/arm64/org/lwjgl/glfw/libglfw.dylib
```

替换后更新sha1 checksum

```
shasum macos/arm64/org/lwjgl/glfw/libglfw.dylib | cut -d' ' -f1 > META-INF/macos/arm64/org/lwjgl/glfw/libglfw.dylib.sha1
```

重新打包

```
jar cf lwjgl-glfw-3.3.3-natives-macos-arm64.jar .
mv lwjgl-glfw-3.3.3-natives-macos-arm64.jar ..
```

到此就完成了。

### Linux

```bash
cd ${MINECRAFT}/libraries/org/lwjgl/lwjgl-glfw/3.3.3/
jar xf lwjgl-glfw-3.3.3-natives-linux-x86.jar
mkdir repack
```

注: 某些情况下可能为`lwjgl-glfw-3.3.3-natives-linux.jar`,这种情况下只需要将所有`-x86`删掉即可

将解压出来的文件移动到刚创建的repack文件夹里（后续操作全部在repack文件夹下执行）。

```
mv linux META-INF -t repack
```

进入repack文件夹，用刚刚本项目编译出的动态链接库`.so`替换linux文件夹下的libglfw.so（文件名保持libglfw.so不变）。

```
mv <替换为 libglfw.3.4.so 的实际路径> linux/x64/org/lwjgl/glfw/libglfw.so
```

替换后更新sha1 checksum

```
shasum linux/x64/org/lwjgl/glfw/libglfw.so | cut -d' ' -f1 > META-INF/linux/x64/org/lwjgl/glfw/libglfw.so.sha1
```

重新打包

```
fastjar cf lwjgl-glfw-3.3.3-natives-linux-x64.jar .
mv lwjgl-glfw-3.3.3-natives-linux-x64.jar ..
```

到此就完成了。
