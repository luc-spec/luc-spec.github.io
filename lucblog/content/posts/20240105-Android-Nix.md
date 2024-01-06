+++
title = '20240105 Android, Nix, and Even Some Education'
date = 2024-01-05T22:11:49-07:00
draft = true
+++

I came across [this guide](https://gadgetbridge.org/internals/development/setup-environment/#installing-dependencies) that lays out how to start contributing to Gadgetbridge.

## It's not Nix, it's me

Since I'm in the mode of playing around with `shell.nix` files, I immediately jumped to creating one for this project.

```nix
{ pkgs ? import <nixpkgs> {} }:
  pkgs.mkShell {
    # nativeBuildInputs is usually what you want -- tools you need to run
    nativeBuildInputs = with pkgs.buildPackages; [
      #android-studio
      androidStudioPackages.dev
      git
      openjdk17-bootstrap
      gradle_7
    ];
}
```

A quick switch, and we're off to the races, right? Well, this spat out an... unhelpful error message.

```bash
~$ nix-shell
~$ ./gradlew test
Welcome to Gradle 7.5!

Here are the highlights of this release:
 - Support for Java 18
 - Support for building with Groovy 4
 - Much more responsive continuous builds
 - Improved diagnostics for dependency resolution

For more details see https://docs.gradle.org/7.5/release-notes.html

Starting a Gradle Daemon, 1 stopped Daemon could not be reused, use --status for details

FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> Multiple entries with same key: main=[] and main=[]

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

Deprecated Gradle features were used in this build, making it incompatible with Gradle 8.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

See https://docs.gradle.org/7.5/userguide/command_line_interface.html#sec:command_line_warnings

BUILD FAILED in 10s
```

```bash
~$ ./gradlew test --info
Initialized native services in: /home/luc/.gradle/native
Initialized jansi services in: /home/luc/.gradle/native
The client will now receive all logging from the daemon (pid: 606976). The daemon log file: /home/luc/.gradle/daemon/7.5/daemon-606976.out.log
Starting 2nd build in daemon [uptime: 4 mins 39.89 secs, performance: 100%]
Using 16 worker leases.
Watching the file system is configured to be enabled if available
File system watching is inactive
Starting Build
Settings evaluated using settings file '/home/luc/Repos/all/Gadgetbridge/settings.gradle'.
Projects loaded. Root project using build file '/home/luc/Repos/all/Gadgetbridge/build.gradle'.
Included projects: [root project 'Gadgetbridge', project ':app', project ':GBDaoGenerator']

> Configure project :
Evaluating root project 'Gadgetbridge' using build file '/home/luc/Repos/all/Gadgetbridge/build.gradle'.

> Configure project :app
Evaluating project ':app' using build file '/home/luc/Repos/all/Gadgetbridge/app/build.gradle'.
Using default execution profile
The com.google.protobuf plugin was already applied to the project: :app and will not be applied again after plugin: android
Starting process 'command 'git''. Working directory: /home/luc/Repos/all/Gadgetbridge/app Command: git rev-parse --short HEAD
Successfully started process 'command 'git''
Starting process 'command 'git''. Working directory: /home/luc/Repos/all/Gadgetbridge/app Command: git rev-parse --short HEAD
Successfully started process 'command 'git''
Starting process 'command 'git''. Working directory: /home/luc/Repos/all/Gadgetbridge/app Command: git rev-parse --short HEAD
Successfully started process 'command 'git''
------------------------------------------------------------------------
Detecting the operating system and CPU architecture
------------------------------------------------------------------------
os.detected.name=linux
os.detected.arch=x86_64
os.detected.bitness=64
os.detected.version=6.6
os.detected.version.major=6
os.detected.version.minor=6
os.detected.release=fedora
os.detected.release.version=39
os.detected.release.like.fedora=true
os.detected.classifier=linux-x86_64

FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> Multiple entries with same key: main=[] and main=[]

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

Deprecated Gradle features were used in this build, making it incompatible with Gradle 8.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

See https://docs.gradle.org/7.5/userguide/command_line_interface.html#sec:command_line_warnings

BUILD FAILED in 1s
```

After some more digging I was able to get everything working using the container approach!

```dockerfile
FROM android-30:latest
COPY ./my/local/copy/of/Gadgetbridge /src
```

```bash
# on the host
podman build -t gadgetbridge .
podman run --rm -ti localhost/gadgetbridge /bin/bash
```

```bash
# in the container
~$ cd /src
~$ ./gradlew test
...
[trimmed build output]
...
BUILD SUCCESSFUL in 9m 30s
255 actionable tasks: 255 executed
Not watching anything anymore
Watching 0 directories to track changes
Some of the file system contents retained in the virtual file system are on file systems that Gradle doesn't support watching. The relevant state was discarded to ensure changes to these locations are properly detected. You can override this by explicitly enabling file system watching.
Watching 0 directories to track changes
```

I still want to get a native build working; more work incoming!

