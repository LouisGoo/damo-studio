# 新人上手代码库指南（Cube Studio）

本文面向第一次接触本仓库的同学，帮助你快速理解整体结构、关键模块与学习路径。

## 1. 这是什么项目

Cube Studio 是一个面向云原生机器学习场景的一站式平台，覆盖：

- 机器学习训练/推理全流程
- 任务流（Pipeline）编排
- 在线开发环境（Notebook/IDE）
- 多集群、多租户的资源与权限管理

可优先阅读仓库根目录 `README.md` 了解完整功能全貌与平台架构图。

## 2. 仓库整体结构（先看这 5 块）

```text
.
├── myapp/                # 平台后端（Flask + Flask-AppBuilder）
│   ├── views/            # 各业务模块的视图/API
│   ├── models/           # 核心数据模型（任务模板、Pipeline、服务等）
│   ├── tasks/            # Celery 异步任务和周期任务
│   ├── init/             # 首次初始化导入的数据（模板、项目、服务等）
│   ├── frontend/         # 主体前端工程（React + TS）
│   ├── vision/           # Pipeline 画布前端
│   └── visionPlus/       # ETL Pipeline 画布前端
├── job-template/         # 平台内置任务模板（镜像、启动器、示例）
├── install/              # 部署与开发脚本（docker、k8s）
├── images/               # 平台基础镜像定义
└── README.md             # 平台总览、能力地图、架构说明
```

## 3. 关键运行链路（后端）

### 3.1 启动入口与应用装配

- `myapp/__init__.py` 负责应用初始化：
  - Flask app、配置加载、数据库 SQLAlchemy、Migrate
  - AppBuilder（权限/菜单/管理界面能力）
  - 缓存、中间件、CORS、日志等全局能力
- `myapp/run.py` 是本地 debug 启动入口。

### 3.2 数据模型（业务实体）

- `myapp/models/model_job.py` 是高频核心模型所在文件之一：
  - `Repository` / `Images` / `Job_Template`
  - `Pipeline`（任务流定义）
  - 相关 workflow/run 等执行态模型

新人建议先理解：
- 「任务模板」如何定义镜像和参数
- 「Pipeline」如何描述 DAG 与调度参数

### 3.3 视图与 API（业务入口）

- `myapp/views/__init__.py` 中集中注册了多数业务模块：
  - 训练、模板、任务、数据集、模型、推理、工作流、AIHub、SQLLab 等
- 健康检查在 `myapp/views/route.py`（`/health`、`/ping`）。

### 3.4 异步任务与运维逻辑

- `myapp/tasks/celery_app.py` 创建全局 Celery app。
- `myapp/tasks/schedules.py` 承载大量系统巡检/清理/通知逻辑（例如过期资源清理、workflow 关联资源处理等）。

## 4. 前端结构（不要只盯一个前端目录）

本仓库有三套前端工程，职责不同：

1. `myapp/frontend`：主站 UI（菜单、页面、管理能力）
2. `myapp/vision`：机器学习 Pipeline 可视化编排
3. `myapp/visionPlus`：ETL Pipeline 可视化编排

因此看到“前端”问题时，先确认改动到底属于哪套工程。

## 5. 部署与本地开发重点

### 5.1 Docker 本地联调

- `install/docker/docker-compose.yml` 定义本地联调核心服务：`mysql`、`redis`、`myapp`、`frontend`。
- `install/docker/entrypoint.sh` 包含初始化流程：
  - 建库与迁移
  - 创建 admin
  - `myapp init`
  - 按 `STAGE` 分支启动开发/生产或构建流程

### 5.2 Kubernetes 生产部署

- `install/README.md` 和 `install/kubernetes/*` 提供完整云原生部署组件说明。

## 6. 任务模板机制（平台特色）

- `job-template/README.md` 说明了模板目录规范、镜像构建方式、参数定义规则。
- `myapp/init/init-job-template.json` 提供系统初始模板样例（字段设计很有参考价值）。

新人若做“新增算法任务/算子”，通常会同时涉及：

1) `job-template/job/<模板名>/...`（代码与镜像）
2) 平台中镜像/模板注册
3) Pipeline 中模板参数联动

## 7. 新人学习路径（建议 2~3 周）

### 第 1 周：先跑通，再看代码

1. 用 `install/docker/README.md` 跑通本地环境。
2. 登录后手动完成一遍：创建项目组 → 添加模板 → 创建 Pipeline → 运行任务。
3. 对照页面功能，定位后端对应 `views/` 与 `models/`。

### 第 2 周：抓主链路

1. 深入阅读：
   - `myapp/models/model_job.py`
   - `myapp/views/view_pipeline.py`
   - `myapp/tasks/schedules.py`
2. 理清“配置入库 → 提交 k8s 工作负载 → 状态回写/清理”闭环。

### 第 3 周：做一个小改动闭环

建议从以下类型挑一个：

- 新增一个任务模板字段并前后端打通
- 给某个列表页增加筛选项
- 增加一个健康检查或清理任务日志字段

目标不是“功能大”，而是验证你已打通：数据库模型、视图逻辑、前端展示、部署验证。

## 8. 常见踩坑提醒

- `myapp/config.py` 在仓库中是空文件，实际运行常通过环境挂载配置（如 `install/docker/config.py`）。
- 前端构建产物会汇总到 `myapp/static/appbuilder`，注意区分 dev 模式热更新与容器内 build。
- 本地调试常见问题集中在依赖安装、镜像网络、以及 k8s 连接配置。

## 9. 给新人的三个“先后顺序”

1. 先理解“业务对象”（项目、模板、Pipeline、服务）再看代码细节。
2. 先掌握“运行链路”（创建→调度→回写→清理）再谈抽象优化。
3. 先做小闭环改动（1~2天可验证）再接大型需求。

---

如果你是带新人同学，建议把本指南和一次屏幕共享 walkthrough 配合使用：
- 30 分钟讲架构
- 30 分钟跑本地
- 30 分钟做一个最小改动

效果通常比单纯“丢文档”更好。
