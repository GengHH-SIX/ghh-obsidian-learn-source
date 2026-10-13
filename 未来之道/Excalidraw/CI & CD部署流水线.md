# 一、个人-中小型项目（docker-compose）

针对 Vite + Vue3 前端和 GitHub Actions 的技术栈，我们要实现 CI/CD，核心逻辑是：

1. 本地/CI 阶段：使用 Vite 打包出静态文件（`dist`），并用 Nginx 镜像将其包裹，构建成一个前端 Docker 镜像并推送到仓库。
2. GitHub Actions 阶段：编写自动化脚本，自动完成打包、登录 Docker Hub/阿里云、推流以及 SSH 连上服务器执行重启。

以下是完整的实战配置文件和指令写法：

---

## 📦 1. 前端 Vue3 的 `Dockerfile` 配置

在前端项目根目录下创建 `Dockerfile`。推荐使用多阶段构建，这样能大幅缩小最终生产镜像的体积（只有十几 MB）：

```dockerfile
# --- Stage 1: 构建阶段 ---
FROM node:20-alpine AS builder
WORKDIR /app

# 复制依赖文件并安装（利用 Docker 缓存机制）
COPY package*.json ./
RUN npm ci

# 复制所有源码并执行打包
COPY . .
RUN npm run build

# --- Stage 2: 运行阶段 ---
FROM nginx:alpine
# 将打包后的静态文件复制到 Nginx 默认静态资源目录
COPY --from=builder /app/dist /usr/share/nginx/html

# 如果需要支持 Vue Router 的 HTML5 History 模式，可以复制自定义的 nginx.conf（可选）
# COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---


### 💡 `npm ci` 与 `npm install` 的区别

 > `RUN npm ci` 运行的不是 `package.json` 中的 `scripts` 脚本，它是一个类似于 `npm install` 的 Node.js 官方内置命令。

> 它的==核心作用==是：专门为了在 CI/CD 流水线（或 Docker 构建）中，快速、干净、严格地安装依赖。


|特性|`npm install` (本地开发用)|`npm ci` (CI/CD / Docker 用)|
|---|---|---|
|依赖来源|依据 `package.json` 寻找匹配版本，可能会升级小版本|严格依据 `package-lock.json`，锁死每一个依赖的版本|
|速度|较慢（会检查版本并增量更新）|非常快（直接删掉原有的 `node_modules` 并一键解压安装）|
|严格度|如果 lock 文件和 json 文件不一致，会自动修改 lock 文件|如果两者不一致，直接报错中断构建，确保环境绝对安全稳定|

---

### 🛠️ 真正运行 `package.json` 脚本的是哪一行？

在你的 `Dockerfile` 中，真正执行 `package.json` 里面自定义脚本的是接下来的这一行：

```dockerfile
RUN npm run build
```

这一行指令，才会去调用你在 `package.json` 的 `"scripts"` 中定义的构建命令。

---

### 📝 一个典型的 Vite + Vue3 `package.json` 脚本长这样

为了让 `RUN npm run build` 正常工作，你的前端 `package.json` 文件中通常会包含以下配置：

```json
{
  "name": "my-vue3-vite-project",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",               // 本地开发启动命令：npm run dev
    "build": "vite build",       // 👈 Docker 内运行的就是这一行！用来打包项目
    "preview": "vite preview"    // 本地预览打包后的效果
  },
  "dependencies": {
    "vue": "^3.4.0",
    "vue-router": "^4.3.0",
    "pinia": "^2.1.0"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^5.0.0",
    "vite": "^5.2.0"
  }
}
```

### 🔍 Docker 执行 `RUN npm run build` 时，背后发生了什么？

1. Docker 容器内的 Node.js 引擎会执行 `vite build`。
2. Vite 会编译、压缩你的 Vue3 源码（包含 `.vue`, `.ts`, `.js` 等文件）。
3. 构建完成后，会在前端项目的根目录下生成一个名为 `dist/` 的文件夹。
4. 这个 `dist/` 文件夹里全是纯静态的 `index.html`、`assets/js/xxx.js` 和 `assets/css/xxx.css`。
5. 在 `Dockerfile` 的第二阶段，Nginx 会把这个 `dist/` 文件夹接管过去，直接面向用户提供网页访问服务。

你的 Vite + Vue3 项目在打包时有区分布署环境吗？例如：

- 是否有需要区分测试环境（Development/Staging）和生产环境（Production）的后端 API 请求地址？
- 本地项目根目录下是否有类似 `.env.development` 或 `.env.production` 的环境变量文件？

如果需要动态切换 API 地址，我可以在 Docker 编译时帮你加上对应的变量注入指令。

## 🚀 2. GitHub Actions 完整的流水线指令（`.github/workflows/deploy.yml`）

在项目根目录下创建 `.github/workflows/deploy.yml` 文件。这个脚本会在你把代码 `git push` 到 `main` 分支时自动触发：

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main # 监听 main 分支的 push 事件

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # 1. 拉取仓库代码
      - name: Checkout code
        uses: actions/checkout@v4

      # 2. 登录到 Docker Hub（如果是阿里云，请更换为 aliyun 的 action 或修改 login-server）
      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # 3. 构建前端镜像并推送到镜像仓库
      - name: Build and Push Frontend Image
        uses: docker/build-push-action@v5
        with:
          context: ./frontend # 指向你前端代码的目录
          file: ./frontend/Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/vue3-frontend:latest

      # 4. 如果有后端，可以在这里并行或串行构建后端镜像（结构同上，换个 tag 即可）

      # 5. 远程部署：SSH 连上你的服务器，拉取新镜像并重启容器
      - name: Deploy to Server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }} # 推荐使用 SSH 密钥，更安全
          script: |
            # 进入你服务器上存放 docker-compose.yml 的目录
            cd /www/wwwroot/your-project
            
            # 登录 Docker（确保服务器有权限拉取私有镜像，公有镜像可忽略此步）
            echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin
            
            # 拉取最新的镜像
            docker compose pull
            
            # 启动并重启发生变更的容器
            docker compose up -d --remove-orphans
            
            # 清理无用的旧镜像（释放服务器磁盘空间）
            docker image prune -f
```

