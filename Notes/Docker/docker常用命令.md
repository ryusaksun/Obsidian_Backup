## Docker 命令行接口 (CLI) 深度解析：架构、生命周期与参数详解研究报告

## 1. 执行摘要与架构背景

Docker 命令行接口（CLI）是现代云原生基础设施中与容器守护进程（Docker Daemon, `dockerd`）交互的核心机制。作为客户端-服务器（Client-Server）架构中的客户端组件，<font color="#00b0f0">Docker CLI 通过 REST API 将用户的操作指令转化为对宿主机内核的高级调用，从而实现对容器（Container）、镜像（Image）、网络（Network）及存储卷（Volume）的全生命周期管理</font>。

本报告旨在为系统架构师、DevOps 工程师及高级运维人员提供一份详尽的 Docker 命令技术分析。报告不仅涵盖常用命令的操作参数及其具体含义，更深入剖析每个命令背后的底层逻辑——包括 Linux 内核命名空间（Namespaces）、控制组（Cgroups）、联合文件系统（UnionFS）以及网络驱动的交互机制。

分析将按照功能域划分为五个核心部分：<font color="#00b0f0">镜像构建与分发体系</font>、<font color="#00b0f0">容器运行时生命周期管理</font>、<font color="#00b0f0">网络编排与隔离</font>、<font color="#00b0f0">持久化存储架构</font>、以及<font color="#00b0f0">系统可观测性与运维</font>。通过对这些维度的深度解构，本报告揭示了 Docker CLI 如何通过参数化的配置，在隔离性、性能与灵活性之间寻找最佳平衡。

## 2. Docker 架构与 CLI 交互机制

在深入具体命令之前，理解 CLI 与 Docker 引擎的交互至关重要。Docker 采用客户端-服务器架构。CLI（`docker` 命令）仅负责解析用户输入并发送 API 请求。实际的"重活"——如构建、运行、分发——由运行在宿主机的 Docker 守护进程（`dockerd`）完成。

这种解耦设计意味着 <font color="#00b0f0">CLI 可以连接本地的守护进程（通过 `/var/run/docker.sock` UNIX 套接字），也可以连接远程服务器上的守护进程</font>。<font color="#00b0f0">每一个 CLI 参数（Flag）最终都会映射为 API 请求体中的一个字段</font>。例如，`docker run` 中的 `--memory` 参数直接对应于创建容器 API 端点中 HostConfig 结构体内的 Memory 字段。

## 3. 镜像管理与构建系统 (Image Management)

镜像是容器的基石，它是只读的、多层的模板。CLI 提供了丰富的工具集用于定义、构建、查询和分发这些不可变的基础设施组件。

### 3.1 镜像构建：`docker build`

<font color="#00b0f0">`docker build` 命令用于根据 Dockerfile 中的指令和构建上下文（Build Context）创建一个新的镜像</font>。这是开发周期的起点。

#### 3.1.1 核心参数详解

- `--tag` / `-t`

    - 含义：为构建生成的镜像指定名称和标签，格式为 `name:tag`。

    - 深度解析：支持多次使用该参数，这在 CI/CD 流水线中极为常见。例如，`docker build -t myapp:1.0 -t myapp:latest .` 可以同时生成版本号标签和最新标签，确保生产环境引用的稳定性与开发环境的便捷性。如果不指定标签，Docker 默认为 `latest`，这在生产环境中通常被视为反模式，因为 `latest` 的非确定性可能导致版本漂移。

- `--file` / `-f`

    - 含义：指定 Dockerfile 的路径。

    - 操作语境：默认情况下，构建命令会在上下文根目录寻找名为 `Dockerfile` 的文件。但在多环境部署中，可能需要 `Dockerfile.dev`、`Dockerfile.prod` 或 `Dockerfile.test`。使用 `-f` 参数显式指定文件路径（如 `docker build -f deployment/Dockerfile.prod .`）可以将构建逻辑与源代码目录结构解耦。

- `--build-arg`

    - 含义：设置构建时的变量。

    - 机制：这些变量必须在 Dockerfile 中通过 `ARG` 命令预定义。它们不同于环境变量（`ENV`），仅在构建过程中有效，不会保留在最终的镜像运行时中。这对于传递版本号、代理设置或非敏感的配置标识符至关重要。

