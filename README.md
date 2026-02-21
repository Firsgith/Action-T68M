# iStoreOS 自动构建系统

![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-Active-brightgreen)
![License](https://img.shields.io/badge/License-MIT-blue)

基于 GitHub Actions 的 iStoreOS 固件自动编译与快速定制方案，无需本地编译环境，一键实现固件定制、编译和发布。

## 🌟 核心优势
- **自动化**：季度自动完整编译，无需人工干预
- **高效率**：Image Builder 快速定制（5-15分钟出固件）
- **低成本**：利用 GitHub 免费算力，无需本地服务器
- **易定制**：通过简单的配置文件即可增减软件包
- **可缓存**：Toolchain 缓存机制，大幅缩短后续编译时间

## 📁 项目结构
├── .config # iStoreOS 完整编译配置文件（核心）├── packages.txt # 快速定制用软件包列表├── .github/│ └── workflows/│ ├── build-full.yml # 工作流 1：季度完整编译（源码级）│ └── build-ib.yml # 工作流 2：Image Builder 快速定制└── README.md # 项目说明文档


## 🔧 工作流说明

### 工作流1：完整编译（季度自动）
基于源码全量编译，生成基础固件和 Image Builder 工具。

#### 触发方式
- **自动触发**：每季度首日（1/4/7/10月）北京时间 8:00 自动运行
- **手动触发**：GitHub 仓库 → Actions → "iStoreOS 完整编译（季度）" → Run workflow

#### 功能特性
- 全流程：Toolchain → Kernel → Packages → 固件镜像
- 自动缓存 Toolchain（首次编译 3-4 小时，后续 40-60 分钟）
- 产物自动发布到 Release（保留最近 10 个版本）
- 生成详细构建日志和更新记录

#### 输出产物
| 文件类型 | 说明 |
|----------|------|
| `*.img.gz` | 可直接刷写的固件镜像（压缩格式） |
| `*imagebuilder*.tar.xz` | Image Builder 工具包（快速定制核心） |
| `config.buildinfo` | 本次构建的完整配置快照 |
| `update_log.md` | Git 提交历史与版本更新日志 |

### 工作流2：Image Builder 快速生成
基于已有 Image Builder 快速定制固件，无需重新编译内核，适合频繁调整软件包。

#### 触发方式
- **仅手动触发**：GitHub 仓库 → Actions → "Image Builder 快速生成" → Run workflow

#### 适用场景
- 快速增减/替换软件包（如 luci 应用、工具类软件）
- 测试不同软件包组合效果
- 基于稳定固件做轻量化/功能增强定制
- 避免全量编译的耗时等待

#### 输入参数（运行时配置）
| 参数名 | 说明 | 默认值 | 注意事项 |
|--------|------|--------|----------|
| `source` | Image Builder 来源 | `release` | release（推荐）/artifact/auto |
| `version_tag` | 指定版本标签 | 空 | 留空自动使用最新版本 |
| `extra_packages` | 额外添加的包 | 空 | 空格分隔多个包名 |
| `remove_packages` | 要移除的包 | 空 | 如：`luci-app-upnp luci-app-qbittorrent` |
| `create_release` | 是否发布到 Release | `true` | false 仅保存到 Artifacts（14天过期） |

#### Source 选项说明
- `release`：从 Release 下载（永久保存，推荐生产环境）
- `artifact`：从 Actions 产物下载（仅保留 30 天，适合测试）
- `auto`：优先 Release，失败自动回退到 Artifact

## 🚀 快速开始

### 前置条件
1. 拥有 GitHub 账号并 Fork 本仓库
2. 了解基础的 iStoreOS 编译配置（.config 文件）
3. 熟悉 OpenWRT/istoreos 软件包命名规则

### 步骤1：首次配置
#### 1.1 配置完整编译文件（.config）
从本地编译环境复制成熟的配置文件（或使用官方默认配置）：
```bash
# 本地操作：复制配置文件到仓库目录
cp /path/to/istoreos/.config /your/github/repo/.config

# 提交到仓库
1.2 配置快速定制包列表（packages.txt）
创建软件包清单，支持添加 / 移除语法：
# 注释：以 # 开头
# 基础系统
luci luci-compat luci-base

# 网络功能
luci-app-firewall luci-app-opkg
luci-app-dockerman docker docker-compose

# 存储工具
smartmontools parted fdisk

# 移除包：前缀加 -
-dnsmasq          # 移除默认dnsmasq
+dnsmasq-full     # 替换为完整版
-luci-app-upnp    # 移除UPnP应用
步骤 2：运行完整编译（首次必做）
进入仓库的 Actions 页面
选择 "iStoreOS 完整编译（季度）"
点击 "Run workflow"，确认参数后运行
等待编译完成（首次 3-4 小时，后续有缓存会更快）
在 Release 页面查看生成的固件和 Image Builder
步骤 3：运行快速定制
编辑 packages.txt 调整软件包列表
进入 Actions → "Image Builder 快速生成"
保持默认参数（或按需调整），点击运行
等待 5-15 分钟，完成后：
临时下载：Actions 运行结果 → Artifacts
永久保存：Release 页面（create_release=true 时）
⚙️ 配置详解
.config 文件
作用：控制完整编译的所有参数（内核、软件包、分区等）
修改影响：修改后 Toolchain 缓存失效，需重新全量编译
建议：尽量使用稳定配置，避免频繁修改
来源：可从本地编译环境导出，或参考官方配置
packages.txt 文件
作用：快速定制时的软件包清单
支持语法：
直接写包名：添加该包（如 luci-app-dockerman）
-包名：移除该包（如 -luci-app-upnp）
+包名：强制添加（替换冲突包时用）
注意事项：
❌ 不要包含：kmod-*（内核模块）、busybox 等基础组件
✅ 可以包含：luci-app-*、用户工具、网络应用等
📌 版本命名规则
表格
工作流类型	Tag 格式	示例
完整编译	{分支名}-{构建编号}	istoreos-24.10-123
快速定制	custom-{设备型号}-{构建编号}	custom-lyt_t68m-456
🚨 故障排查
问题 1：Cache not found（缓存未命中）
原因：首次运行、修改 .config、缓存过期（7 天）
解决：属于正常现象，系统会自动重新编译并生成新缓存
问题 2：找不到 Image Builder
原因：完整编译未成功运行，或 Artifact 已过期（30 天）
解决：
先手动运行一次完整编译工作流
确认 Release 中有 Image Builder 产物
快速定制时选择 source=release
问题 3：软件包编译 / 安装失败
原因：包依赖冲突、feeds 源更新、包名错误
解决：
检查 packages.txt 中的包名是否正确
移除冲突的软件包（如同时添加 dnsmasq 和 dnsmasq-full）
确认包名与 iStoreOS 版本匹配
问题 4：磁盘空间不足
原因：GitHub Actions 默认空间 14GB，编译需 40-60GB
解决：
工作流已配置自动清理临时文件
精简 .config，移除不必要的软件包
避免同时编译多个设备的固件
⚠️ 注意事项
依赖关系：快速定制（工作流 2）必须依赖完整编译（工作流 1）生成的 Image Builder
产物保留：
Release：完整编译 / 快速定制各保留最近 10 个版本（永久）
Artifact：完整编译保留 30 天，快速定制保留 14 天
缓存策略：Toolchain 缓存基于 .config 哈希值，修改配置会触发全量重新编译
使用限制：GitHub Actions 每月有免费运行时长限制，避免频繁全量编译
兼容性：不同设备的 .config 不通用，需为目标设备单独配置
📄 许可证
本项目采用 MIT 许可证开源 - 详见 LICENSE 文件
plaintext

### 总结
1. **核心优化点**：新增了徽章、核心优势、前置条件等实用模块，优化了排版和语法说明，补充了故障排查的具体解决步骤，让新手更容易上手。
2. **使用逻辑**：先完成完整编译生成基础固件和 Image Builder，再通过 Image Builder 快速定制，遵循 "先全量、后增量" 的使用流程。
3. **关键注意事项**：修改 .config 会导致缓存失效，快速定制依赖完整编译产物，产物保留周期不同需及时备份重要固件。

这份 README 既保持了原有核心功能说明，又增强了可读性和实用性，符合开源项目的文档规范，新手可以按步骤快速上手，有经验的用户也能快速找到需要的配置信息。

git add .config
git commit -m "初始化完整编译配置"
git push
