# 去你妈的 Discord 自动更新

[English](./README.md) | **简体中文**

不再需要等待 Discord 更新检查 - Discord，启动！

已测试版本：`1.0.9021` (Windows)

## 步骤 1 - 替换快捷方式 / 链接

- 如果您在桌面或其他文件夹上有快捷方式 / 链接，请复制它们指向的 Discord 的安装路径。 如果没有，请尝试自己找到路径。
    - 在 Windows 上，路径通常为 `C:\Users\<username>\AppData\Local\Discord\`
- 从现在开始，我们将使用 `Discord/` 来表示 Discord 的安装路径。
- 打开上述路径，然后寻找 `app-<version>` 文件夹，其中 `<version>` 是 Discord 当前版本号（一串用句点分隔的数字）。
- 在上述文件夹下找到 `Discord` 可执行文件，然后复制它的完整路径。
    - 在 Windows 上，路径通常为 `C:\Users\<username>\AppData\Local\Discord\app-<version>\Discord.exe`
- 删除快捷方式（若有）并用指向上述 `Discord` 可执行文件的快捷方式替换它们。

## 步骤 2 - 删除更新程序（可选）

如果您在安装路径下找到 `Update.exe` 或类似的可执行文件，可以删除它。

## 步骤 3 - 修改代码

- 打开 `Discord/app-<version>/resources/` 文件夹并备份其中的 `app.asar` 文件。
- 使用 Node 的 asar 工具解包 `app.asar`。
    - 如果您没有安装 Node 并且不想安装它，可以从 [此仓库](https://github.com/async3619/asar-exec/releases) 下载预编译的独立 asar 工具。
    - 示例命令（在 Windows 上）：`.\asar.exe e .\app.asar discord`
- 修改解包后的文件夹中的 `app_bootstrap/splashScreen.js` 文件。
    - 将原先的：

        ```js
        async function updateUntilCurrent() {
          const retryOptions = {
            skip_host_delta: false,
            skip_module_delta: {}
          };
        ```

    - 后面添加：

        ```js
          // HACK
          await newUpdater.startCurrentVersion();
          newUpdater.setRunningInBackground();
          newUpdater.collectGarbage();
          launchMainWindow();
          updateBackoff.succeed();
          updateSplashState(LAUNCHING);
          return;
        ```

    - 从而得到类似于：

        ```js
        async function updateUntilCurrent() {
          const retryOptions = {
            skip_host_delta: false,
            skip_module_delta: {}
          };

          // HACK
          await newUpdater.startCurrentVersion();
          newUpdater.setRunningInBackground();
          newUpdater.collectGarbage();
          launchMainWindow();
          updateBackoff.succeed();
          updateSplashState(LAUNCHING);
          return;

          while (true) {
            updateSplashState(CHECKING_FOR_UPDATES);
            try {
        ```

    - 这里：

        ```js
        function initOldUpdater() {
          modulesListeners = {};
          addModulesListener(CHECKING_FOR_UPDATES, () => {
            console.log(`splashScreen: ${CHECKING_FOR_UPDATES}`);
            startUpdateTimeout();
            updateSplashState(CHECKING_FOR_UPDATES);
          });
        ```

    - 改为：

        ```js
        function initOldUpdater() {
          modulesListeners = {};
          addModulesListener(CHECKING_FOR_UPDATES, () => {
            console.log(`splashScreen: ${CHECKING_FOR_UPDATES}`);

            // HACK
            console.log("Skipping update check for you...");
            moduleUpdater.setInBackground();
            launchMainWindow();
            updateSplashState(LAUNCHING);

            // startUpdateTimeout();
            updateSplashState(CHECKING_FOR_UPDATES);
          });
        ```

    - 你可以参考我的样例文件：[splashScreen_original.js](./examples/splashScreen_original.js) 和 [splashScreen_patched.js](./examples/splashScreen_patched.js)。
- 使用 asar 工具重新打包 `app.asar`。
    - 示例命令（在 Windows 上）：`.\asar.exe p .\discord .\app.asar`
- 用我们刚刚重新打包的 `app.asar` 替换原始的 `Discord/app-<version>/resources/app.asar` 文件。

## 步骤 4 - 尽情享受 🎉

大功告成！现在您可以快速启动 Discord 了，而无需等待更新检查完成。

## 后言

若此项目给了您帮助，欢迎给我一个 Star ⭐️。这不仅是对我的认可，也可以让更多人发现这个项目，免除等待更新检查的烦恼。
