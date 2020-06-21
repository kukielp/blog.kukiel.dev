---
layout: post
title:  "Compiling Fuseless in Cloud 9 for Coldfusion/CFML in AWS Lambda"
date:   2020-06-20 19:18:01 +1000
categories: aws lambda apigateway cfml lucee java coretto
---

![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/header.png "Function")

In the [previouse article on CFML](https://blog.kukiel.dev/posts/CFML-Lucee-in-Lambda-with-APIGateway.html) I provided a very high level overview of CFML in Lambda.  In this post I will show how easy it is to:
- Install a JDK on Cloud9
- Install gradle for java projects
- Compile Fuseless from source
- Deploy a CFML mini app to AWS Lamba

The video provided shows htat in 7 minutes I was able to from a completly fresh enviroment deploy a PoC CFML lambda app.

Every step exectued is listed below with some commentary.

```bash
# Install Coretto ( java 1.8 ) - https://aws.amazon.com/corretto this is the jdk required to compile Fuseless
sudo yum install -y https://corretto.aws/downloads/latest/amazon-corretto-8-x64-linux-jdk.rpm
```
![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/coretto.png "Function")


```bash

# Download gradle and unzip it to /opt/gradle
wget https://services.gradle.org/distributions/gradle-6.5-bin.zip -P /tmp

sudo unzip -d /opt/gradle /tmp/gradle-6.5-bin.zip

# Set 2 EVN variables to add gradel to the path 
export GRADLE_HOME=/opt/gradle/gradle-6.5
export PATH=${GRADLE_HOME}/bin:${PATH}

# Ensure the ENV variables persist in a new terminal session
cat >> ~/.bash_profile  <<TXT
export GRADLE_HOME=${GRADLE_HOME}
export PATH=${GRADLE_HOME}/bin:${PATH}
TXT
```
![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/coretto.png "Function")

Open a new terminal ( Java home wont be set after the corretto install untill you set it or opena  new terminal ).

```bash
# Check is java is avaliable for use
java -version
# Check is gradle is avaliable for use
gradle -v
```
![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/java.png "Function")

Clone and build Fuseless
```bash
git clone https://github.com/foundeo/fuseless.git

cd fuseless

# Run the test shell script, this will build the package, download lucee light and run some tests.
./test.sh
```
![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/clone-fuse.png "Function")

Look for the passing tests:

![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/pass.png "Function")

# Deployment

```bash
# Deploy section, clone the fuseless-template repo, there is an open PR to include the out outputs of the API endpoint 
# for now the repo is a fork of fuseless-template
cd ..

git clone https://github.com/kukielp/fuseless-template.git

cd fuseless-template

# This shell script will download the fuseless jar and lucee light plus 
./init.sh

# This will package up/zip the cfml and jars required
gradle build
```

![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/build-2.png "Function")

```bash
# Guideded process to deploy.
sam deploy --guided
```
![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/1.png "Function")

![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/2.png "Function")

![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/3.png "Function")

Open the URL, be sure to append the template in this example it's dump.cfm

![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part2/final.png "Function")

Full video:

<div class="containerFrame">
  <iframe class="responsive-iframe" src="https://www.youtube.com/embed/K3fO9buAdtE" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

If you are not sure what CLoud 9 is it's basical a Virtual machine with a web based IDE.  I am runnign Amazon Linux.  You can read more about it here: [https://aws.amazon.com/cloud9/](https://aws.amazon.com/cloud9/)