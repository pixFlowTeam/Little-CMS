
![Little CMS](doc/logo-small.png)

# About Little CMS
[www.littlecms.com](https://www.littlecms.com)

Little CMS intends to be an **OPEN SOURCE** small-footprint color management engine, with special focus on accuracy and performance. It uses the International Color Consortium standard (ICC), which is the modern standard when regarding to color management. The ICC specification is widely used and is referred to in many International and other de-facto standards. It was approved as an International Standard, ISO 15076-1, in 2005. 



# Conformance
Little CMS is a **FULL IMPLEMENTATION** of ICC specification 4.4, it fully supports all kind of V2 and V4 profiles, including abstract, devicelink and named color profiles. Check the tutorial for a exhaustive list of features. 


# A bit of story
Since the initial release, back in 1998, Little CMS has grown to become one of the most popular open-source color management libraries, and has been used in a large number of production projects, in areas as printer firmware, monitors, digital cameras, RIPs, publishing, scientific, and many others. You can find Little CMS in most Linux distributions, and it's released under an open source license. 

### Please see the complete documentation in doc folder

## 构建说明

本项目提供了统一的多平台构建脚本，支持 Windows、macOS 和 Linux 平台。

### 快速开始
```bash
# 构建所有平台
./scripts/build-unified.sh all

# 构建特定平台
./scripts/build-unified.sh macos-arm64
```

### 详细文档
- [构建说明](docs/BUILD.md) - 完整的构建指南
- [快速参考](快速参考.md) - 常用命令和选项
