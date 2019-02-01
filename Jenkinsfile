pipeline {
    agent any
    tools { 
        maven 'maven3.6.0' 
        jdk 'java1.8.0'
    }
    stages {
        stage ('GitHub Clone') {
            steps {
                git branch: 'master', url: "https://github.com/dipankar2019/simple-java-maven-app.git"
            }
        }
        
        stage('Maven Test') {
            steps {
                sh "mvn test"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('My SonarQube Server') {
                  // requires SonarQube Scanner for Maven 3.2+
                  sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
                }
            }
        }

        stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "ARTIFACTORY_SERVER",
                    url: "http://52.10.9.231:8081/artifactory",
                    credentialsId: 'ARTIFACTORY'
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )
            }
        }

        stage ('Maven Build') {
            steps {
                rtMavenRun (
                    tool: 'maven3.6.0', // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
        }

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "ARTIFACTORY_SERVER"
                )
            }
        }
        
        stage('Deploy JAR') {
            steps {
                sh 'java -jar target/sample-app-1.0-SNAPSHOT.jar'
            }
        }
    }
}
