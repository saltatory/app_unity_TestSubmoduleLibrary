Unity + Git Demonstration Project
=================================

This project demonstrates a sample project layout for Unity using Git. Specifically, it uses Git submodules to organize shared components.

# Checkout and Run

* Checkout by `git clone <url> --recursive`. This will fetch the submodules as well.
* Open in Unity and run!

# Directory Structure

Assets and libraries may be externally developed either internally by other teams or externally by other companies. These libraries should be cloned as Git Submodules under `Assets/Libraries`. 

For example, the Test library is stored in `Assets/Libraries/lib_unity_Test`.

# Motivation

In a previous post, I talked about the challenges of developing platforms and tools at game companies. From an organization perspective, the chief challenge is to provide value to game teams without becoming either a bottleneck or a body shop. 

Becoming a bottleneck is easy. Game teams are almost always under a time crunch. At various points in the project a game team will be very open to the idea of someone from outside the team swooping in, solving problems, and generally taking work off their plate. The problem is that platform and tools teams serve multiple masters. The face that there is a platform and tools team means there are several game teams acting as customers to that team. That means that the platform and tools team is taking requests for features or interrupts for problems from Team B at the same time they are trying to develop a feature for Team A. Often that makes it impossible to plan effectively and often ends up with the shared code being late and ultimately blocking progress by the game team. 

Another common way to become a bottleneck is to build something so general that it takes longer to build and is ultimately less useful. In trying to serve multiple internal customers, the platform & tools team will be thinking ahead to how a feature will be used by multiple customers. Often, this results in over-thinking the feature and building in stuff that is not needed and perhaps no one will use.

Becoming a body shop is equally easy. A game team that has been a customer of a platform & tools team may have been burned by the bottleneck factor. On feature 2, they are now gun-shy and want to control the scope and timeline of the work being done by the platform & tools team. To accomplish this, they fully incorporate the engineer working on the shared feature into the game team working on their project schedule and process. Thing is, the platform & tools team loses all control over what that person engineer is working on. It's so so easy for the game team to think "Hey, I have a talented engineer here and plenty to work on. Let's have him/her start on the next feature". The shared feature might never be completed now or, if it is, the shared engineer might never return to the platform & tools team!

# Solution

## TL;DR

Use Open Source methodologies to develop shared components.

The solutions to the problem are manifold. In a separate post, I will discuss the process solution. However, to facilitate the process solution, we need some engineering practices. The best ones come from the Open Source software world.

## Version Control - Use Git

I'm opinionated but, in my opinion, Git is the best thing going when it comes to version control. It supports branch in place. It allows you to work offline. With Git Flow, it has a mature branching strategy that plays nicely with how production code is actually written and deployed. It has great merging and diff tools. The list goes on.

The big win for component development is that it is common practice to have a separate repository for every component.

## Git Submodules for Libraries

There is quite a bit of discussion on the use of package managers for handling external dependencies. See the references section at the end for more information on the opinion of others. 

For iOS projects, which a Unity project might be, there is CocoaPods. For C# projects, there is NuGet. A package manager offers a declarative way of specifying project dependencies.

To summarize the arguments in favor:
* Easy to include. Typically including a dependency is a one-liner.
* Small. The default use of most package managers keeps external libraries out of your source control.
* Easy to update. As the open source (internal or external) community makes changes to a library, it is automatically updated.
* Faster builds. You don't rebuild intermediate artifacts over and over again.
* Build artifacts are stable. If you lose a build environment or change a config (e.g. upgrading a compiler), you don't lose the stable artifact.

The problem that I see with package managers is that they go against how we treat other development artifacts within our environment. Now that disk is incredible cheap, I'm a fan of version controlling almost everything. In particular, I want everything that goes into constructing a game to be versioned and promoted from environment to environment in an automated and controlled fashion. It is this automation that yields the certainty that what is built is what is tested and what is tested is what is launched into the wild. For configuration systems like A/B testing systems, this means version controlling the configs as well as the code that consumes them. For external libraries, this means putting them under version control.

Internal version control. While the OSS community is very active, the repository containing an artifact will be maintained. However, when development stops for any reason including that the original developer got bored and lost interest, the repository might simply up and disappear.

