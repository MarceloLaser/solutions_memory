# DISCLAIMER

You found my notes. Congratulations. If you'd like to give me suggestions on how to improve the snippets, I'd love that. If my snippets are wrong, let me know, it's been a while since I wrote some of these. If you don't like my tone, do kindly keep it to yourself.

# Git

A few helpful tidbits for dealing with Git, since Git is quite the git.

## Lola

This is a helpful command for visualizing the commit history of your repository. To exit lola, press q. Taken from http://blog.kfish.org/2010/04/git-lola.html, which itself has all the information you need, but for reference, copy this into gitconfig and be happy:

[alias]
        lol = log --graph --decorate --pretty=oneline --abbrev-commit
        lola = log --graph --decorate --pretty=oneline --abbrev-commit --all
[color]
        branch = auto
        diff = auto
        interactive = auto
        status = auto

## Removing file from history

I've tried half a dozen things, and for some reason I can't get this right, UNLESS I use the trick described in https://stackoverflow.com/a/2158271. Specifically, I use the filter-branch option. His tutorial is far better than anything I could ever make, so just look there.

# Maven

For starters just a few snippets for Maven. The Maven documentation, as well as its community, drive me nuts. Either its over-specified or under-specified, and half the time I get holier-than-thou responses on why I shouldn't do what I want to do in the first place, rather than helping me solve the damn problem. So, here are some snippet solutions that'll just do what I want without wasting my time.

## Template

Sometimes I forget the damn structure of the XML. This is it.

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion></modelVersion>
  <groupId></groupId>
  <artifactId></artifactId>
  <packaging></packaging>
  <version></version>
  <name></name>
  <url></url>
  <dependencies>
    <dependency>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
      </plugin>
    </plugins>
  </build>
</project>
```

## Compiler Version

If the bloody Java compiler acts up, this seems to fix it. Sometimes it's not necessary. I dunno why. Theoretically, the default Maven Java compiler uses version 1.5 for some ungodly reason, or at least that's what I found online. Either way, if javac whines, add this plugin.

```
<plugin>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration>
    <source>1.8</source>
    <target>1.8</target>
  </configuration>
</plugin>
```

Another option is to add the following before your dependencies:

```
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

You should add this anyway so maven quits whining about warnings. I don't know why the archetype generator doesn't just add it automatically, and I'm afraid to ask.

## Linking plugins to phases

To run a plugin automatically during a particular phase, add this to it at the end. GOAL depends on the plugin, each plugin has a number of goals.

```
<executions>
  <execution>
    <id>make-assembly</id>
    <phase>package</phase>
    <goals>
      <goal>GOAL</goal>
    </goals>
  </execution>
</executions>
```

## Jar with dependencies

Because there are 317 different assembly plugins, if you search how to make a jar with dependencies you'll get some 318 different answers. This is how I've been doing it, and it works okay. This will make sure that the .jar output will contain all of the necessary dependencies (will be a "fat jar").

```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-dependency-plugin</artifactId>
  <version>3.1.1</version>
  <executions>
    <execution>
      <id>copy-dependencies</id>
      <phase>prepare-package</phase>
      <goals>
        <goal>copy-dependencies</goal>
      </goals>
      <configuration>
        <outputDirectory>${project.build.directory}/lib</outputDirectory>
        <overWriteReleases>false</overWriteReleases>
        <overWriteSnapshots>false</overWriteSnapshots>
        <overWriteIfNewer>true</overWriteIfNewer>
      </configuration>
    </execution>
  </executions>
</plugin>
```

## Using local libraries

This one is a nightmare. Google how to use a local library and at least 60% of the answers will be mobs with pitchforks and torches trying to burn you at the stake. Just add this plugin. You're an adult, you know what you're doing. If you don't, I don't particularly care.

```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-install-plugin</artifactId>
  <version>3.0.0-M1</version>
  <executions>
    <execution>
      <id>hack-binary</id>
      <phase>clean</phase>
      <configuration>
        <file>${basedir}/lib/[[[YOUR_LIB]]].jar</file>
        <repositoryLayout>default</repositoryLayout>
        <groupId>[[[LIB_GROUP_ID]]]</groupId>
        <artifactId>[[[LIB_ARTIFACT_ID]]]</artifactId>
        <version>[[[LIB_VERSION]]]</version>
        <packaging>jar</packaging>
        <generatePom>true</generatePom>
      </configuration>
      <goals>
        <goal>install-file</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

To install multiple ones, just add extra executions. To use them, add them as a dependency, same as any other.

```
<dependency>
  <groupId>[[[LIB_GROUP_ID]]]</groupId>
  <artifactId>[[[LIB_ARTIFACT_ID]]]</artifactId>
  <version>[[[LIB_VERSION]]]</version>
</dependency>
```

## Building the damn thing

This just builds an executable jar. The classpath prefix makes it access your local libs (that were packaged into the jar with maven-dependency-plugin). Surprisingly enough, the documentation on this is actually quite good, so:

http://maven.apache.org/shared/maven-archiver/examples/classpath.html

```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <version>3.0.2</version>
  <configuration>
    <archive>
    <manifest>
    <addClasspath>true</addClasspath>
    <classpathPrefix>lib/</classpathPrefix>
    <mainClass>[[[FULLY_QUALIFIED_CLASSNAME]]]</mainClass>
    </manifest>
    </archive>
  </configuration>
</plugin>
```

## Building multiple jars

Sometimes a project can have several different packagings. You can specify them with this.

```
<plugin>
  <artifactId>maven-assembly-plugin</artifactId>
  <version>2.5.5</version>
  <executions>
    <execution>
      <id>[[[EXECUTION_ID]]]</id>
      <configuration>
        <descriptorRefs>
          <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <finalName>[[[OUTPUT_JAR_NAME]]]</finalName>
        <archive>
          <manifest>
            <mainClass>[[[FULLY_QUALIFIED_CLASSNAME]]]</mainClass>
          </manifest>
        </archive>
      </configuration>
      <phase>package</phase>
      <goals>
        <goal>single</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

Again, to add more packagings, add more executions. All packages will contain everything, the only difference is the main class. There is a way to specify exactly what you want in the package, but I don't remember the details on how to do that.

## Useful dependencies

commons-lang3: I used this one primarily for the org.w3c.dom.Document class. Handy representation of XML-style data.

```
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.2</version>
</dependency>
```

guava: It's guava. Has a bunch of stuff. I used it for the iterators.

```
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>19.0</version>
</dependency>
```

Apache Commons IO: Neat for file handling.

```
<dependency>
  <groupId>commons-io</groupId>
  <artifactId>commons-io</artifactId>
  <version>2.6</version>
</dependency>
```
