# jx-app-jacoco

jx-app-jacoco provides a means for transferring a Jacoco XML code coverage report 
from a [Jenkins X](https://jenkins-x.github.io/jenkins-x-website/) build to 
a `Fact` in the `PipelineActivity` custom resource.

You must have a Jenkins X cluster to install and use the jx-app-jacoco app.  If
you do not have a Jenkins X cluster and you would like to try it out, the 
[Jenkins X Google Cloud Tutorials](https://jenkins-x.io/getting-started/tutorials/) 
is a great place to start.

## Installation

Using the [jx command line tool](https://jenkins-x.io/getting-started/install/), run 
the following command.

`jx add app jx-app-jacoco --repository "http://chartmuseum.jenkins-x.io"`

NOTE: The syntax of this command is evolving and will change.

Upon successful installation, you should see jx-app-jacoco in the list of pods 
running in your cluster - it will be called `jx-app-jacoco-jx-app-jacoco`.

NOTE: The name repitition is a common pattern in helm,

`kubectl get pods`

## Usage

The current usage of jx-app-jacoco is limited to Maven projects.  You must 
configure the build section of your Maven POM file for Jacoco to generate
an XML report in addition to the, default, jacoco.exec file.

Example Maven POM file:

    <build>
      <plugins>
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
         <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.2</version>
            <executions>
               <execution>
                  <id>default-prepare-agent</id>
                  <goals>
                     <goal>prepare-agent</goal>
                  </goals>
               </execution>
               <execution>
                  <id>prepare-xml-report</id>
                  <goals>
                     <goal>report</goal>
                  </goals>
                  <phase>verify</phase>
               </execution>
            </executions>
         </plugin>
      <plugin>
     </plugins>
    </build>

NOTE: We have an open issue to not have to generate the XML report in the project.

Ensure that your Jenkinsfile includes the following command so the Jacoco XML report is 
collected.

`sh "jx step collect --pattern=target/site/jacoco/jacoco.xml --classifier=jacoco"`

Example JenkinsFile:

    pipeline {
       agent any
       stages {
          stage ('Build App' /*, compliance-check: "build-checks" */ ) {
             steps {
                sh "mvn verify"
                sh "jx step collect --pattern=target/site/jacoco/jacoco.xml --classifier=jacoco"
             }
          }
       }
    }


Jacoco code coverage facts will now be stored in the PipelineActivity CRD for each build.

`kubectl get act -o yaml <org>-<repo>-pr-<pull request number>-<build-number>`

      factType: jx.coverage
        id: 0
        measurements:
        - measurementType: percent
          measurementValue: 6
          name: Instructions-Coverage
        - measurementType: percent
          measurementValue: 7
          name: Instructions-Missed
        - measurementType: percent
          measurementValue: 13
          name: Instructions-Total
        - measurementType: percent
          measurementValue: 2
          name: Lines-Coverage
        - measurementType: percent
          measurementValue: 3
          name: Lines-Missed
        - measurementType: percent
          measurementValue: 5
          name: Lines-Total
        - measurementType: percent
          measurementValue: 2
          name: Complexity-Coverage
        - measurementType: percent
          measurementValue: 2
          name: Complexity-Missed
        - measurementType: percent
          measurementValue: 4
          name: Complexity-Total
        - measurementType: percent
          measurementValue: 2
          name: Methods-Coverage
        - measurementType: percent
          measurementValue: 2
          name: Methods-Missed
        - measurementType: percent
          measurementValue: 4
          name: Methods-Total
        - measurementType: percent
          measurementValue: 2
          name: Classes-Coverage
        - measurementType: percent
          measurementValue: 0
          name: Classes-Missed
        - measurementType: percent
          measurementValue: 2
          name: Classes-Total
