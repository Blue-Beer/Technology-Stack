# Cloud-Native 12 Factor

12-Factor 为构建如下的 SaaS 应用提供了方法论：

- 使用标准化流程自动配置，从而使新的开发者花费最少的学习成本加入这个项目。
- 和操作系统之间尽可能的划清界限，在各个系统中提供最大的可移植性。
- 适合部署在现代的云计算平台，从而在服务器和系统管理方面节省资源。
- 将开发环境和生产环境的差异降至最低，并使用持续交付实施敏捷开发。
- 可以在工具、架构和开发流程不发生明显变化的前提下实现扩展。

这套理论适用于任意语言和后端服务（数据库、消息队列、缓存等）开发的应用程序。

## Codebase

使用一份基准代码来进行多份部署。

每个应用只对应一份基准代码，但可以同时存在多份部署。每份 **部署** 相当于运行了一个应用的实例。通常会有一个生产环境，一个或多个预发布环境。此外，每个开发人员都会在自己本地环境运行一个应用实例，这些都相当于一份部署。

## Dependencies

显示声明依赖关系。

12-Factor规则下的应用程序不会隐式依赖系统级的类库。 它一定通过 **依赖清单** ，确切地声明所有依赖项。此外，在运行过程中通过 **依赖隔离** 工具来确保程序不会调用系统中存在但清单中未声明的依赖项。这一做法会统一应用到生产环境。
显式声明依赖的优点之一是为新进开发者简化了环境配置流程。新进开发者可以检出应用程序的基准代码，安装编程语言环境和它对应的依赖管理工具，只需通过一个 **构建命令** 来安装所有的依赖项，即可开始工作。
12-Factor 应用同样不会隐式依赖某些系统工具，即使这些工具存在于几乎所有系统，但终究无法保证所有未来的系统都能支持应用顺利运行，或是能够和应用兼容。

## Config

在环境中存储配置。

配置信息存储在环境中才能保证一套版本的代码可以依靠不同的配置信息顺利运行在多个环境中。

## Backing services

把后端服务当作附加资源。

这里的后端服务指的是程序在运行时需要通过网络调用的各种服务，如数据库，消息、队列系统等。12-Factor应用不会区别对待本地或第三方服务。对应用程序而言，两种都是附加资源，通过一个url或是其他存储在配置中的服务定位/服务证书来获取数据。12-Factor应用的任意部署，都应该可以在不进行任何代码改动的情况下，将本地MySQL数据库换成第三方服务（例如 Amazon RDS），“松耦合”。

## Build,release,run

严格分离构建和运行。

基础代码转化为一份部署需要三个阶段：

- 构建阶段 将仓库中的代码转化为可执行包。构建时会使用指定版本的代码，获取和打包依赖项，编译成二进制文件和资源文件。
- 发布阶段会将构建的结果和当前部署所需配置相结合，并能够立即在运行环境中投入使用。
- 运行阶段指针对选定的发布版本，在执行环境中启动一系列应用程序的进程。

## Precess

12-Factor应用的进程必须是无状态且无共享的。任何需要持久化的数据都要存储在数据库中。

- 无状态：客户端的每次请求必须具备自描述信息，通过这些信息识别客户端身份。服务端不保存任何客户端请求者信息。
- 无共享：每个请求都有服务集群中的唯一一个节点来完成。

## Port binding

通过端口绑定来提供服务。

12-Factor 应用完全自我加载而不依赖任何网路服务器就可以创建一个面向网络的服务。在线上环境中，请求统一发送到公共域名然后通过路由进行转发。

## Concurrency

添加并发可以水平扩展。

## Disposability

快速启动和优雅优雅终止可最大化健壮性。

- 进程应当追求 **最小启动时间** 。理想状态下，进程从敲下命令到真正启动并等待请求的时间应该只需很短的时间。更少的启动时间提供了更敏捷的**发布**以及扩展过程，此外还增加了健壮性，因为进程管理器可以在授权情形下容易的将进程搬到新的物理机器上。
- 进程 **一旦接收终止信号（SIGTERM）** 就会优雅的终止 。就网络进程而言，优雅终止是指停止监听服务的端口，即拒绝所有新的请求，并继续执行当前已接收的请求，然后退出。此类型的进程所隐含的要求是HTTP请求大多都很短(不会超过几秒钟)，而在长时间轮询中，客户端在丢失连接后应该马上尝试重连。
- 进程还应当在**面对突然死亡时保持健壮**，例如底层硬件故障。虽然这种情况比起优雅终止来说少之又少，但终究有可能发生。一种推荐的方式是使用一个健壮的后端队列，例如 Beanstalkd ，它可以在客户端断开或超时后自动退回任务。无论如何，12-Factor 应用都应该可以设计能够应对意外的、不优雅的终结。

## Dev/prod parity

尽可能的保持开发，预发布，线上环境相同。

12-Factor应用的开发人员应该反对在不同环境间使用不同的后端服务来减少因为配置的不兼容导致的部署阻塞。

## Logs

把日志当作事件流。

每一个运行的进程都会直接的标准输出（stdout）事件流。输出流可以发送到 Splunk 这样的日志索引及分析系统，或 Hadoop/Hive 这样的通用数据存储系统。这些系统为查看应用的历史活动提供了强大而灵活的功能。

## Admin processes

后台管理任务当作一次性进程运行。

开发人员经常希望执行一些管理或维护应用的一次性任务。使用shell脚本来执行这些管理任务。