pipeline { 
    agent any 
    
    environment { 
        PATH = "/usr/bin:$PATH" 
        tag = "1.0" 
        dockerHubUser = "imrandevops12"
        //dockerPassword = credentials('Raftaar@1994') 
        containerName = "insure-me" 
        httpPort = "8081" 
    } 
    
    stages { 
        stage("code clone"){ 
            steps { 
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Imrankh4n/asi-insurance.git']]) 
            } 
        } 
        stage("Maven build"){ 
            steps { 
                sh "mvn clean install -DskipTests" 
            } 
        } 
        stage("Build Docker Image"){ 
            steps{ 
                sh "docker build -t ${dockerHubUser}/insure-me:${tag} ." 
            } 
        } 
        stage("Push image to DockerHub"){ 
            steps{ 
                withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) { 
                   sh "docker login -u $dockerUser -p $dockerPassword"
		   sh "docker push $dockerHubUser/$containerName:$tag"
                } 
            } 
        } 
        stage("Docker container deployment"){ 
            steps { 
                sh """ 
                    docker rm $containerName -f
                    docker pull $dockerHubUser/$containerName:$tag
                    docker run -d --rm -p $httpPort:$httpPort -v /var/run/docker.sock:/var/run/docker.sock --name $containerName $dockerHubUser/$containerName:$tag
                    echo "Application started on port: ${httpPort} (http)"
                """
            } 
        } 
    } 
}
