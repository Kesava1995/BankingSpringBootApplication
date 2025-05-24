node{
    
    def tag, dockerHubUser, containerName, httpPort = ""
    
    stage('Prepare Environment'){
        echo 'Initialize Environment'
        tag="3.0"
	withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
		dockerHubUser="$dockerUser"
        }
	containerName="bankingapp"
	httpPort="8989"
    }
    
    stage('Code Checkout'){
        try{
            checkout scm
        }
        catch(Exception e){
            echo 'Exception occured in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
        }
    }
    stage('Maven Build'){
        echo '--- Diagnosing Maven Build ---'
        echo '1. Current Working Directory:'
        sh 'pwd'
        echo '2. Contents of pom.xml:'
        sh 'cat pom.xml' // CRITICAL: This will show us what pom.xml Jenkins is actually using
        echo '3. Java Version on Jenkins Agent:'
        sh 'java -version' // CRITICAL: Confirm the JDK version
        echo '4. Running Maven with debug logging:'
        sh 'mvn clean package -X' // CRITICAL: This will provide verbose output
        echo '--- End Diagnosis ---'
    }
    /*stage('Maven Build'){
        sh "mvn clean package"        
    }*/
    
    stage('Docker Image Build'){
        echo 'Creating Docker image'
        sh "docker buildx build --platform linux/amd64 -t $dockerHubUser/$containerName:$tag --load ."
    }  
	
    stage('Publishing Image to DockerHub'){
        echo 'Pushing the docker image to DockerHub'
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
		sh "docker login -u $dockerUser -p $dockerPassword"
		sh "docker push $dockerUser/$containerName:$tag"
		echo "Image push complete"
        } 
    }    
	stage('Ansible Playbook Execution'){
		withCredentials([string(credentialsId: 'ssh_password', variable: 'password')]) {
			sh "export ANSIBLE_HOST_KEY_CHECKING=False && ansible-playbook -i inventory.yaml containerDeploy.yaml -e httpPort=$httpPort -e containerName=$containerName -e dockerImageTag=$dockerHubUser/$containerName:$tag -e ansible_password=$password -e key_pair_path=/var/lib/jenkins/server.pem --become" 
		}
	}
    	/*stage('Maven Build'){
        	sh "mvn clean package"        
    	}*/
}
