---
layout: post
title: Releasing artifacts by maven-release-plugin
---

# Prerequisites
* installed Git Bash (Windows)

The project's pom.xml also needs 
* Description
* SCM-Section
* in build-section
** JavaDoc-Artifact-Generation
** Sources-Artifact-Generation
** Signing

# GitHub: Checking for existing SSH keys
* open Git Bash
* enter `ls -al ~/.ssh` to see if there are existing SSH keys present

Default filenames of the public keys are:

* id_dsa.pub
* id_ecdsa.pub
* id_ed25519.pub
* id_rsa.pub

Checking for existing SSH keys](https://help.github.com/articles/checking-for-existing-ssh-keys/)

# GitHub: Generating a new SSH key
* open Git Bash
* generate ssh key pair
```shell
# Generating public/private rsa key pair.
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

* When you're prompted to "Enter a file in which to save the key," press Enter to accept the default file location
```shell
Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
```
* in the next step the passphrase should **not** be **empty**
```shell
Enter passphrase (empty for no passphrase): [Type a passphrase]
Enter same passphrase again: [Type passphrase again]
```

[Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

# GitHub: Adding SSH key to the ssh-agent
* turn on ssh-agent (Git Bash):
```shell
$ eval "$(ssh-agent -s)"
Agent pid 59566
```

* add ssh key to ssh-agent
```shell
$ ssh-add ~/.ssh/id_rsa
```

[Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

# GitHub: Adding SSH key to GitHub account
* copy ssh key to clipboard
```shell
$ clip < ~/.ssh/id_rsa.pub
```

* add ssh key from clipboard in GitHub **Settings** -> **SSH and GPG keys** -> **New SSH keys**

[Adding a new SSH key to your GitHub account](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)

# GitHub: Testing SSH connection
* open Git Bash
```shell
$ ssh -T git@github.com
```
* accept fingerprint
* verify message
```
Hi username! You've successfully authenticated, but GitHub does not
provide shell access.
```

[Adding a new SSH key to your GitHub account](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)

# GPG: Generate a Key Pair
* open Git Bash
```shell
$ gpg --gen-key
```
** type, size, and time of validity: default values can be used
** name, email and comment are essential as they will be seen by anyone downloading a software artifact and validating a signature
** passphase should be empty
```shell
# list existing keys
$ gpg --list-keys
/c/Users/bjoern/.gnupg/pubring.gpg

pub   2048R/27467A0F 2016-06-18 #<1>
uid                  Björn Engel <bjoern@example.de>
sub   2048R/4AC3ED13 2016-06-18
```
<1> the relevant public key (27467A0F)

* distribute public key to a key server with 
```shell
$ gpg --keyserver hkp://pgp.mit.edu --send-keys 27467A0F
```

[How to Generate PGP Signatures with Maven](http://blog.sonatype.com/2010/01/how-to-generate-pgp-signatures-with-maven/)

# Sonatype: add required elements in pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
...
<groupId>com.github.justcoke</groupId> //<1>
<artifactId>demo</artifactId> //<2>
<version>1.0</version> //<3>
...
<name>Demo Application</name> //<4>
<description>Artifact's description</description> //<5>
<url>http://www.example.com/example-application</url> //<6>
...
<licenses> //<7>
<license>
<name>Apache 2.0</name>
<url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
<distribution>repo</distribution>
</license>
</licenses>
...
<developers> //<8>
<developer>
<name>Björn Engel</name>
<url>http://about.me/bjoernengel</url>
<timezone>Europe/Berlin</timezone>
</developer>
</developers>
...
<scm> //<9>
<developerConnection>scm:git:git@github.com:justcoke/properties-maven-plugin.git</developerConnection>
<connection>scm:git:https://github.com/justcoke/properties-maven-plugin.git</connection>
<url>https://github.com/justcoke/properties-maven-plugin.git</url>
  <tag>HEAD</tag>
   </scm>
...
<!-- configure Maven to deploy to the OSSRH Nexus server with the Nexus
Staging Maven plugin -->
<distributionManagement> //<10>
<!-- the Maven deploy plugin needs a full distributionManagement section -->
<repository>
<id>ossrh-release</id>
<url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
<uniqueVersion>true</uniqueVersion>
</repository>
<snapshotRepository>
<id>ossrh-snapshot</id>
<url>https://oss.sonatype.org/content/repositories/snapshots</url>
<uniqueVersion>false</uniqueVersion>
</snapshotRepository>
<!-- The above configurations will get the user account details to deploy
to OSSRH from Maven settings.xml file. -->
</distributionManagement>
...
<build>
<plugins>
<plugin> //<11>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-release-plugin</artifactId>
<version>2.5.3</version>
<configuration>
<autoVersionSubmodules>true</autoVersionSubmodules>
<releaseProfiles>release</releaseProfiles> //<12>
</configuration>
</plugin>
</plugins>
...
</build>

<profiles>
<profile>
<id>release</id> //<13>
<build>
<plugins>
<plugin> //<14>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-source-plugin</artifactId>
<version>2.2.1</version>
<executions>
<execution>
<id>attach-sources</id>
<goals>
<goal>jar-no-fork</goal>
</goals>
</execution>
</executions>
</plugin>
<plugin> //<15>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-javadoc-plugin</artifactId>
<version>2.9.1</version>
<executions>
<execution>
<id>attach-javadocs</id>
<goals>
<goal>jar</goal>
</goals>
<configuration>
<additionalparam>-Xdoclint:none</additionalparam>
<show>public</show>
<quiet>true</quiet>
</configuration>
</execution>
</executions>
</plugin>
<plugin> //<16>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-gpg-plugin</artifactId>
<version>1.6</version>
<executions>
<execution>
<id>sign-artifacts</id>
<phase>verify</phase>
<goals>
<goal>sign</goal>
</goals>
</execution>
</executions>
</plugin>
</plugins>
</build>
</profile>
...
</profiles>
</project>
```

<1> top level namespace level for your project starting with the reverse domain name
<2> unique name for component
<3> version string for component
<4> project name
<5> project description
<6> project website
<7> license information
<8> developer information
<9> SCM information
<10> distribution management's information
<11> using maven-release-plugin
<12> profile's name for release
<13> profile for release
<14> released artifact's source
<15> released artifact's javadoc
<16> signing released artifact

[Requirements](http://central.sonatype.org/pages/requirements.html)

# Updating settings.xml
* add sonatype login data in maven's settings.xml
```xml
<server>
    <id>ossrh-release</id>
    <username>your-jira-id</username>
    <password>your-jira-pwd</password>
</server>
```

# Releasing
```shell
$ mvn release:clean release:prepare
```
and
```shell
$ mvn release:perform
```

If an error occures run `mvn release:rollback` to revert changes made by prepare-step.

* visit [https://oss.sonatype.org/](https://oss.sonatype.org/) to check uploaded artifact
* **close**
* **release**
* wait

[Maven Tips and Tricks: Using GitHub](http://blog.sonatype.com/2009/09/maven-tips-and-tricks-using-github/)
[Releasing the Deployment](http://central.sonatype.org/pages/releasing-the-deployment.html)