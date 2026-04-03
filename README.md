# 智能云图库 (Intelligent Cloud Gallery)

本项目是一个基于 **Spring Boot 2.7** + **Vue 3** + **Vite** 的企业级协同智能云图库系统。它不仅提供基础的图片存储与管理，更深度集成了 AI 能力，并针对高并发协同和海量数据存储进行了架构优化。

---

## 🚀 项目核心亮点

### 1. 智能化图片处理 (AI-Powered)
- **AI 扩图 (Out-painting)**：集成阿里云 AI 视觉算法，支持对图片进行四周比例扩增，智能填充背景。
- **智能搜图**：
  - **主色调搜索**：通过算法提取图片主色，支持按 Hex 颜色值筛选。
  - **以图搜图**：基于图像特征指纹，实现相似图片检索。
- **自动标签/分类**：上传时自动识别图片特征，简化管理流程。

### 2. 高并发协同架构 (High Performance)
- **无锁队列协同**：利用 **LMAX Disruptor** 处理 WebSocket 编辑事件，确保多人在同一空间协作时的高吞吐量与低延迟。
- **多级缓存体系**：
  - **一级缓存 (Caffeine)**：本地缓存热点数据，减少网络开销。
  - **二级缓存 (Redis)**：分布式缓存，保障数据一致性。
- **分库分表**：使用 **ShardingSphere-JDBC** 对图片表进行水平拆分，解决单表数据量过大导致的性能瓶颈。

### 3. 企业级安全与权限 (Enterprise Security)
- **动态空间权限**：基于 **Sa-Token** 自定义注解 `@SaSpaceCheckPermission`，实现空间维度的 RBAC 权限控制。
- **团队协作**：支持创建团队空间，精细化管理成员权限（管理员、编辑者、查看者）。
- **图片审核流**：内置管理员审核机制，确保平台内容合规。

---

## 🛠️ 技术选型

### 后端架构
| 技术 | 说明 |
| :--- | :--- |
| **Spring Boot** | 核心脚手架 2.7.6 |
| **MyBatis Plus** | 持久层框架，支持多租户与自动填充 |
| **Sa-Token** | 登录认证、权限校验、多端登录 |
| **ShardingSphere** | 分库分表，提升海量数据查询效率 |
| **Disruptor** | 高性能无锁队列，用于 WebSocket 事件分发 |
| **Redis** | 分布式缓存与会话共享 |
| **COS** | 腾讯云对象存储，保证存储的高可用性 |
| **Knife4j** | Swagger 增强，自动生成交互式 API 文档 |

### 前端架构
| 技术 | 说明 |
| :--- | :--- |
| **Vue 3** | 组合式 API (Script Setup) |
| **TypeScript** | 强类型支持，提升代码可维护性 |
| **Ant Design Vue** | 企业级 UI 组件库 |
| **Pinia** | 现代化状态管理 |
| **Vite** | 下一代前端构建工具 |
| **Cropper.js** | 客户端图片裁剪与处理 |

---

## 📦 功能模块详述

### 📂 空间系统 (Space)
- **私有空间**：用户个人资料库，支持额度限制（图片数、存储量）。
- **团队空间**：多成员协作环境，支持成员邀请、权限变更。
- **空间分析**：通过 ECharts 实现存储占用、图片类型分布、标签热力图等可视化监控。

### 🖼️ 图片管理 (Picture)
- **上传方式**：
  - 本地上传：支持拖拽、预览、压缩。
  - URL 上传：一键抓取远程图片。
  - 批量导入：支持关键词批量抓取并自动入库。
- **图片处理**：在线裁剪、AI 扩图、元数据编辑。
- **多维搜索**：关键词、分类、标签、颜色、相似度。

### 👥 协作与安全
- **协同编辑**：WebSocket 实时显示“谁正在编辑此图片”，防止编辑冲突。
- **权限模型**：基于空间的独立权限体系，不干扰全局系统权限。

---

## 📂 目录结构预览

```text
├── sql/                        # 数据库初始化脚本
├── src/main/java/com/gallery/picture/
│   ├── annotation/             # 自定义权限注解
│   ├── aop/                    # 权限与日志切面
│   ├── controller/             # 接口层 (User, Picture, Space, etc.)
│   ├── manager/                # 封装第三方服务 (Cos, Upload, Auth)
│   ├── mapper/                 # MyBatis Mapper
│   ├── model/                  # 实体类、DTO、VO
│   ├── service/                # 业务逻辑层
│   └── utils/                  # 颜色转换、相似度算法工具
└── src/main/resources/
    └── application.yml         # 核心配置文件
```

---

## 🏁 快速上手

### 后端配置
1. **数据库**：运行 `sql/create_table.sql`。
2. **COS 配置**：在 `application.yml` 中填入腾讯云 COS 的 `secretId`、`secretKey`、`bucket` 和 `region`。
3. **AI 配置**：配置阿里云 DashScope 的 API Key 以开启 AI 扩图功能。
4. **运行**：执行 `PictureBackApplication`。

### 前端配置
1. 进入 `yu-picture-frontend`。
2. 安装依赖：`npm install`。
3. 启动：`npm run dev`。
4. 访问：`http://localhost:5173`。

---

## 📄 接口说明
项目集成了 **Knife4j**，启动后访问 `http://localhost:8123/api/doc.html` 即可查看完整的 API 列表及进行接口调试。
