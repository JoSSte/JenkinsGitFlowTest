#!groovy​
@Library('JoSSteJenkinsGlobalLibraries')
import com.stevnsvig.jenkins.release.Release
def rls = new Release()

properties([[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', numToKeepStr: '10']]])

stage('build') {
    node {
        checkout scm
        def v = version()
        currentBuild.displayName = "${env.BRANCH_NAME}-${v}-${env.BUILD_NUMBER}"
    }
}

stage('build docker image') {
    node {
        echo "docker being built"
    }
}

def branch_type = rls.get_branch_type "${env.BRANCH_NAME}"
def branch_deployment_environment = rls.get_branch_deployment_environment branch_type

if (branch_deployment_environment) {
    stage('deploy') {
        if (branch_deployment_environment == "prod") {
            timeout(time: 1, unit: 'DAYS') {
                input "Deploy to ${branch_deployment_environment} ?"
            }
        }
        node {
            deploy "Deploying to ${branch_deployment_environment}"
        }
    }

    if (branch_deployment_environment != "prod") {
        stage('integration tests') {
            node {
                echo "Running integration tests in ${branch_deployment_environment}"
                //TODO do the actual tests
            }
        }
    }
}

if (branch_type == "dev") {
    stage('start release') {
        timeout(time: 1, unit: 'HOURS') {
            input "Do you want to start a release?"
        }
        node {
            echo "starting release"
        }
    }
}

if (branch_type == "release") {
    stage('finish release') {
        timeout(time: 1, unit: 'HOURS') {
            input "Is the release finished?"
        }
        node {
            echo "do cleanup after the release"
        }
    }
}

if (branch_type == "hotfix") {
    stage('finish hotfix') {
        timeout(time: 1, unit: 'HOURS') {
            input "Is the hotfix finished?"
        }
        node {
            echo " the hotfix is done - merge?"
        }
    }
}

// Utility functions

//def mvn(String goals) {
//    def mvnHome = tool "Maven-3.2.3"
//    def javaHome = tool "JDK1.8.0_102"
//
//    withEnv(["JAVA_HOME=${javaHome}", "PATH+MAVEN=${mvnHome}/bin"]) {
//        sh "mvn -B ${goals}"
//    }
//}

def version() {
    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
    return matcher ? matcher[0][1] : null
}
