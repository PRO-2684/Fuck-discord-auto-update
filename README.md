# Fuck-discord-auto-update

**English** | [简体中文](./README_zh.md)

No more waiting for Discord to check for updates - start it **RIGHT AWAY**!

Tested version(s): `1.0.9021` (Windows)

## Step 1 - Replace shortcuts / links

- If you have shortcuts / links on your desktop or other folders, copy down the installation path of Discord. If not - find the path by yourself.
    - On Windows, for example, the path is typically `C:\Users\<username>\AppData\Local\Discord\`
- From now on, installation path will be denoted simply as `Discord/`
- Open the installation folder mentioned above and locate a folder named like `app-<version>`, where `<version>` represents the current version (a string of digits separated by dots).
- Find the `Discord` executable under that path and copy down its entire path.
    - On Windows, for example, the path is typically `C:\Users\<username>\AppData\Local\Discord\app-<version>\Discord.exe`
- Delete the shortcuts (if any) and replace them with shortcuts pointing to the path of `Discord` executable (as mentioned above).

## Step 2 - Delete updater (optional)

If you found an `Update.exe` or some executable like that under the installation path, feel free to delete it.

## Step 3 - Modify code

- Open `Discord/app-<version>/resources/` folder and backup `app.asar` under it.
- Unpack `app.asar` using Node's asar tool.
    - If you don't have Node installed and don't want to install it, you can download a pre-compiled standalone asar tool from [this repo](https://github.com/async3619/asar-exec/releases).
    - Example command (on Windows): `.\asar.exe e .\app.asar discord`
- Modify the unpacked folder's `app_bootstrap/splashScreen.js` file.
    - If you prefer viewing diff to accompish this step, you may checkout commit [`77f6660`](https://github.com/PRO-2684/Fuck-discord-auto-update/commit/77f6660d2441c7cd343f5f35f0ea79bcf4a56265) and omit the following sub-steps.
    - Find the following code:

        ```js
        async function updateUntilCurrent() {
          const retryOptions = {
            skip_host_delta: false,
            skip_module_delta: {}
          };
        ```

    - Add the following code snippet after it:

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

    - So that the code nearby becomes something like:

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

    - Also find the following code:

        ```js
        function initOldUpdater() {
          modulesListeners = {};
          addModulesListener(CHECKING_FOR_UPDATES, () => {
            console.log(`splashScreen: ${CHECKING_FOR_UPDATES}`);
            startUpdateTimeout();
            updateSplashState(CHECKING_FOR_UPDATES);
          });
        ```

    - Change it to:

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

    - You may review my sample file [splashScreen_original.js](./examples/splashScreen_original.js) and [splashScreen_patched.js](./examples/splashScreen_patched.js) as a reference.
- Re-pack the unpacked folder back to `app.asar` using Node's asar tool.
    - Example command (on Windows): `.\asar.exe p .\discord .\app.asar`
- Replace the original `Discord/app-<version>/resources/app.asar` with the one we just re-packed.

## Step 4 - Enjoy 🎉

That's it! Now you can launch Discord lightning fast, without having to wait for update checks.

## Epilogue

If this project helped you, feel free to give me a Star ⭐️. It's not only a recognition for me, but also helps more people to discover this project and to save them the trouble of waiting for update checks.
