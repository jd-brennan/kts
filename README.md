# kts

Playground for Kotlin Scripts that use private libraries

# Design One - Authenticated Maven Repos

The code for Repository annotation (defined in
[annotations.kt](https://github.com/JetBrains/kotlin/blob/v1.9.21/libraries/scripting/dependencies/src/kotlin/script/experimental/dependencies/annotations.kt#L31))
takes variable arguments:

```
annotation class Repository(vararg val repositoriesCoordinates: String, val options: Array<String> = [])
```

From reading [SimpleExternalDependenciesResolverOptionsParser.kt](https://github.com/JetBrains/kotlin/blob/v1.9.21/libraries/scripting/dependencies/src/kotlin/script/experimental/dependencies/impl/SimpleExternalDependenciesResolverOptionsParser.kt)
it parses the options by looking for name=val in each option. So let's try:

```
@file:Repository("https://invitae.jfrog.io/artifactory/java", options=arrayOf(
    "username=${System.getenv("ARTIFACTORY_USER")}", 
    "password=${System.getenv("ARTIFACTORY_TOKEN")}")
)
```
which yields:
```
error: an annotation argument must be a compile-time constant
```

I guess the script has to contain the password. Yuck! Let's try it:

```
@file:Repository("https://invitae.jfrog.io/artifactory/java", options=arrayOf(
    "username=jd.brennan@invitae.com", 
    "password=REDACTED")
)
```
which yields:
```
error: failed to parse options from annotation. Expected a valid option name but received:
@invitae.com
```
Damn. If only my username didn't have an @ character. Tried \\@ and \\\\@ but those didn't work either.

Filed [PR 5237](https://github.com/JetBrains/kotlin/pull/5237) in the kotlin repo to fix this.

# Design Two - Local Maven Repo

Let's create a lib folder and put the jar file there

```
mkdir lib
cp /wherever/invitae_reqrepo_v3-3.6.22.jar lib
```
And run our script
```
@file:Repository("file:/Users/jd.brennan/dev/jd-brennan/kts/lib")
@file:DependsOn("invitae_reqrepo_v3-3.6.22.jar")

import com.invitae.invitae_reqrepo_v3.models.RequisitionItem
```

Success!