---

## 🛡️ 3. 必须在 GitHub 仓库中配置的机密变量 (Secrets)

为了不把密码和密钥暴露在代码中，你需要在你的 GitHub 仓库页面，点击 Settings -> Secrets and variables -> Actions -> New repository secret，依次添加以下 5 个变量：

- `DOCKERHUB_USERNAME`：你的 Docker Hub 账号名。
- `DOCKERHUB_TOKEN`：你的 Docker Hub 个人访问令牌（Personal Access Token，不建议直接用登录密码）。
- `SERVER_HOST`：你的服务器公网 IP 地址。
- `SERVER_USER`：服务器登录用户名（如 `root`）。
- `SERVER_SSH_KEY`：你服务器的私钥（即 `~/.ssh/id_rsa` 的内容，用来免密登录）。

---

## 🐳 4. 服务器上的 `docker-compose.yml` 示例

在你的远程服务器（如 `/www/wwwroot/your-project` 路径下），放置一个专门供生产环境运行的 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  frontend:
    image: your_dockerhub_username/vue3-frontend:latest # 对应自动化构建出来的镜像地址
    container_name: vue3-frontend-prod
    ports:
      - "80:80" # 把服务器的 80 端口映射到容器内 Nginx 的 80 端口
    restart: always
    depends_on:
      - backend

  backend:
    image: your_dockerhub_username/js-backend:latest
    container_name: js-backend-prod
    ports:
      - "3000:3000"
    restart: always
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:password@db:5411/mydb

  db:
    image: postgres:15-alpine
    container_name: postgres-prod
    volumes:
      - pgdata:/var/lib/postgresql/data # 数据持久化，防止重启丢失数据
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb

volumes:
  pgdata:
