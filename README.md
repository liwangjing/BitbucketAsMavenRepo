# BitbucketAsMavenRepo

In this section, I will introduce how to use your bitbucket private repository as a maven private repository to distribute your Android Archive(aar file) with simple configured gradle file. 

###Prepare
#####1. Create a remote private repository in Bitbucket. 
* 3 most important points: (1). The owner name of the repository[it could be your bitbucket user name, or the name of your develop team]; (2). the name of the repository; (3). the repository should be set to private.

#####2. Create a local repository in your Android Studio project.
Create a git repository in the workspace directory, and create a 'releases' branch, then you can create a README.txt file, then push the 'releases' branch to the remote repo.
```bash
git push origin releases:releases
```
* in this case, in the remote repository, it will automatically create a 'releases' branch and set it as the main branch.
* in order to push a artifact to the maven repository, you must push it through a ```releases``` branch.

###Explanation of config.gradle file
#####1. Use the `wagon-git` 
`Wagon-git` is a Maven component that enables deploying artifacts in remote Git SCM repositories. For more information from [here](http://synergian.github.io/wagon-git/index.html).<br>
In order to use 'wagon-git', first add 'maven' plugin: 
```bash
apply plugin: 'maven'
```
<br>
Then import the 'wagon-git':
go to the repository through 'url', fatch the jar file through 'dependencies', then add configuration:
```bash
configurations {
    deployerJar
}

repositories {
    jcenter()
    maven {
        url "https://raw.github.com/synergian/wagon-git/releases"  // url of wagon-git in maven repository
    }
}

dependencies {
    deployerJar "ar.com.synergian:wagon-git:0.2.5"  // use the latest version file 
}
```
*Notice:* The congifuration is according to the gradle manual, for more information, check the `Example 33.4. Upload of file via SSH` in [gradle website](https://docs.gradle.org/current/userguide/userguide_single.html?_ga=1.15647102.998671448.1486064310#sec:deploying_to_a_maven_repository). In this case we are using git, instead of SSH.

#####2. Configure the pom file
POM:Project Object Model, is an XML file that contains information about the project and configuration details used by Maven to build the project.<br>
In this case, we would configure pom file for our maven repository in the uploadArchives task:
```bash
uploadArchives {
    repositories.mavenDeployer {
        configuration = configurations.deployerJar
        repository(url: 'git:releases://git@bitbucket.org:' + COMPANY + '/' + REPOSITORY_NAME + '.git') { // url of your artifact repository
            authentication(userName: USERNAME, password: PASSWORD) // access authentication for the private repository
        }

        pom.project {
            groupId = ARTIFACT_PACKAGE
            version = ARTIFACT_VERSION
            artifactId = ARTIFACT_NAME
            packaging ARTIFACT_PACKAGING
        } // configure the pom file for the maven repository
    }
}
```
**Several Parameters you need to config according to your situation: **<br>
`USERNAME`: your user name for the Bitbucket account<br>
`PASSWORD`: your password for the Bitbucket account<br>
<br>
`COMPANY`: the name of the owner of the repository<br>
`REPOSITORY_NAME`: the name of the repository<br>
`ARTIFACT_PACKAGE`: the package name of the artifact<br>
`ARTIFACT_VERSION`: the version of the artifact, like 1.0.0<br>
`ARTIFACT_NAME`: the name of the artifact<br>
`ARTIFACT_PACKAGING`: the packaging type for the artifact, in this case, we need aar file, so set it to aar<br>

*Notice:* The congifuration is according to the gradle manual, for more information, check the `Table 33.5. Default Values for Maven POM generation` in [gradle website](https://docs.gradle.org/current/userguide/userguide_single.html?_ga=1.15647102.998671448.1486064310#sec:maven_pom_generation).

###Configure the gradle file in Android Studio
#####1. Create a .gradle file
Create a .gradle file(in this case,it's config.gradle) under the library module(at the same level as build.gradle), copy the content from [config.gradle](https://github.com/liwangjing/BitbucketAsMavenRepo/blob/master/config.gradle).<br>
Or you can download the config.gradle file from [here](https://github.com/liwangjing/BitbucketAsMavenRepo), put it in the library module.

#####2. Create a gradle.properties file to configure the gradle file
Create a new file named "gradle.properties", it's under the same directory as config.gradle file. you copy the content from [gradle.properties](https://github.com/liwangjing/BitbucketAsMavenRepo/blob/master/gradle.properties)<br>
Or you can download the gradle.properties file from [here](https://github.com/liwangjing/BitbucketAsMavenRepo)

#####3. Configure the gradle.properties in the project module
```bash
USERNAME=Bitbucket_username
PASSWORD=Bitbucket_password
```
*NOTICE:* **DO NOT upload this file to the remote repository for the sake of security of your account**

#####4. Apply the .gradle file to the build.gradle file.
In the `build.gradle(library)`, add this code as the second line.
```bash
apply from: 'config.gradle' 
```

#####5. Upload the aar file to your Bitbucket repository
In the Android Studio interface, find the `gradle` in the right tool bar,  expand it and click the gradle project of the android library, click `'execute gradle task'` icon(same icon as gradle), it will pop up a dialog, and in the `Command line`, enter:
```bash
assembleRelease uploadArchives
```
Or you can clean it first, with this command, after that, uploadArchives to the Bitbucket repository.
```bash
clean
```
Click `OK`, it will run the `build.gradle` and the `config.gradle` file, upload the aar file to the Bitbucket repository.


###Import the aar from a maven repository
Congrates! You know how to deploy aar file to a Bitbucket private repositoy, what if you want to use the library for another project?<br>
Now we are going to explore how to fectch a well encapsulated Android Library from a maven repository, and use it for another Android application development.

#####1. prepare
Create a new Android project that will depend on the aar file.

#####2. declare the repository in the `build.gradle(app)`
```bash
repositories {
    mavenCentral() // this could be JCenter().
    maven {
        credentials {
            username USERNAME
            password PASSWORD
        }
        authentication {
            basic(BasicAuthentication)
        }
        url "https://api.bitbucket.org/1.0/repositories/COMPANY/REPO_NAME/raw/releases" // you need to configure the COMPANY & REPO_NAME.
    }
}
```
`credentials`: as the repository is private, you need to provide your Bitbucket username and password in order to fetch the artifact.<br>
`url`: url for your maven repository, REMEMBER to modify the COMPANY and REPO_NAME.<br>
`COMPANY`: the owner name of the repository.<br>
`REPO_NAME`: the name of the repository.<br>

#####3.declare the dependency to the aar file in `build.gradle(app)`
add compile path to dependencies{} in the gradle file, path is ARTIFACT_PACKAGE:ARTIFACT_NAME:ARTIFACT_VERSION, for example:
```bash
compile 'com.example.demo:lib:1.0.0'
```

#####4. modify the gradle.properties
In the ```gradle.properties```, add your Bitbucket account info:
```bash
USERNAME=Bitbucket_username
PASSWORD=Bitbucket_password
```
*NOTICE:* **DO NOT upload this file to the remote repository for the sake of security of your account**

###Special thanks:
#####During the exploration of using Bitbucket repository to host artifact, the following articles helped a lot:<br>
[Git as a secure private maven repository](http://jeroenmols.com/blog/2016/02/05/wagongit/)<br>
[how-to-publish-an-android-library-as-a-maven-artifact-on-bitbucket](http://stackoverflow.com/questions/33812099/how-to-publish-an-android-library-as-a-maven-artifact-on-bitbucket)<br>
[how to write README with markdown](http://georgeosddev.github.io/markdown-edit/)


####**Thank you for reading this article, feel free to contact me if you have encountered any issue. ^^**
