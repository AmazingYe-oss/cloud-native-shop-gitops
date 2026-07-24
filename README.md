# Cloud Native Shop — GitOps 配置仓库

ArgoCD GitOps 声明式交付配置，包含 Helm Chart、环境覆盖值、ArgoCD Application 和告警规则。

## 项目结构

```
├── charts/cloud-native-shop/       # Helm Chart
│   ├── Chart.yaml                  # Chart 元数据
│   ├── values.yaml                 # 默认值（镜像、端口、副本数）
│   └── templates/
│       ├── deployment.yaml         # Deployment 模板（遍历 services）
│       └── service.yaml            # Service 模板（遍历 services）
├── environments/
│   └── dev/
│       └── values.yaml             # dev 环境覆盖值（tag 由 CI 自动更新）
├── argocd/
│   ├── application.yaml            # ArgoCD Application 定义
│   └── alert-rules.yaml            # Prometheus 告警规则
└── README.md
```

## GitOps 工作流

```
代码 push → CI 构建镜像 → CI 更新 environments/dev/values.yaml 的 tag
                                        ↓
                          ArgoCD 检测 git 变更（轮询间隔 3 分钟）
                                        ↓
                          自动同步 Helm Chart 到 K8s 集群
```

## ArgoCD Application

- **仓库**：本仓库 main 分支
- **Chart 路径**：`charts/cloud-native-shop`
- **目标集群**：本集群（k3d）
- **目标 Namespace**：`cloud-native-shop`
- **同步策略**：自动同步 + prune + selfHeal

## 告警规则

| 规则 | 条件 | 严重程度 |
|------|------|----------|
| PodRestartingTooOften | 10 分钟内重启 > 3 次 | warning |
| PodNotReady | Not Ready 超过 2 分钟 | critical |
| ContainerOOMKilled | 容器 OOM 被杀 | critical |

## 关键设计决策

- **Namespace 由 ArgoCD 控制**：模板使用 `$.Release.Namespace`，不在 values.yaml 中硬编码
- **镜像地址拼接**：`registry` + `image` + `tag` 三段式，registry 在 values.yaml 中统一配置
- **CI 更新 tag**：GitHub Actions 的 `update-gitops` 阶段用 yq 修改 `environments/dev/values.yaml`
