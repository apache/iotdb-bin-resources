<!--

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.

-->

# 发布 IoTDB Tools: Thrift

本模块用于发布 IoTDB 构建过程中使用的、按平台区分的 Apache Thrift
compiler 压缩包。压缩包中只包含 `thrift` 可执行文件。

## 环境准备

构建本模块前需要安装以下软件：

- JDK
- Flex
- Bison
- CMake 支持的 C/C++ 构建工具链
- 已配置好 Apache 发布签名的 GPG
- 在 Maven `settings.xml` 中配置好 server id 为 `apache.releases.https` 的 Apache Nexus 凭据

Linux 静态构建可能还需要安装 zlib 和 OpenSSL 的静态开发包。

Windows 上请使用 `mvnw.cmd` 替代 `./mvnw`。

## 本地构建

在当前目录执行：

    ./mvnw clean package -DskipTests

构建产物会生成到 `target/` 目录下。请确认压缩包中包含 `bin/thrift`，并确认该
二进制文件输出的 Apache Thrift 版本符合预期。

## 发布到 Nexus

先在一个平台上执行第一次 deploy，不要传入 `stagingRepositoryId`：

    ./mvnw clean deploy -P apache-release

这一步会在 Nexus 中创建新的 staging repository。deploy 完成后，打开
https://repository.apache.org/#stagingRepositories，复制新创建出来的 staging
repository id，例如 `orgapacheiotdb-1234`。

然后在其余平台上执行 deploy，并传入第一次创建出来的 staging repository id：

    ./mvnw clean deploy -P apache-release -DstagingRepositoryId=orgapacheiotdb-1234

`stagingRepositoryId` 必须是第一次 deploy 后 Nexus 里真实存在的 staging
repository id，不能随便填写一个不存在的 id。

支持的平台 classifier 由当前操作系统激活的 Maven profile 决定：

- `linux-x86_64`
- `linux-aarch64`
- `mac-x86_64`
- `mac-aarch64`
- `windows-x86_64`
- `windows-aarch64`

所有平台压缩包都 deploy 完成后，在 Nexus 中检查 staging repository，确认无误后
close，然后继续 Apache release vote 和 release 流程。