- `--no-cache`

    - 含义：构建镜像时禁用缓存。

    - 架构洞察：Docker 构建引擎利用分层缓存机制来加速构建。如果某一层指令未变，Docker 会复用缓存层。然而，对于 `RUN apt-get update` 这类指令，即使 Dockerfile 文本未变，外部源的内容可能已更新。使用 `--no-cache` 强制每一层重新执行，确保包含最新的安全补丁和依赖包。

- `--target`

    - 含义：在多阶段构建（Multi-stage Build）中，指定构建停止的目标阶段。

    - 应用场景：现代 Dockerfile 常包含构建阶段（含编译器和源码）和运行阶段（仅含二进制）。开发人员可以使用 `--target builder` 仅构建并通过调试环境，而 CI 系统则构建最终的生产镜像。这极大地优化了镜像大小和构建效率。

- `--pull`

    - 含义：始终尝试拉取最新版本的镜像。

    - 重要性：即使本地存在基镜像（如 `FROM ubuntu:20.04`），远程仓库可能已更新了该标签的指向（如修复了 CVE）。使用 `--pull` 确保构建基于最新的上游基础，防止"陈旧镜像"导致的安全隐患。

#### 3.1.2 构建上下文（Build Context）

命令末尾的路径（通常是 `.`）指定了构建上下文。Docker CLI 会将该目录下的所有文件打包发送给守护进程。如果目录下包含 `.git` 文件夹或巨大的临时文件，构建过程会出现明显的 I/O 延迟。这里隐含的最佳实践是配合 `.dockerignore` 文件使用，以剔除无关文件，这直接影响 `docker build` 的性能。

### 3.2 镜像分发与生命周期：`pull`, `push`, `rmi`

- `docker pull NAME`

    - 含义：从注册中心下载镜像。

    - 参数 `-a` / `--all-tags`：下载仓库中所有标签的镜像。慎用，因为可能导致磁盘瞬间被填满。

    - 机制：Docker 仅下载本地不存在的层（Layers）。如果多个镜像共享基础层（如都基于 Alpine），`pull` 操作会跳过已存在的层，体现了联合文件系统的存储效率。

- `docker push`

    - 含义：将本地镜像上传至注册中心。

    - 前提：必须先使用 `docker login` 进行身份验证。

- `docker rmi` (或 `docker image rm`)

    - 含义：删除本地一个或多个镜像。

    - 参数 `-f` / `--force`：强制删除。如果有一个停止的容器是基于该镜像创建的，Docker 默认会阻止删除以保护容器元数据。使用 `-f` 会强制移除镜像，导致对应的容器状态变得不稳定（无法被正常查看详情），通常只在清理脚本中使用。

    - 关联命令 `docker image prune`：

        - `docker image prune`：仅删除"悬空"（dangling）镜像，即那些没有标签且没有被容器使用的镜像层。

        - `docker image prune -a`：<font color="#00b0f0">删除所有未被现有容器（无论运行或停止）引用的镜像</font>。这是释放磁盘空间的强力工具。

### 3.3 镜像归档与迁移：`save` 与 `load` vs `export` 与 `import`

Docker 提供了两套机制用于导出镜像数据，这在物理隔离（Air-gapped）环境的数据传输中尤为关键。

#### 3.3.1 全量层保存：`save` 与 `load`

- `docker save -o <file.tar> <image>`：将镜像的所有层、元数据、标签和版本历史打包成一个 tar 归档文件。

- `docker load -i <file.tar>`：从 tar 包恢复镜像。

- 特点：完全保留镜像的构建历史和分层结构。适用于备份或在没有 Registry 的环境间完整迁移镜像。

#### 3.3.2 文件系统快照：`export` 与 `import`

- `docker export -o <file.tar> <container>`：将容器的当前文件系统状态导出为 tar 包。注意，对象是容器而非镜像。

- `docker import <file.tar> <new-image-name>`：将 tar 包导入为新的文件系统镜像。

