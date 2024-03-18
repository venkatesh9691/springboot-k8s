pipeline {
    agent any
    
    tools{
        jdk'jdk8'
        maven'maven3'
    }

    stages {
        stage('Cloning From GIT') {
            steps {
                git branch: 'main', url: 'https://github.com/venkatesh9691/springboot-k8s.git'
            }
        }
        
        stage("Maven Compile"){
            steps{
                sh'mvn compile'
            }
        }
        stage("Maven Testing"){
            steps{
                sh'mvn test -DskipTests=true'
            }
        }
        stage("OWASP Dependency Checking"){
            steps{
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
        stage("file system scanning"){
            steps{
	            sh'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage("Maven Build"){
            steps{
                sh'mvn clean package -DskipTests=true'
            }
        }
        stage("Nexus Artifact Uploading"){
            steps{
                nexusArtifactUploader artifacts: [
                    [
                        artifactId: 'spring-boot-starter-parent',
                        classifier: '',
                        file: 'target/springboot-crud-k8s.jar',
                        type: 'jar'
                    ]
                ],
                credentialsId: 'Nexus',
                groupId: 'com.javatechie',
                nexusUrl: '51.20.181.109:8081',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: 'venkat',
                version: '0.0.1-SNAPSHOT'
            }
        }
        stage('Build Docker Image'){
            steps{
                sh'docker stop s4'
                sh 'docker rm s4'
                sh 'docker build -t venkatesh9691/venkatesh-projects-new .'
                sh 'docker build -t tomcat:${BUILD_NUMBER} .'
                sh 'docker run -itd --name s4 -p 3800:8080 tomcat:${BUILD_NUMBER}'
            }
        }
        stage("Docker image scaning"){
            steps{
                sh'trivy image --format table -o trivy-fs-report.html venkatesh9691/venkatesh-projects-new'
            }
        }
        stage("Push To Docker Hub"){
            steps{
                withCredentials([string(credentialsId: 'DOCKER_HUB_CREDENTIALS', variable: 'DOCKER_HUB_CREDENTIALS')]) {
                    sh'docker login -u ${DOCKER_HUB_CREDENTIALS} -p ${DOCKER_HUB_CREDENTIALS}'
                }
                sh 'docker push venkatesh9691/venkatesh-projects-new'
            }
        }
        /*stage("sonarqube analysis"){
            steps{
                withSonarQubeEnv( 'sonarqube') {
                    sh'mvn clean sonar:sonar package'
                }
            }
        }*/
	    stage('SonarQube Analysis') {
		    steps{
        def mvnHome =  tool name: 'maven-3', type: 'maven'
        withSonarQubeEnv('sonaqube') { 
          sh "${mvnHome}/bin/mvn sonar:sonar"
	}
        }
    }
    }
}
