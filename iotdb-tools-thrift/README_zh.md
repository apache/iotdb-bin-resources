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

Linux profile 会用 `-static` 链接 `thrift` 可执行文件，因此构建机器需要具备
当前 C/C++ 工具链静态链接所需的运行时库。

Windows 上请使用 `mvnw.cmd` 替代 `./mvnw`。

## 本地构建

在当前目录执行：

    ./mvnw clean package -DskipTests

构建产物会生成到 `target/` 目录下。请确认压缩包中包含 `bin/thrift`，并确认该
二进制文件输出的 Apache Thrift 版本符合预期。

## 使用 GitHub Actions 构建发布 artifacts

`Build IoTDB Tools Thrift Artifacts` workflow 会构建六个平台的 zip artifacts，
但不会签名，也不会 deploy 到 Nexus。请在 GitHub Actions 页面手动触发该
workflow，可以通过 `git_ref` 输入指定要构建的分支、tag 或 commit SHA。
workflow 会验证每个平台生成的 compiler 是否输出预期的 Apache Thrift 版本，
并确认 Linux compiler 是静态链接的。

workflow 会上传一个名为 `iotdb-tools-thrift-all-platforms` 的汇总 artifact。
下载并解压到当前模块的以下目录：

    iotdb-tools-thrift/prebuilt-artifacts/

如果使用 GitHub CLI，可以执行：

    gh run download <run-id> --name iotdb-tools-thrift-all-platforms --dir prebuilt-artifacts

该目录中必须包含以下文件：

- `iotdb-tools-thrift-${project.version}-linux-x86_64.zip`
- `iotdb-tools-thrift-${project.version}-linux-aarch64.zip`
- `iotdb-tools-thrift-${project.version}-mac-x86_64.zip`
- `iotdb-tools-thrift-${project.version}-mac-aarch64.zip`
- `iotdb-tools-thrift-${project.version}-windows-x86_64.zip`
- `iotdb-tools-thrift-${project.version}-windows-aarch64.zip`

`${project.version}` 是 Maven 项目版本，例如 `0.23.0.0`。

deploy 前请先检查这些压缩包。每个压缩包应当只包含 `bin/thrift` 可执行文件，
Windows 平台为 `bin/thrift.exe`，并且该可执行文件输出的 Apache Thrift 版本
符合预期。

## 将预构建 artifacts 发布到 Nexus

在当前目录本地执行 deploy。该命令会对 `prebuilt-artifacts/` 下的六个平台
artifacts 签名并 deploy：

    ./mvnw clean deploy -P apache-release,prebuilt-artifacts

如果下载目录不是默认的 `prebuilt-artifacts/`，可以通过
`prebuilt.artifacts.dir` 指定：

    ./mvnw clean deploy -P apache-release,prebuilt-artifacts -Dprebuilt.artifacts.dir=/path/to/prebuilt-artifacts

这一步会在 Nexus 中创建新的 staging repository。deploy 完成后，打开
https://repository.apache.org/#stagingRepositories，检查上传的 artifacts。

如果需要重新 deploy 到已有 staging repository，请传入真实存在的 staging
repository id：

    ./mvnw clean deploy -P apache-release,prebuilt-artifacts -DstagingRepositoryId=orgapacheiotdb-1234

`stagingRepositoryId` 必须是 Nexus 里真实存在的 staging repository id，不能
随便填写一个不存在的 id。

## 在每个平台本地构建并发布

如果不使用预构建 artifacts workflow，也可以继续按平台分别构建并 deploy。

先在一个平台上执行第一次 deploy，不要传入 `stagingRepositoryId`：

    ./mvnw clean deploy -P apache-release

这一步会在 Nexus 中创建新的 staging repository。deploy 完成后，打开
https://repository.apache.org/#stagingRepositories，复制新创建出来的 staging
repository id，例如 `orgapacheiotdb-1234`。

然后在其余平台上执行 deploy，并传入第一次创建出来的 staging repository id：

    ./mvnw clean deploy -P apache-release -DstagingRepositoryId=orgapacheiotdb-1234

支持的平台 classifier 包括：

- `linux-x86_64`
- `linux-aarch64`
- `mac-x86_64`
- `mac-aarch64`
- `windows-x86_64`
- `windows-aarch64`

所有平台压缩包都 deploy 完成后，在 Nexus 中检查 staging repository，确认无误后
close，然后继续 Apache release vote 和 release 流程。
