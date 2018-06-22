# Travis CI msbuild example setup
### License: MIT

----

This repository contains a pre-compiled copy of .NET Core MSBuild able to run on Travis CI.

Clone this repository in your .travis.yml, add this repository as submodule or copy its contents to your repository.  
Apply the settings in the .travis.yml to the .travis.yml in your repository and replace xbuild with msbuild.

All pre-compiled files here (.NET Core MSBuild; Mono Framework API) were compiled from MIT-licensed projects. The Mono Framework API .dlls are required to compile projects against the .NET Framework API.

# Possibly Asked Questions (PAQ)

### *Why?*

Why not?

On a more serious note, there are some issues when compiling some projects using xbuild.  
For example, the resulting WinExe requires administrator permissions on Windows, the WinForms layout is different than when building using msbuild or drag-n-drop support is simply missing.

### Why not use Ubuntu 12.04 (precise)?

MSBuild, as of time of writing, officially supports Ubuntu 14.04, while also running on other distributions with minor modifications (16.04 after installing libicu52).
I've tried getting MSBuild to run on the Travis Ubuntu 12.04 container environment, but it failed due to issues with the libunwind8 dependency.

### Why not use the Travis container environment / Why does this require root?

Root / sudo is not required to get msbuild running on Travis. It's just that as of time of writing, the Ubuntu 14.04 (trusty) environment beta only runs with `sudo: required`.

### Why not use git lfs?

git lfs currently requires additional setup in your .travis.yml. The currently best method I've found to install git lfs is:

```yaml
before_install:
    - curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
    - sudo apt-get install -y git-lfs=1.1.0
	- git lfs install
    - git lfs pull
```

`git-lfs` isn't even whitelisted, so it's not possible to setup git lfs that way in builds running in containers (without sudo access).

### Why ship msbuild.zip? Why not already extract it?

Compression. And if `p7zip` would be installed by default, I would've 7zip-d it instead.

### Are there limitations?

* Currently debug symbol compilation is impossible - .NET Core MSBuild simply fails and .NET Core uses the new portable pdb format. Apply following changes to your .csproj and build using Release:

```xml
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|x86' ">
    <DebugType Condition=" '$(TravisCore)' == '' ">pdbonly</DebugType>
```

* .NET Core MSBuild doesn't handle compiling projects against the complete .NET Framework by default. The setup here bypasses that limitation by shipping with a subset of pre-compiled .NET 4.5 API .dlls from Mono.
If you need to target a larger (untested) set of .NET Framework APIs or another (untested) framework version, ship the required Mono pre-compiled API .dlls and set `TRAVIS_MSBUILD_FRAMEWORK` in .travis.yml accordingly.

* Ubuntu 14.04+ is required (until the libunwind8 dependency issue is fixed for Travis container builds).

* As the build no longer runs in the new shiny container setup, the build takes longer to kick off. At least it builds faster as mono doesn't need to be installed by Travis anymore.
