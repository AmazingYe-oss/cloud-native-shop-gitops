# Cloud Native Shop GitOps 仓库

本仓库为云原生电商系统的 GitOps 配置仓库，存放所有 Kubernetes 部署配置与环境参数，由 Argo CD 自动同步到集群。

## 仓库结构

.
├── charts/                    # Helm Chart 源码
│   └── cloud-native-shop/     # 应用 Chart
├── environments/              # 各环境 values 配置
│   ├── dev/                   # 开发环境
│   ├── test/                  # 测试环境
│   └── prod/                  # 生产环境
└── apps/                      # Argo CD Application 定义

## 交付流程

1. 应用代码仓库 CI 构建镜像，更新本仓库的镜像版本
2. Argo CD 监听仓库变更，自动同步到 Kubernetes 集群
3. 所有发布变更可追溯，可回滚，符合声明式交付理念

## 许可证

MIT License
