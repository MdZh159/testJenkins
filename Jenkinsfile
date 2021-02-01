#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    docker.image('jhipster/jhipster:v6.10.1').inside('-u root -e MAVEN_OPTS="-Duser.home=./"') {
        stage('check java') {
            sh "java -version"
        }

        stage('clean') {
            sh "chmod +x mvnw"
            sh "./mvnw -ntp clean -P-webpack"
        }
        stage('nohttp') {
            sh "./mvnw -ntp checkstyle:check"
        }

        stage('install tools') {
            sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:install-node-and-npm -DnodeVersion=v12.16.1 -DnpmVersion=6.14.5"
        }

        stage('npm install') {
            sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm"
        }

        stage('backend tests') {
            try {
                sh "./mvnw -ntp verify -P-webpack -DskipTests"
            } catch(err) {
                throw err
            } finally {
                junit allowEmptyResults: true, testResults: '**/test-results/*.xml'
            }
        }

        stage('frontend tests') {
            try {
               sh "npm install"
               sh "npm test"
            } catch(err) {
                throw err
            } finally {
                junit allowEmptyResults: true, testResults: '**/test-results/*.xml'
            }
        }

        stage('packaging') {
            sh "./mvnw package -DskipTests"
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }
    }
}