- 区别：`export` 会丢弃所有的构建历史和层信息，导出的仅仅是文件系统的"快照"（Snapshot）。导入后生成的镜像只有一层。这对于制作极简的基础镜像或压缩镜像体积非常有用，但也意味着失去了层复用的优势。

## 4. 容器生命周期管理 (Container Lifecycle)

容器是镜像的运行时实例。对容器生命周期的精准控制是 Docker CLI 的核心功能。这包括创建、启动、停止、暂停和销毁。

### 4.1 核心指令：`docker run`

`docker run` 是 CLI 中最复杂、参数最丰富的命令。它实际上是 `create`（创建）和 `start`（启动）的原子组合，并在需要时包含 `pull`（拉取）操作。

#### 4.1.1 运行模式与身份识别

- `--detach` / `-d`

    - 含义：<font color="#00b0f0">后台运行模式</font>。

    - 深度解析：默认情况下，容器在前台运行，并将 STDOUT/STDERR 绑定到当前终端。对于 Web 服务器（如 Nginx）或数据库（如 Redis），必须使用 `-d` 使其在后台作为守护进程运行。此时 CLI 会打印完整的容器 ID 并退出，交还终端控制权。

- `--name`

    - 含义：<font color="#00b0f0">为容器指定一个人类可读的名称</font>。

    - 重要性：如果不指定，Docker 会生成随机名称（如 `serene_leavitt`）。<font color="#00b0f0">在容器编排和互联中，自定义名称充当了内部 DNS 解析的主机名（在用户定义网络中），也是后续管理命令（如 `stop`, `exec`）的便捷引用句柄</font>。

- `--rm`

    - 含义：<font color="#00b0f0">容器退出后自动删除</font>。

    - 场景：适用于一次性任务，如运行测试脚本、构建工件或数据迁移。如果不使用 `--rm`，每次运行都会遗留一个"Exited"状态的容器，迅速污染宿主机环境。

#### 4.1.2 <font color="#00b0f0">交互式会话：`-it` 组合</font>

- `--interactive` / `-i`：保持标准输入（STDIN）打开，即使没有连接。

- `--tty` / `-t`：分配一个伪终端（Pseudo-TTY）。

- 组合解析：通常联合使用 `-it`。`-t` 让容器内的 Shell 感觉像是在真实的终端中运行（支持命令提示符、Ctrl+C 等信号），而 `-i` 允许数据流从宿主机传入容器。如果只用 `-i`，你可以通过管道传入数据但看不到提示符；如果只用 `-t`，你能看到提示符但无法输入命令。运行交互式 Shell（如 `docker run -it ubuntu bash`）时，这两个参数是必须的。

#### 4.1.3 重启策略：`--restart`

Docker 提供了内置的进程守护机制，定义了容器退出时的行为。

| 策略参数                       | 行为描述                                                              | 适用场景                       |
| -------------------------- | ----------------------------------------------------------------- | -------------------------- |
| `no`                       | <font color="#00b0f0">默认值。容器退出后不自动重启</font>。                      | 开发调试、一次性批处理任务。             |
| `on-failure[:max-retries]` | 仅当容器以非零退出码退出时重启。可指定最大重试次数。                                        | 应用程序可能因临时故障崩溃，但不应无限重试。     |
| `always`                   | <font color="#00b0f0">无论退出状态如何，总是尝试重启容器。守护进程启动时也会自动启动该容器</font>。  | 生产环境的关键服务（Web Server, DB）。 |
| `unless-stopped`           | 类似 `always`，但如果容器在守护进程重启前已被用户显式停止（`docker stop`），则守护进程重启后不会自动启动它。 | 需要人工干预维护的长期服务。             |

隐式行为：<font color="#00b0f0">`always` 策略包含指数退避（Exponential Backoff）机制。如果容器在启动后立即崩溃，Docker 会逐渐增加重启间隔（100ms, 200ms, ...），以防止"重启风暴"耗尽服务器 CPU 资源</font>。

#### 4.1.4 资源限制（Cgroups 参数）

为了防止某个容器独占宿主机资源，`run` 命令通过 Linux Cgroups 暴露了资源配额参数。

