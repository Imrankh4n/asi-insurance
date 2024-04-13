pipeline { 
	agent any 
	environment { 
		PATH = "/usr/bin:$PATH" 
		tag = "1.0" 
		dockerHubUser="imrankha4n" 
		DOCKER_USERNAME = credentials('imrankha4n')
        	DOCKER_PASSWORD = credentials('Raftaar@1996')
		containerName="insure-me"
		httpPort="8081" 
		} 
	stages { 
		stage("code clone"){ 
			steps { 
				checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Imrankh4n/asi-insurance.git']])				} 
			} 
		stage("Maven build"){ 
			steps { 
				sh "mvn clean install -DskipTests" 
			} 
		} 
		stage('Build and push image') {
            		steps {
                		script {
		                    // Login to Docker registry
		                    echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
		
		                    // Build and push Docker image
		                    sh 'docker build -t my-image .'
		                    sh 'docker push my-image'
                		     }
			}
		}
		stage("Build Docker Image"){ 
			steps{ 
				sh "docker build -t ${dockerHubUser}/insure-me:${tag} ." 
			} 
		} 
		stage("push image to dockerhub"){ 
			steps{ 
				withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', passwordVariable: 'dockerPassword', usernameVariable: 'dockerHubUser')]) { 
					sh "docker login -u $dockerHubUser -p $dockerPassword && sh docker push $dockerHubUser/$containerName:$tag" 
					} 
				}
		} 

		stage("Docker container deployment"){ 
			steps { 
				sh "docker rm $containerName -f sh docker pull $dockerHubUser/$containerName:$tag && docker run -d --rm -p $httpPort:$httpPort0 -v /var/run/docker.sock:/var/run/docker.sock admin/jenkins --name $containerName $dockerHubUser/$containerName:$tag && echo \"Application started on port: ${httpPort} (http)\""
				} 
		} 
	} 
}
node{
    
    def mavenHome, mavenCMD, docker, tag, dockerHubUser, containerName, httpPort = ""
    
    stage('Prepare Environment'){
        echo 'Initialize Environment'
        mavenHome = tool name: 'maven' , type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        tag="1.0"
	dockerHubUser="imrankha4n"
	containerName="insure-me"
	httpPort="8081"
    }
    
    stage('Code Checkout'){
        try{
            checkout scm
        }
        catch(Exception e){
            echo 'Exception occured in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            //emailext body: '''Dear All,
            //The Jenkins job ${JOB_NAME} has been failed. Request you to please have a look at it immediately by clicking on the below link. 
            //${BUILD_URL}''', subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} is failed', to: 'jenkins@gmail.com'
        }
    }
    
    stage('Maven Build'){
        sh "${mavenCMD} clean package"        
    }
    
    stage('Publish Test Reports'){
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    }
    
    stage('Docker Image Build'){
        echo 'Creating Docker image'
        sh "docker build -t $dockerHubUser/$containerName:$tag --pull --no-cache ."
    }
	
    stage('Docker Image Scan'){
        echo 'Scanning Docker image for vulnerbilities'
        sh "docker build -t ${dockerHubUser}/insure-me:${tag} ."
    }   
	
    stage('Publishing Image to DockerHub'){
        echo 'Pushing the docker image to DockerHub'
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'dockerHubUser', passwordVariable: 'dockerPassword')]) {
			sh "docker login -u $dockerHubUser -p $dockerPassword"
			sh "docker push $dockerHubUser/$containerName:$tag"
			echo "Image push complete"
        } 
    }    
	
	stage('Docker Container Deployment'){
		sh "docker rm $containerName -f"
		sh "docker pull $dockerHubUser/$containerName:$tag"
		sh "docker run -d --rm -p $httpPort:$httpPort -v /var/run/docker.sock:/var/run/docker.sock admin/jenkins --name $containerName $dockerHubUser/$containerName:$tag"
		echo "Application started on port: ${httpPort} (http)"
	}
}