```

配置过程中最容易卡在网络和权限上，请问：

1. 你的后端技术栈是什么（如 Express、Koa、NestJS 或 Next.js 开启了 SSR）？
2. 你的前端生产环境需要支持 HTTPS 证书 或者由外部的 Nginx 做网关反向代理吗？


---

# 二、大型项目（docker & k8s集群）

你的理解非常敏锐，切中了现代云原生架构演进的核心。

结论是：是的，在现代大型 Kubernetes (K8s) 项目中，GitLab CI 和 Jenkins 越来越倾向于“只做 CI（构建/测试）”，而把 CD（部署）的能力交棒给像 ArgoCD 或 Flux 这样的专门 GitOps 工具。

不过，这并不是绝对的，我们可以把企业的大型 K8s 项目落地演进分为`“传统模式”`和`“现代 GitOps 模式”`两种来看：

---

## 🧱 模式一：传统模式（GitLab CI / Jenkins “一管到底”）

在没有 ArgoCD 之前，或者在很多企业还没完全转到 GitOps 架构时，GitLab CI 和 Jenkins 是既做 CI 又做 CD 的。

- 流程：
    
    1. Jenkins/GitLab CI 编译代码、跑测试、打包 Docker 镜像。
    2. 镜像推送到私有仓库。
    3. CD 阶段：流水线通过安装 `kubectl` 或 `helm` 插件，持有 K8s 集群的高权限认证（比如 `kubeconfig` 凭证）。
    4. 流水线直接向 K8s 集群发送指令：`kubectl set image deployment/vue3-frontend ...` 或 `helm upgrade ...`。
    

## ❌ 传统模式在大型 K8s 项目中的致命痛点：

1. 安全隐患（凭证外泄）：Jenkins 或 GitLab CI 需要掌握你生产环境 K8s 集群的最高控制权（Admin 权限）。一旦 CI 工具被黑客攻破，整个生产集群就面临瘫痪风险。
2. 状态漂移（State Drift）：CI 工具是“单向推（Push）”的。如果有人偷偷用命令行修改了集群里的配置，或者集群因为故障自动重启了，Jenkins 是不知道的。它只负责触发的那一刻，无法保证集群状态和 Git 代码永远一致。
3. 回滚极其痛苦：如果新版本上线后崩溃了，你需要重新去 Jenkins 找到上一个健康的流水线点击“Replay”或重新触发，耗时较长。

---

## 🚀 模式二：现代模式（Jenkins/GitLab 做 CI + ArgoCD 做 CD）

为了解决上述痛点，大型项目开始流行 GitOps 模式（即我上一条回答提到的模式）。在这种模式下，分工变得极其明确：

```unset
[ 研发代码库 ] ────> 触发 ────> 【 GitLab CI / Jenkins 】 (只负责 CI)
                                        │
                                        ▼ (构建镜像、修改配置清单)
[ 部署清单库 (Git) ] <───────────────────┘
       ▲
       │ (时刻监听、拉取对比)
【 ArgoCD 】 (只负责 CD，运行在 K8s 内部) ───> 自动同步 ───> [ 生产 K8s 集群 ]
```

## ⚖️ 它们如何完美配合？

- GitLab CI / Jenkins 的唯一任务（CI）：  
    它们只负责和源码打交道。拉取 Vue3/Node.js 代码、跑单元测试、打包成 `v1.2.3` 镜像推送到 Harbor。最后，它们唯一的“CD动作”不是去调 K8s，而是把存放配置的 Git 仓库里的版本号从 `v1.2.2` 改成 `v1.2.3`。
- ArgoCD 的唯一任务（CD）：  
    它作为一个特务（Agent）常驻在 K8s 集群内部。它不需要知道代码是怎么写的，也不需要知道镜像怎么打的。它只盯着配置仓库。只要发现 Git 里的配置变了，它就在 K8s 内部以“拉（Pull）”的方式完成滚动更新。

## ⭕ 现代模式的核心优势：

- 无敌的安全性：Jenkins 和 GitLab 再也不需要 K8s 集群的任何密码或证书。它们只需要对 Git 仓库的写入权限即可。
- 自动修复防篡改：如果有运维人员擅自在 K8s 里改了配置，ArgoCD 会立刻发现“集群现状”与“Git 预期”不符，瞬间报警并自动把集群强行恢复成 Git 里写的样子。
- 秒级回滚：线上出 Bug 了？不需要动 Jenkins。直接在 Git 仓库把提交回滚（Git Revert），或者在 ArgoCD 界面上点一下 `Rollback`，2秒钟内集群自动退回上一个稳定版。

---

## 📝 总结大厂的选型现状

并不是 Jenkins 和 GitLab 搞不定 CD，而是因为 K8s 的声明式架构天然和 GitOps (ArgoCD) 是绝配。

在大型项目中，架构师通常这样安排：

1. GitLab CI / GitHub Actions：因为深度集成在代码托管平台里，开发者体验最好，用来做 CI 极为顺滑。
2. Jenkins：由于其强大的插件生态和老牌的调度能力，在大型传统混合架构（既有 K8s，又有旧虚拟机/物理机）中，依然作为 CI 统一调度器。
3. ArgoCD / Flux：专职作为 K8s 环境的 CD 守护神。

了解了这一套大厂的分布式职责划分后，如果回归到你目前的实际场景：

- 你是希望先用传统模式（通过 GitHub Actions 一管到底，直接用 SSH/kubectl 更新 K8s）作为过渡？
- 还是打算直接一步到位，在本地用 Minikube/Kind 搭建 K8s 并把 ArgoCD 跑起来体验一整套 GitOps 流程？



---


# 三、两种方式区别对比

[[项目部署运维#docker & k8s集群]]