- CPU 限制：

    - `--cpus=<value>`：<font color="#00b0f0">指定容器可以使用的 CPU 核心数</font>（例如 `1.5` 代表 1.5 个核）。

    - `--cpu-shares`：设置相对权重（默认 1024）。在 CPU 繁忙时，权重为 2048 的容器将获得比 1024 的容器多一倍的 CPU 时间。这是软限制，CPU 空闲时容器可以突破此限制。

    - `--cpuset-cpus`：绑定容器到特定的物理 CPU 核（如 `0,3`），用于高性能计算场景以减少上下文切换。

- 内存限制：

    - `--memory` / `-m`：设置内存硬限制，如 `512m`。如果容器进程尝试使用超过此量的内存，内核的 OOM Killer（Out of Memory Killer）将直接终止该进程。

    - `--memory-swap`：控制交换空间的使用。

    - `--oom-kill-disable`：防止容器被 OOM Killer 杀掉，但这可能导致宿主机本身因内存耗尽而死机，需慎用。

#### 4.1.5 网络与端口暴露

- `--publish` / `-p`：端口映射。

    - 格式 `hostPort:containerPort`（如 `-p 8080:80`）。

    - 机制：<font color="#00b0f0">Docker 在宿主机的 iptables 中创建 DNAT 规则，将宿主机 8080 端口的流量转发到容器 IP 的 80 端口</font>。<font color="#00b0f0">支持绑定特定接口 IP</font>（如 `-p 127.0.0.1:8080:80`），增强安全性。

    - `-P` (大写)：将 Dockerfile 中 `EXPOSE` 命令声明的所有端口随机映射到宿主机的高位端口。

- `--network`：指定容器连接的网络。

    - <font color="#00b0f0">默认为 `bridge`</font>。

    - <font color="#00b0f0">可设置为 `host`（共享宿主机网络栈，无隔离）、`none`（无网络）或自定义网络名称</font>。

- `--hostname` / `-h`：<font color="#00b0f0">设置容器内部的主机名</font>。

- `--dns`：覆盖容器内的 `/etc/resolv.conf`，指定特定的 DNS 服务器。

### 4.2 容器状态控制：`start`, `stop`, `kill`, `pause`

容器创建后，其状态流转由以下命令控制。

#### 4.2.1 优雅停止 vs 强制终止

- `docker stop CONTAINER`

    - 机制：首先向容器内 PID 1 进程发送 `SIGTERM` 信号。这<font color="#00b0f0">给应用程序一个"清理现场"的机会</font>（如完成正在处理的 HTTP 请求、关闭数据库连接、写入日志）。

    - 参数 `-t` / `--time`：默认等待 10 秒。如果进程在 10 秒内未退出，Docker 会发送 `SIGKILL` 强制终止。对于需长时间关闭的应用（如大型 Java 应用），应显式增加此时长（如 `docker stop -t 60 my-app`）。

- `docker kill CONTAINER`

    - 机制：<font color="#00b0f0">默认直接发送 `SIGKILL` 信号，进程立即终止，无任何清理机会</font>。

    - 参数 `-s` / `--signal`：可以发送自定义信号。例如，`docker kill -s HUP my-nginx` 通常用于让 Nginx 重新加载配置文件而不重启容器。

#### 4.2.2 冻结与解冻

- `docker pause CONTAINER`

    - 机制：利用 Cgroups 的 freezer 子系统。<font color="#00b0f0">它并不终止进程，而是将进程及其所有子线程从 CPU 调度队列中移除</font>。<font color="#00b0f0">进程状态保留在内存中，但完全不消耗 CPU 周期</font>。

    - 场景：对容器文件系统进行一致性快照时，或者在资源紧张时暂时挂起低优先级任务。

- `docker unpause CONTAINER`：<font color="#00b0f0">将进程恢复到调度队列</font>。

#### 4.2.3 容器创建与启动的分离

- `docker create`

    - 含义：<font color="#00b0f0">执行 `run` 命令的所有准备工作（拉取镜像、建立读写层、配置网络），但不启动进程。容器状态为 `Created`</font>。

    - 用途：允许在运行前对容器进行精细化配置或检查，是编排工具（如 Kubernetes 早期版本）常用的模式。

### 4.3 容器清理：`rm` 与 `prune`

