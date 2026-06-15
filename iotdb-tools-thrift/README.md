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

# Releasing the IoTDB Tools: Thrift

This module publishes platform-specific Apache Thrift compiler archives used by
IoTDB builds. The archives contain the `thrift` executable only.

## Prerequisites

Install the following software before building this module:

- JDK
- Flex
- Bison
- A C/C++ build toolchain supported by CMake
- GPG configured for Apache release signing
- Apache Nexus credentials configured as `apache.releases.https` in Maven `settings.xml`

Linux profiles build the `thrift` executable with `-static`, so the build host
needs the static runtime libraries required by its C/C++ toolchain.

Use `mvnw.cmd` instead of `./mvnw` on Windows.

## Build Locally

Run the following command from this directory:

    ./mvnw clean package -DskipTests

The archive is generated under `target/`. Check that it contains `bin/thrift`
and that the binary reports the expected Apache Thrift version.

## Deploy to Nexus

Run the first deploy on one platform without `stagingRepositoryId`:

    ./mvnw clean deploy -P apache-release

This creates a new staging repository in Nexus. After the deploy completes, open
https://repository.apache.org/#stagingRepositories and copy the generated staging
repository id, for example `orgapacheiotdb-1234`.

Run the deploy on each remaining platform with that exact staging repository id:

    ./mvnw clean deploy -P apache-release -DstagingRepositoryId=orgapacheiotdb-1234

The `stagingRepositoryId` value must be an existing Nexus staging repository id
created by the first deploy. Do not use a made-up id.

Supported classifiers are selected by the active OS profile:

- `linux-x86_64`
- `linux-aarch64`
- `mac-x86_64`
- `mac-aarch64`
- `windows-x86_64`
- `windows-aarch64`

After all platform archives have been deployed, verify the staging repository in
Nexus, close it, and continue with the Apache release vote and release process.
