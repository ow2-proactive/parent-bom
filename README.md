# parent-bom
parent bom project managing dependencies version

# How to use the plugin in other micro-service projects
## dependency management plugin definition in the micro-service project's build.gradle file

1. Declaire the dependency management plugin in the buildscript block as following

```
buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath 'org.springframework.boot:spring-boot-gradle-plugin:1.3.3.RELEASE'
        classpath "com.diffplug.gradle.spotless:spotless:2.4.0"
        classpath "org.ow2.proactive:coding-rules:1.0.0"
        ... ...
        classpath "io.spring.gradle:dependency-management-plugin:0.3.0.RELEASE"
    }
}
```

2. Apply the plugin
```
apply plugin: "io.spring.dependency-management"
```

3. Import this parent-bom pom file into your micro-service project
```
dependencyManagement {
    imports {
        mavenBom "org.ow2.proactive:parent-bom:${version}"
    }
}
```

4. Remove the versions from the dependencies block, except the proactive dependencies

For example,

```
dependencies {
    providedCompile "org.ow2.proactive_grid_cloud_portal:rest-client:$version"
    providedCompile "org.ow2.proactive:common-client:$version"

    compile 'commons-fileupload:commons-fileupload'

    compile "org.apache.logging.log4j:log4j-web"

    compile 'org.springframework.boot:spring-boot-starter-data-jpa'

    compile 'org.hibernate:hibernate-entitymanager'
    compile 'org.hibernate:hibernate-core'
    
    ...
    ...
}
```

**Note: the exclusions must be done in parent-bom pom file, not in the dependencies block in the build.gradle file**

## How to add new dependencies version in parent-bom project
All dependencies version must be managed in this parent-bom project. When a new dependency version is needed, the parent-bom's pom.xml file needs to be updated to add this new dependency's version.

1. Verify the same dependency does not exist already by checking in the pom.xml file the ```properties``` block, which all dependencies versions are declared there.
```
<properties>
        <springboot.version>1.3.3.RELEASE</springboot.version>
        <commonsfileupload.version>1.3.1</commonsfileupload.version>
        <log4jweb.version>2.7</log4jweb.version>
        <junit.version>4.11</junit.version>
        ...
        ...
</properties>        
```

2. Create a new property for the new dependency's version in the ```properties``` block

3. Add a new ```dependency``` sub-block in the ```dependencyManangement``` block, like following

```
 <dependencyManagement>
        <dependencies>
            <!-- spring boot -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-data-jpa</artifactId>
                <version>${springboot.version}</version>
                <exclusions>
                    <exclusion>
                        <groupId>org.hibernate</groupId>
                        <artifactId>hibernate-entitymanager</artifactId>
                    </exclusion>
                    <exclusion>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-starter-validation</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
            ...
            ...
</dependencyManagement>            
```
as you can see in the sample, the exlusions are mamaged here in the block too, so that when this dependency is imported in the micro-service's build.gradle, the exclusions are already done, do not repeat the exclusions declaration again in the build.gradle.

4. Build and commit the parent-bom project, the nightly-release will generate and upload the artifact to the proactive repository, after which the new dependency can be imported into the micro-service project then.

## Common issues

- The Error "detached configuration" means that it cannot solve the dependencies.

- When parent-bom will be updated, you will have to launch the parent-bom-release jenkins job to update the version on the nexus repository.

- Be sure that activeeon repository is in your buildscript repositories in build.gradle.