- `docker rm CONTAINER`

    - 含义：<font color="#00b0f0">删除处于停止状态的容器</font>。

    - 参数 `-f` / `--force`：通过发送 SIGKILL 强制停止并删除正在运行的容器。

    - 参数 `-v` / `--volumes`：极其常用且常被忽略的参数。<font color="#00b0f0">删除容器时一并删除其关联的"匿名卷"（Anonymous Volumes）</font>。如果不加此参数，Docker 会在磁盘上遗留大量僵尸卷，导致磁盘空间悄无声息地耗尽。

- `docker container prune`：<font color="#00b0f0">批量删除所有停止的容器</font>。

## 5. 运维交互与可观测性 (Operations & Observability)

在容器运行期间，运维人员需要<font color="#00b0f0">深入容器内部进行调试、监控状态或提取日志</font>。CLI 提供了全套的透视工具。

### 5.1 进程注入：`docker exec`

- 命令：`docker exec CONTAINER COMMAND`

- 含义：在已运行的容器命名空间内启动一个新的进程。这是现代 Docker 运维的首选方式，取代了 SSH。

- 常用组合：`docker exec -it <container> /bin/bash`（或 `/bin/sh`）。

    - <font color="#00b0f0">这会在容器内启动一个交互式的 Shell，用于排查文件系统、网络连通性或环境变量问题</font>。

- 关键参数：

    - `-u` / `--user`：以特定用户身份执行命令（如 `-u root` 或 `-u www-data`）。这对于测试权限问题至关重要，因为默认 `exec` 可能以 root 身份运行，掩盖了真实应用的权限错误。

    - `-w` / `--workdir`：指定命令执行的工作目录。

    - `--privileged`：赋予该命令扩展的特权，使其能访问宿主机设备。

### 5.2 日志系统：`docker logs`

- 命令：`docker logs CONTAINER`

- 机制：获取容器 PID 1 进程及其子进程输出到 `STDOUT`（标准输出）和 `STDERR`（标准错误）的内容。默认情况下，这些日志被 Docker 守护进程捕获并以 JSON 格式存储在宿主机磁盘上。

- 参数详解：

    - `-f` / `--follow`：实时跟踪日志输出，功能等同于 Linux 的 `tail -f`。

    - `--tail`：仅显示最后 N 行日志（如 `--tail 100`）。对于运行已久的容器，如果不加此参数，Docker 会尝试读取整个日志文件，造成 I/O 阻塞。

    - `--since` / `--until`：按时间过滤。支持绝对时间戳（`2023-01-01T15:00:00`）或相对时间（`30m`）。这是故障排查的神器，允许运维人员精确定位事故发生时间段的日志。

    - `--timestamps`：在每行日志前添加时间戳。对于本身不打印时间的应用程序日志非常有用。

- 局限性：此命令仅对使用 `json-file` 或 `journald` 日志驱动的容器有效。如果容器配置了将日志直接发送到 Splunk 或 Syslog（通过 `--log-driver`），`docker logs` 可能无法显示任何内容。

### 5.3 状态检查：`inspect`, `top`, `stats`

#### 5.3.1 `docker inspect`：元数据真理之源

- 含义：返回容器或镜像的底层配置和运行时状态的 JSON 数据。包含 IP 地址、MAC 地址、挂载点详情、环境变量哈希等所有信息。

- 参数 `-f` / `--format`：利用 Go 模板语言提取特定字段，避免人工解析庞大的 JSON。

    - 获取 IP 地址：`docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container>`

    - 检查重启策略：`docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' <container>`

    - 提取 JSON 子集：`docker inspect -f '{{json .Config}}' <container>`

#### 5.3.2 `docker top`：进程视图

- 含义：<font color="#00b0f0">显示容器内部运行的进程列表</font>。

- 机制：<font color="#00b0f0">它实际上是查询宿主机的进程表，但过滤并仅显示属于该容器 PID 命名空间的进程</font>。

- 用途：验证容器内是否有多余的进程，或者主进程是否已产生僵尸子进程。支持传递标准 `ps` 参数，如 `docker top <container> -aux`。

#### 5.3.3 `docker stats`：资源遥测

