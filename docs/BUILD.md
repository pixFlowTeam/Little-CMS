# Little-CMS 构建说明

## 概述

Little-CMS 是一个开源的色彩管理库，支持 ICC 配置文件处理。本项目提供了统一的多平台构建脚本，支持 Windows、macOS 和 Linux 平台的交叉编译。

## 构建系统

Little-CMS 使用标准的 GNU Autotools 构建系统（autoconf + automake + libtool），并提供了统一构建脚本 `scripts/build-unified.sh` 来简化多平台构建过程。

## 快速开始

### 构建所有平台
```bash
./scripts/build-unified.sh all
```

### 构建特定平台
```bash
# macOS ARM64
./scripts/build-unified.sh macos-arm64

# macOS x64
./scripts/build-unified.sh macos-x64

# Windows x64
./scripts/build-unified.sh windows-x64

# Linux x64
./scripts/build-unified.sh linux-x64
```

## 构建选项

### 基本选项
- `--clean`: 清理构建目录
- `--verbose`: 详细输出
- `--help`: 显示帮助信息

### 功能选项
- `--static`: 构建静态库（默认）
- `--shared`: 构建动态库
- `--no-jpeg`: 禁用 JPEG 支持
- `--no-tiff`: 禁用 TIFF 支持
- `--no-zlib`: 禁用 ZLIB 支持
- `--no-threads`: 禁用线程支持
- `--no-sse2`: 禁用 SSE2 优化

### 示例
```bash
# 构建动态库版本
./scripts/build-unified.sh macos-arm64 --shared

# 构建静态库，禁用 JPEG 支持
./scripts/build-unified.sh windows-x64 --static --no-jpeg

# 清理并重新构建
./scripts/build-unified.sh all --clean --verbose
```

## 构建输出

构建完成后，生成的文件将位于 `build/<平台>/install/` 目录中：

```
build/
├── macos-arm64/install/
│   ├── lib/
│   │   ├── liblcms2.a          # 静态库
│   │   └── liblcms2.2.dylib    # 动态库
│   ├── bin/
│   │   ├── transicc            # 转换工具
│   │   ├── linkicc             # 链接工具
│   │   └── psicc               # 配置文件工具
│   └── include/
│       ├── lcms2.h             # 主头文件
│       └── lcms2_plugin.h      # 插件头文件
├── macos-x64/install/
├── windows-x64/install/
└── linux-x64/install/
```

## 依赖要求

### 系统依赖
- **macOS**: Xcode Command Line Tools
- **Windows**: MinGW-w64 或 Visual Studio
- **Linux**: GCC 和开发工具

### 交叉编译工具链

#### macOS 上安装交叉编译工具
```bash
# Windows x64 交叉编译工具
brew install mingw-w64

# Linux x64 交叉编译工具
brew install x86_64-linux-gnu-binutils
brew install FiloSottile/musl-cross/musl-cross
```

#### 工具链说明
- **Windows x64**: 使用 MinGW-w64 工具链，生成 Windows DLL
- **Linux x64**: 使用 musl 交叉编译工具链，生成静态链接的 Linux 可执行文件
- **macOS**: 使用原生 clang 编译器

### 可选依赖
- **JPEG**: libjpeg-turbo 或 libjpeg
- **TIFF**: libtiff
- **ZLIB**: zlib

## 手动构建

如果需要手动构建，可以使用传统的 Autotools 方式：

```bash
# 配置
./configure --prefix=/path/to/install

# 编译
make

# 安装
make install
```

### 配置选项
- `--enable-static`: 构建静态库
- `--enable-shared`: 构建动态库
- `--with-jpeg`: 启用 JPEG 支持
- `--with-tiff`: 启用 TIFF 支持
- `--with-zlib`: 启用 ZLIB 支持
- `--with-threads`: 启用线程支持
- `--disable-sse2`: 禁用 SSE2 优化

## 交叉编译

### Windows 交叉编译
需要安装 MinGW-w64 工具链：
```bash
# macOS 上安装
brew install mingw-w64

# 验证安装
x86_64-w64-mingw32-gcc --version

# 然后运行
./scripts/build-unified.sh windows-x64
```

### Linux 交叉编译
需要安装 musl 交叉编译工具链：
```bash
# macOS 上安装
brew install x86_64-linux-gnu-binutils
brew install FiloSottile/musl-cross/musl-cross

# 验证安装
x86_64-linux-musl-gcc --version

# 然后运行
./scripts/build-unified.sh linux-x64
```

### 交叉编译优势
- **统一构建环境**: 在 macOS 上构建所有平台
- **无系统依赖**: Linux 版本使用 musl libc，完全自包含
- **生产就绪**: 生成的库文件可直接用于生产环境

## 故障排除

### 常见问题

1. **config.guess 找不到**
   - 确保在项目根目录运行脚本
   - 检查 config.guess 文件是否存在

2. **交叉编译工具链缺失**
   - 安装相应的交叉编译工具
   - 检查 PATH 环境变量

3. **依赖库缺失**
   - 安装相应的开发库
   - 使用 `--no-*` 选项禁用不需要的功能

### 调试
使用 `--verbose` 选项查看详细的构建过程：
```bash
./scripts/build-unified.sh macos-arm64 --verbose
```

## 版本信息

- **Little-CMS 版本**: 2.17
- **构建脚本版本**: 1.0
- **支持平台**: Windows x64, macOS ARM64, macOS x64, Linux x64

## 许可证

Little-CMS 使用双许可证：
- LGPL 2.1 或更高版本
- CDDL 1.0

详见 `LICENSE.LGPL` 和 `LICENSE.CDDL` 文件。
