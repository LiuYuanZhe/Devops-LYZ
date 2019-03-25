介绍：spinnaker是netflix开源的一款可以对接云服务（自由kubernetes，aws等）的持续交付平台，通过pipeline可以将docker仓库中的镜像持续发布到云集群中，在云平台项目中，主要用于打通docker repository与kubernetes集群进行容器的发布。

spinnaker可以创建loadbalance等service资源对运行的pod进行服务的暴露，创建副本集来控制pod进行容器的部署，在容器构建的时候通过配置去控制pod的属性（副本数，configmap等），还能构建pipeline通过对资源库的监控（比如对资源库镜像的更新监控）实现自动部署。

在容器云平台中spinnaker负责continue deploy部分，当jenkins流水线触发后对版本库中的代码进行拉取，构建镜像，推送到镜像仓库后，spinnaker的pipeline触发对镜像进行下载，并按照设置好的配置进行在kubernetes集群中的部署。

对spinnaker的配置见 **《DevOps体系建设以及相关问题.docx》**



- **Deck**：面向用户 UI 界面组件，提供直观简介的操作界面，可视化操作发布部署流程。

- **API**： 面向调用 API 组件，我们可以不使用提供的 UI，直接调用 API 操作，由它后台帮我们执行发布等任务。
- **Gate**：是 API 的网关组件，可以理解为代理，所有请求由其代理转发。
- **Rosco**：是构建 beta 镜像的组件，需要配置 Packer 组件使用。
- **Orca**：是核心流程引擎组件，用来管理流程。
- **Igor**：是用来集成其他 CI 系统组件，如 Jenkins 等一个组件。
- **Echo**：是通知系统组件，发送邮件等信息。
- **Front50**：是存储管理组件，需要配置 Redis、Cassandra 等组件使用。
- **Cloud driver** 是它用来适配不同的云平台的组件，比如 Kubernetes，Google、AWS EC2、Microsoft Azure 等。
- **Fiat** 是鉴权的组件，配置权限管理，支持 OAuth、SAML、LDAP、GitHub teams、Azure groups、 Google Groups 等。

