# Project description
This is a showcase for Kotlin Android application. Project utilizes most popular libraries, unit testing and architecture which is suitable
for long [development live cycle](https://en.wikipedia.org/wiki/Systems_development_life_cycle).

[![CircleCI](https://circleci.com/gh/igorwojda/lastfm.svg?style=svg)](https://circleci.com/gh/igorwojda/lastfm)
[![codebeat badge](https://codebeat.co/badges/4e47bc97-16fa-4dda-ae36-c24da47ea4c4)](https://codebeat.co/projects/github-com-igorwojda-lastfm-master)

# Project characteristics
* 100% Kotlin
* Kotlin Coroutines
* CA + MVVM (Clean Architecture + Model-View-ViewModel)
* Gradle Kotlin DSL
* Feature modules
* Dependency Injection (Kodein)
* AndroidX support libraries
* Android Architecture components
* Unit Tests
* Utilise [many popular libraries](buildSrc\src\main\kotlin\LibraryDependency.kt) from Android ecosystem
* Takes advantage of most popular static analysis tools
* Gradle dependency autocompletion

# Kotlin
Project takes full advantage of Kotlin language by maximizing it's usage across project:
* Application code is written in Kotlin
* Gradle build scripts (eg. [top level](build.gradle.kts), [module level](app/build.gradle.kts) etc.) are written in Kotlin utilising
[Gradle Kotlin DSL](https://github.com/gradle/kotlin-dsl)
[Kotlin DSL](https://confluence.jetbrains.com/display/TCD18/Kotlin+DSL)
* Dependency injection is implemented using [KodeinDI](https://kodein.org/di/)

Heavy usage of Kotlin allows to o speed up development process, decrease learning curve and improves project maintainability.

# Architecture
Some architectural decisions may look like overkill for such small project, however they will scale very well for a project with long live
cycle, maintained by larger team.

## Feature modules
Each feature is contained in separate module. This allows to easily maintain particular feature, move feature to different
project, or just delete it. Project utilizes domain centric
[clean architecture](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) at feature level. This means that each
feature follows [dependency rule](https://proandroiddev.com/clean-architecture-data-flow-dependency-rule-615ffdd79e29) having its own
set of layers (`presentation`/`domain`/`data`). All layers (for a given feature) exists in a single module, so dependency rule have to be
manually enforced by custom lint check (instead of module dependency).

## MVVM
MVVM (presentation layer) utilises Android Architecture components (`ViewModel` + `LiveData`). ViewModel uses Kotlin coroutines to retrieve data
using background thread, while `LiveData` is responsible for delivering data to a View (`Fragment`) in Lifecycle aware manner.

## Architecture extension
Architecture of this project can be scalded further up depending on he project needs. If we want to delay UseCase execution we could
introduce additional `Request` and `Response` objects for each `UseCase`. To deal with data caching we could introduce more layers eg.
split `data` layer into `network`, `memory` and `disk` layers containing own data models (`AlbumNetworkModel` / `AlbumMemoryModel` /
`AlbumDiskModel`). To introduce multiple types of items in RecyclerViews and easily share them across app we could take advantage of
[AdapterDelegates](https://github.com/sockeqwe/AdapterDelegates) library etc.

As a result of using CA we gain clean layer separation for feature contains multiple files, however this lead to large amount of files.
To safe precious developer time we can create a feature template using AS or some template language (after entering feature name all files
and packages will be quickly generated for new feature)

Also since all layers of CA are inside single module layer dependencies can't be enforced simply by defining module dependencies. We
need to define additional lint check to make sure that dependency rule is protected.

Finally due to proper layer separation we can easily swap libraries by modifying only small part fo the application eg. Retrofit can
be used instead of Fuel - only data layer will be affected, no additional code change is required in other layers (presentation/domain)

# CI configuration
CI configuration is stored in the repository. This approach allows to easily update CI build configuration and validate it's correctness
together with each PR before merging the code.

# Gradle
## Gradle Kotlin DLS
[Kotlin Gradle DSL](https://github.com/gradle/kotlin-dsl) provides statically-typed approach that allows many real-time code checks and
better IDE support.

##  Centralized dependencies
Gradle does pretty good job regarding to [dependency management](https://docs.gradle.org/current/userguide/introduction_dependency_management.html).
For projects that include multiple modules, we also need to structure our project in proper way so simple tasks like upgrading library
version in 20 modules are easy to accomplish (single place code change). To achieve this project utilizes [project-level properties](buildSrc/src/main/kotlin)
that are shared across all modules.

## Dependency code completion
All dependencies in the project are defined in Gradle [buildSrc](https://docs.gradle.org/current/userguide/organizing_gradle_projects.html#sec:build_sources)
directory. Upon discovery of the directory, Gradle automatically compiles code in `buildSrc` and puts it in the classpath of our build
script. We can easily access various kinds of dependencies in our build scripts, have a full code completion and do not worry about
misspelled, hardcoded dependency strings:

- [LibraryDependency](buildSrc\src\main\kotlin\LibraryDependency.kt) - class contains all dependencies of the libraries used in project
- [ModuleDependency](buildSrc\src\main\kotlin\LibraryDependency.kt) - class contains all dependencies of project modules
- [GradleDependency](buildSrc\src\main\kotlin\LibraryDependency.kt) - class contains all dependencies of Gradle plugins.

Here is a code snippet from build script:

```kotlin
//Module depends on base feature module
implementation(project(ModuleDependency.featureBase))

//We want to use these libraries in this module
implementation(LibraryDependency.supportAppCompact)
implementation(LibraryDependency.timber)
```

This allows to unify libraries versions across project and easily share them across all the modules.

# Color management
When the project gets bigger and requires more colors, it is getting really confusing and less traceable how to use the same colors in
different contexts for different views. Solution for this problem is spiting collars into application color and color palette:
* palette colors are named by color name (using [name the color tool](http://chir.ag/projects/name-that-color/)) eg. `blue`,
`magenta`. They can only be used to define application colors.
* application colors are named by their function, from general function to more specific e.g. `buttonPrimaryEnabled`, `buttonPrimaryText`.
They must always use color from the palette instead of defining new color and they can be widely used across all application.

More info about
<https://blog.novatec-gmbh.de/name-android-colors-palettes/>

# Tools

## Dependency version checker
By running `./gradlew dependencyUpdates` task (from [Gradle Versions Plugin](https://github.com/ben-manes/gradle-versions-plugin)) we can
easily list all outdated dependencies in the project.

## Static analysis
Project includes all modern [static code analisys](https://en.wikipedia.org/wiki/Static_program_analysis) tools for Kotlin
 ([ktlint](https://github.com/shyiko/ktlint), [detekt](https://github.com/arturbosch/detekt)) and Android platform
 ([Android lint](https://developer.android.com/studio/write/lint)). Each of those tools is focusing on different area of static analysis.
  `ktlint` checks code formatting, `detekt` deals with code smells, complexity, performance checks and finally `lint` verifies behaviours
   specific to android platform.

To have better understanding, the difference between those tools let's look at sample checks performed by each tool:

`ktlint` - guards the code formatting eg. each function parameter must be on a separate line / no space allowed after colon in variable declaration line

`detekt` - guards code complexity, correctness and security eg. method is to long / property may be const

`Android lint` - guards Android platform specific rules eg. unused android resource / activity not declared in manifest

Project contains custom gradle task that can run all of the static analysis checks at once - `./gradlew staticCheck`.

BTW: [Checkstyle](https://checkstyle.sourceforge.io), [PMD](https://pmd.github.io/) and [FindBugs](http://findbugs.sourceforge.net/)
don't work with Kotlin (only Java), so they are not included in this project.

### Android lint
`./gradlew lint` - run lint checkGradle Kotlin DSL

### ktlint
ktlint is integrated via [Ktlint Gradle](https://github.com/jlleitschuh/ktlint-gradle) witch is gradle plugin over the `ktlint` project.

`./gradlew ktlintCheck` - run ktlint check

`./gradlew ktlintFormat` - runs the ktlint formatter on all kotlin sources in this project.

Ktlint follows `Android Kotlin Style Guide`. For `Android Studio` to be compliant with `Android Kotlin Style Guide` we need to run
gradle task that will update IDE code formatting settings (generate Kotlin style files).

`./gradlew ktlintApplyToIdeaGlobally` - The task generates IntelliJ IDEA (or Android Studio) Kotlin style files in the user home IDEA
(or Android Studio) settings folder. This allows to keep consistent Kotlin formatting across all projects and this task can be runned only
once (no matter in which project). Usually this is best way to go, unless you have some old projects that you don't want to update then
`./gradlew ktlintApplyToIdea` would be better option.

`./gradlew ktlintApplyToIdea` - The task generates IntelliJ IDEA (or Android Studio) Kotlin style files in the project `.idea/` folder.
This allows to keep consistent formatting in a single project and have to be run separately for each project.

### detekt
`./gradlew detekt` - run detekt check

# Contribute
If you think something is incorrect or you have found a better solution please create PR or open a new issue.

# Follow me
![avatar.png](misc/image/avatar.png)

[Twitter](https://twitter.com/igorwojda) | [Medium](https://medium.com/@igorwojda) | [Linkedin](https://www.linkedin.com/in/igorwojda/)
