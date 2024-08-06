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

    ./mvnw clean deploy -P apache-release

On the first run, comment out the "stagingRepositoryId" property. 
This will make the build deploy the rc in a new staging repository in nexus.

As soon as the build is finished, go to https://repository.apache.org/#stagingRepositories and take the new repository id from there and comment in the property again and update the value to that of the staging repository and commit that.

This will make it easy for the other platform deployment. 

Then checkout the repo on the other platforms and run the following on each of the other platforms:

    ./mvnw clean deploy -P apache-release

> Note: For some reason you will see errors a the end when deploying the other artifacts, however in all cases I did see the artifacts deployed correctly.

Once this has been run on each of the supported platforms, go back to Nexus and close the staging repository.

## Prerequisites

The following software needs to be installed in order to build the thrift module:

- Java
- Flex
- Bison
- Boost
- Ssl

Please look in the IoTDB documentation for information on how to install them on your particular OS.