- 含义：<font color="#00b0f0">提供实时的容器资源使用流，包括 CPU %、内存使用量/限制、网络 I/O 和磁盘 I/O</font>。

- 参数 `--no-stream`：仅获取当前时刻的快照并退出，适合脚本抓取数据。

- 参数 `--format`：自定义输出表格格式，便于生成报告。

### 5.4 文件传输：`docker cp`

- 命令：`docker cp CONTAINER:SRC_PATH DEST_PATH`（反之亦然）。

- 功能：<font color="#00b0f0">在容器与宿主机之间复制文件或目录</font>。

- 关键点：

    - 状态无关性：容器无需处于运行状态，停止的容器也可以进行拷贝。这对于从崩溃的容器中提取日志或核心转储文件（Core Dump）极为重要。

    - 参数 `-L` / `--follow-link`：如果源文件是符号链接，复制其指向的目标文件内容，而非链接本身。

    - 路径陷阱：类似于 Unix `cp`，如果在源路径末尾加 `/.`（如 `src/.`），则复制目录内的内容；否则复制目录本身。

## 6. 网络编排与隔离 (Network Orchestration)

Docker 的网络子系统（Docker Network）提供了强大的软件定义网络（SDN）能力，允许容器跨越物理主机边界进行通信，同时保持隔离。

### 6.1 `docker network create` 与驱动模型

创建一个新的网络对象。最关键的参数是 `--driver` (`-d`)，它决定了网络的拓扑结构。

| 驱动类型 | 架构与用途 |
| --- | --- |
| bridge (默认) | 单机桥接网络。在宿主机创建一个虚拟网桥（如 `docker0`）。连接到同一 bridge 的容器可以通过 IP 互通。Docker 提供自动 DNS 解析（服务发现），允许容器通过名称互相访问。适用于单机部署的微服务。 |
| host | 无隔离网络。容器移除网络命名空间隔离，直接使用宿主机的网络栈。容器没有自己的 IP，端口直接监听在宿主机上。性能最高（无 NAT 消耗），但存在端口冲突风险。 |
| overlay | 跨主机覆盖网络。主要用于 Swarm 集群。利用 VXLAN 隧道技术将多个 Docker 守护进程的网络联通，使容器感觉像是在同一局域网内。 |
| macvlan | 物理层透传。为每个容器分配唯一的 MAC 地址，使其在物理网络交换机看来像是一台独立的物理设备。用于旧应用改造或需要极高性能且不经过 NAT 的场景。 |
| none | 完全隔离。容器有自己的网络栈但没有外部接口（只有 lo）。用于安全性要求极高、不需要联网的计算任务。 |

#### 6.1.1 关键创建参数

- `--subnet`：显式指定子网网段（如 `172.20.0.0/16`）。这在企业网络中必须使用，以防止 Docker 默认分配的网段与公司内部物理网络冲突。

- `--gateway`：指定网关 IP。

- `--internal`：创建一个封闭网络，禁止容器访问外部互联网，仅允许容器间通信。

### 6.2 动态连接：`connect` 与 `disconnect`

Docker 允许在容器运行时动态调整其网络连接，这是相对于虚拟机的巨大优势。

- `docker network connect NETWORK CONTAINER`

    - 功能：将已运行的容器"插入"到另一个网络中。容器内部会增加一个新的网络接口（如 `eth1`）。

    - 参数 `--ip`：请求分配固定的静态 IP 地址（如 `--ip 10.0.0.5`）。注意：静态 IP 只能在用户自定义网络中使用，默认的 `bridge` 网络不支持。

- `docker network disconnect`：将容器从网络中断开。

### 6.3 网络诊断：`inspect`

`docker network inspect <network>` 输出包含该网络所有配置及当前连接的所有容器及其 IP 地址。这是排查"容器 A 连不上容器 B"时的第一步。

## 7. 持久化存储管理 (Storage & Persistence)

容器的文件系统是临时的。为了保存数据库文件、日志或配置文件，必须使用存储挂载。CLI 区分了 Volumes（卷）和 Bind Mounts（绑定挂载）。

### 7.1 `docker volume` 命令族