One might argue that you could version control the binary artifact and use the package manager as a developer tool. For example, running `pod install` on an iOS library, will download the source code and/or a library. You could then put the `Pods` directory under version control. The good part is that you have the asset under version control. The downside is that you don't have a "mature" process for upgrading the dependency. Sure, you can make a change to the referenced release version and re-run `pod install` or run `pod upgrade`. But what if you want to make a source code change especially one that only applies to you? To keep that change, you have to commit the repo to version control locally.

One proposed solution to the problem of foreign repositories going away is to host intermediate binary artifacts. This practice is popular in the Java world. In Java, maven will build `.jar` files. It will also download and cache dependent `.jar` files. Local caches will get purged periodically so to prevent needed items from disappearing, you could use a product like Artifactory or Sonatype Nexus. The challenge here is that these tools can be complicated to use. You have to host them, back them up, and establish access and permissions. All things you have already done in git especially if you are using Stash or Bitbucket.

What about compile speed? I'm sure it matters. Somewhere. In recent years, it just doesn't seem to be an issue with modern compilers and tools.

What about stable build artifacts? This is a real problem. Some build systems like Bamboo cache these intermediate artifacts alleviating the need for a separate artifact service. That's good for overall build speed but I think it masks the stability problem. A build environment should be versioned just like what it is meant to build. Tools like Vagrant/Docker are a better solution to the problem of a stable build environment.

Replacing artifact storage is where git and git submodules comes it. The better approach is to add a git submodule to your parent project. If the code is originally written by an external developer, you should fork the repository to a local copy. This protects you from the original disappearing. It also gives a mature process for pushing proposed changes upstream. That process works equally well for internally developed modules on different teams. For example, if a game team is using a module produced by a tools team and finds that the original has issues or is missing a needed feature, they can simply fork the repo and use the forked copy instead. They can then push proposed changes upstream as well as pull changes made by the upstream team. 

It sounds so good, what are the cons? As far as I can tell, the chief con to using git submodules is that they are not easy to understand. I have personal experience that this is so. Some of the command line invocations are arcane. The most frequent problem is that it is easy to make a commit in one repository of a project e.g. the submodule repository and forget to make a corresponding add/commit to include the change e.g. in the parent repository. It's a problem, to be sure. However, this is the price we pay for a solid process and I think it is eminently teachable.

# Git Submodule and Unity

What follows is my proposed structure for Unity projects that include common types of shared libraries.

## Overall Project Structure

Clearly, we should use [Unity special folder names](http://docs.unity3d.com/Manual/SpecialFolders.html) where appropriate.

```
|- Assets
|--| Libraries
|----| <submodule>
```

For example, suppose that I have a shared library called `lib_unity_Debug` which is, appropriately, a library for debugging output. I would create a git repo for that component and then clone it into the `Assets/Libraries` folder with a command like :

```
~/workspace/app_unity_TestProject/Assets/Libraries $ git submodule add git@github.com:saltatory/lib_unity_Debug.git
```

Yielding a directory structure like :

```
|- Assets
|--| Libraries
|----| lib_unity_Debug
```

Because of the special folder names, our library may look like this :

```
|- Assets
|--| Libraries
|----| lib_unity_Debug
|------| Editor			# Extensions to the Editor
|------| Gizmos			# Sceneview graphics
|------| Plugins		# Compiled native libraries produced at build
|------| Resources		# Graphics, sounds, etc. for this library
|------| Editor			# Extensions to the Editor
|------| Prefabs		# Recommended parent for prefabs
```


# References

* http://gitslave.sourceforge.net/
* http://answers.unity3d.com/questions/631546/using-a-nuget-package-jsonnet.html
* http://askubuntu.com/questions/481002/how-to-install-nuget-addin-for-monodevelop
* http://code.tutsplus.com/tutorials/creating-your-first-cocoapod--cms-24332
* http://twobitlabs.com/2013/12/not-cuckoo-for-cocoapods/
* http://albertodebortoli.github.io/blog/2014/03/11/cocoapods-working-with-internal-pods/
* http://www.brianjcoleman.com/tutorial-using-sub-projects-git-submodules-to-create-a-framework-in-ios/#prettyPhoto

