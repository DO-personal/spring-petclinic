node {
    projectName="mypetclinic";
    dockerRegistryOrg="roxor007"
    environment { 
        softwareVersion()
    }
    stage('Code') {
        stage('clean') {
            sh """ #!/bin/bash
                rm -rf spring-petclinic
            """
        }
        stage('clone') {
            git branch: 'main', url: 'https://github.com/do-personal/spring-petclinic.git'
        } // stage: clone
        stage('compile') {
            sh """ #!/bin/bash
                ./mvnw clean install -DskipTests=true
            """
        } // stage: compile
    } // stage: code
    stage('Tests') {
        parallel unitTest: {
            stage ("unitTest") {
                timeout(time: 10, unit: 'MINUTES') {
                    sh """ #!/bin/bash
                        ./mvnw test surefire-report:report

                        echo 'surefire report generated'
                    """
                } // timeout
            } // stage: unittest
        }, checkstyle: {
            stage ("checkStyle") {
                timeout(time: 2, unit: 'MINUTES') {
                    sh """ #!/bin/bash
                        ./mvnw validate
                    """
                } // timeout
            } // stage: validate
        }, codeCoverage: {
            stage ("codeCoverage") {
                timeout(time: 2, unit: 'MINUTES') {
                    sh """ #!/bin/bash
                        ./mvnw jacoco:report
                                    
                        echo 'Jacoco report generated in http://localhost:8080/job/${projectName}/${env.BUILD_ID}/execution/node/3/ws/target/site/jacoco/index.html'
                    """
                } // timeout
            } // stage: Jacoo
        } // parallel
    } // stage: tests
    stage ("Container") {
        stage('build') {
            sh """ #!/bin/bash
                docker image build -f Dockerfile -t ${projectName}:${env.BUILD_ID} .
            """
        } // stage: build
        stage('tag') {
            parallel listContainers: {
                sh """ #!/bin/bash
                    docker container ls -a
                """
            }, listImages: {
                sh """ #!/bin/bash
                    docker image ls -a
                """
            }, tagBuildNumb: {
                    sh """ #!/bin/bash
                        docker tag ${projectName}:${env.BUILD_ID} roxor007/${projectName}:${env.BUILD_ID}
                    """
            }, tagLatest: {
                sh """ #!/bin/bash
                    docker tag ${projectName}:${env.BUILD_ID} roxor007/${projectName}:latest
                """
            }
        } // stage: tag
        stage('publish') {
            withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_REGISTRY_PWD', usernameVariable: 'DOCKER_REGISTRY_USER')]) {
                sh """ #!/bin/bash
                    docker login -u $DOCKER_REGISTRY_USER -p $DOCKER_REGISTRY_PWD
                    echo 'Login success...'

                    docker push roxor007/${projectName}:${env.BUILD_ID}
                    docker push roxor007/${projectName}:latest

                    docker logout
                    echo 'Logut...'
                """
            } // withCredentials: dockerhub
        } // stage: push
    } // stage: docker
}
def softwareVersion() {
    sh """ #!/bin/bash
        java -version
        ./mvnw -version
        docker version
	./mvnw package
	java -jar target/*.jar
    """
}