<font color="#00b0f0">卷是 Docker 管理的、独立于容器生命周期的存储实体</font>，通常位于 `/var/lib/docker/volumes`。

- `docker volume create[VOLUME]`

    - 参数 `--driver` / `-d`：默认为 `local`。通过插件可以使用第三方驱动将数据存入 AWS EBS、NFS 或 GlusterFS。

    - NFS 挂载示例：可以通过 `docker volume create` 直接创建一个连接到远程 NFS 服务器的卷，容器挂载该卷时无需安装 NFS 客户端。

    - 参数 `--label`：添加元数据以便管理。

- `docker volume ls`：列出所有卷。

    - 参数 `-f dangling=true`：筛选出未被任何容器引用的"僵尸卷"。

- `docker volume prune`：一键删除所有未使用的卷。危险操作，需确认数据确实不再需要。

### 7.2 挂载参数：`-v` vs `--mount`

在 `docker run` 中挂载存储有两种语法：

1. `-v` / `--volume`（旧语法）

    - 格式：`source:target:options`（如 `-v /data/db:/var/lib/mysql:rw`）。

    - 隐患：如果宿主机上的 `source` 路径不存在，`-v` 会自动创建一个空目录并挂载。这常导致权限问题（创建的目录归属 root），导致容器启动失败。

2. `--mount`（新语法，推荐）

    - 格式：键值对结构，逗号分隔。

    - 示例：`--mount type=bind,source=/data/db,target=/var/lib/mysql,readonly`

    - 优势：

        - 明确性：必须指定 `type`（`volume`, `bind`, `tmpfs`）。

        - 安全性：对于 `bind` 类型，如果 `source` 不存在，Docker 会报错而不是自动创建空目录，从而避免了意外的配置错误。

        - 功能：支持更复杂的配置，如 tmpfs 的大小限制、卷驱动的特殊选项。

### 7.3 挂载类型对比

| 类型 | 描述 | 最佳实践场景 |
| --- | --- | --- |
| Volume | 存储在 Docker 管理的区域。 | 数据库数据、跨容器共享数据、不依赖特定宿主机路径的应用。 |
| Bind Mount | 映射宿主机的任意文件或目录。 | 开发环境代码挂载（代码热重载）、挂载宿主机配置文件（如 `/etc/localtime`）。 |
| tmpfs | 仅存储在宿主机内存中，不写入磁盘。 | 存储敏感数据（密钥）或追求极高性能的临时缓存。 |

## 8. 系统维护与环境清理 (System Maintenance)

随着时间推移，Docker 宿主机会积累大量未使用的镜像层、停止的容器和废弃的网络，占用大量磁盘空间。

- `docker system prune`

    - 默认行为：删除所有停止的容器、所有未被容器使用的网络、所有悬空（无标签）的镜像。

    - 参数 `--volumes`：关键。默认情况下，为了防止数据丢失，prune 不删除卷。必须显式加上此参数才能清理未使用的匿名卷。

    - 参数 `-a` / `--all`：删除所有未使用的镜像（不仅仅是悬空的）。这意味着不仅清理构建失败的中间产物，还会清理掉哪怕是完好但当前没有容器在跑的镜像（如旧版本的应用镜像）。

- `docker system df`

    - 显示 Docker 对象（镜像、容器、卷）占用的磁盘空间概览以及可回收的空间比例。这是决定是否需要执行 prune 的依据。

## 9. 结论

Docker CLI 不仅仅是一个简单的运行工具，它是一套精密的基础设施管理语言。从 `run` 命令中对 Cgroups 资源的精细调度，到 `build` 命令中对联合文件系统层的巧妙复用，再到 `network` 和 `volume` 命令对隔离性与持久性的抽象，每一个参数背后都蕴含着对 Linux 内核特性的封装。

对于专业技术人员而言，熟练掌握这些命令的参数细节（如 `--mount` 优于 `-v`，`exec` 优于 SSH，`--init` 对信号处理的重要性）是构建稳定、安全且高效的容器化环境的先决条件。通过合理利用重启策略、健康检查和资源限制，可以将单机 Docker 环境的可靠性提升至生产级标准。

---

**<font color="#2ecc71">✅ 已格式化</font>**
