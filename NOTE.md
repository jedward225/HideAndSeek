# gpu_hideseek 安装笔记

本文档总结了在 `devcontainer` (Linux/Ubuntu) 环境下安装 `gpu_hideseek` 项目到目前为止的关键步骤、遇到的问题和解决方案。

## 1. 环境与前提条件

*   **操作系统:** Linux (Ubuntu on devcontainer)
*   **必要工具:**
    *   `git`: 用于克隆仓库和管理子模块。
    *   `cmake`: >= 3.24 (项目本身可能需要较新版本，但其依赖项需要兼容旧版本)。
    *   `python`: >= 3.9 (检测到用户环境为 3.12.9)。
    *   C/C++ 编译器: Madrona 会下载并使用其自带的 Clang 工具链 (检测到 Clang 17.0.2)。
*   **网络:** 可能需要配置代理才能访问 GitHub。

## 2. 安装步骤回顾与要点

1.  **克隆仓库 (必须带 `--recursive`)**:
    ```bash
    git clone --recursive https://github.com/shacklettbp/gpu_hideseek.git
    cd gpu_hideseek
    ```
    *   **注意:** ` --recursive` 标志对于拉取 `madrona` 等子模块至关重要。如果忘记了，需要手动更新。

2.  **(如果需要) 初始化/更新子模块**:
    如果在克隆时忘记 `--recursive`，或者子模块拉取失败，请在 `gpu_hideseek` 根目录下运行：
    ```bash
    git submodule update --init --recursive
    ```

3.  **(如果需要) 配置代理**:
    *   如果访问 GitHub (用于克隆子模块或下载依赖) 存在网络问题 (例如 `gnutls_handshake() failed` 错误)，需要配置代理。
    *   **为 Git 配置:**
        ```bash
        # 使用你的代理地址替换
        git config --global http.proxy http://your_proxy_address:port
        git config --global https.proxy http://your_proxy_address:port
        ```
    *   **为其他工具 (如 CMake 下载) 配置:** 可能需要设置环境变量（如用户已做的）：
        ```bash
        export http_proxy=http://your_proxy_address:port
        export https_proxy=http://your_proxy_address:port
        ```

4.  **创建构建目录**:
    ```bash
    mkdir build
    cd build
    ```

5.  **运行 CMake (带兼容性参数)**:
    ```bash
    cmake .. -DCMAKE_POLICY_VERSION_MINIMUM=3.5
    ```
    *   **注意:** 添加 `-DCMAKE_POLICY_VERSION_MINIMUM=3.5` 是为了解决 `meshoptimizer` 子模块的 CMake 版本兼容性错误。
    *   此步骤会下载 Madrona 工具链和预编译依赖。如果网络有问题且未配置代理环境变量，此步骤可能会失败。
    *   可能会出现关于 CMake < 3.10 的 *弃用警告* (Deprecation Warning)，这些目前可以忽略。

6.  **编译项目 (下一步)**:
    在 `build` 目录下运行：
    ```bash
    make -j<核心数> # 例如 make -j8
    ```

7.  **安装 Python 包 (编译后)**:
    回到项目根目录 (`gpu_hideseek`) 运行：
    ```bash
    pip install -e .
    ```
    *   **注意:** `-e` 表示以可编辑模式安装。

## 3. 遇到的主要问题及解决方案

1.  **CMake 错误: 找不到 `madrona_init.cmake` / `setup` / `dependencies` / 递归深度超限**
    *   **原因:** Git 子模块 (`external/madrona`) 没有被正确克隆或初始化。
    *   **解决:** 确保克隆时使用了 `--recursive` 标志，或者在项目根目录运行 `git submodule update --init --recursive`。

2.  **Git 子模块更新错误: `gnutls_handshake() failed`**
    *   **原因:** 通过 HTTPS 连接 GitHub 时发生 TLS/SSL 错误，通常与网络代理有关。
    *   **解决:** 使用 `git config --global http(s).proxy ...` 为 Git 明确配置代理。同时确保代理服务器允许访问 GitHub 且能处理 HTTPS。

3.  **CMake 错误: `Compatibility with CMake < 3.5 has been removed` (来自 `meshoptimizer`)**
    *   **原因:** `meshoptimizer` 依赖声明了过旧的 CMake 最低版本要求，而当前 CMake 版本不再完全兼容该声明方式。
    *   **解决:** 在运行 `cmake` 时添加参数 `-DCMAKE_POLICY_VERSION_MINIMUM=3.5` 强制使用旧策略。

## 4. 依赖版本小结

*   **Python:** 3.12.9 (满足 >= 3.9)
*   **CMake:** (用户版本未知，但需 >= 3.24 且能处理策略 3.5)
*   **C/C++ Compiler:** Clang 17.0.2 (来自 Madrona toolchain)
*   **PyTorch:** (根据 `README.md`，运行 `benchmark.py` 需要)

## 5. 后续步骤

1.  在 `build` 目录下运行 `make -j<cores>`。
2.  在 `gpu_hideseek` 根目录下运行 `pip install -e .`。
3.  参考 `README.md` 运行示例或基准测试 (例如 `python scripts/benchmark.py ...`)。 