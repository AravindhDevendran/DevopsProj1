pipeline {
agent any
 environment {
  DOCKER = credentials ('dockerhub')
registry = "aravindhdeva5/devopsproj2" 
        registryCredential = 'dockerhub'
 }
 stages {
     stage('Build') {
            steps {
                echo 'Running build automation'
		sh 'chmod 700 *'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
  stage ('create docker image') {
   steps {
    sh 'sudo docker build -t aravindhdeva5/devopsproj2:1111.$BUILD_NUMBER .'
   }
  }
  stage ('upload image to docker registry') {
   steps {
     sh '''
           sudo docker login --username $DOCKER_USR --password $DOCKER_PSW
           sudo docker push aravindhdeva5/devopsproj2:1111.$BUILD_NUMBER
        '''
   }   
  }
  stage ('Run Docker Container and global service') {
   steps {
    sh '''
        sudo docker run -d --name DevopsDockerContainer aravindhdeva5/devopsproj2:1111.$BUILD_NUMBER
	#sudo docker swarm leave -f
	sudo docker swarm init
	sudo docker service create --name devopsproj1 --mode global -d -p 9000:80 aravindhdeva5/devopsproj2:1111.$BUILD_NUMBER
	'''
   }
  }
     
     stage('Deploy') {
            steps {
                sh'''
		sudo kubeadm init --ignore-preflight-errors=Port-6443,Port-10259,Port-10257,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Port-2379,Port-2380,DirAvailable--var-lib-etcd
		sudo kubectl apply -f train-schedule-kube-canary.yml
		sudo kubectl get deployments.apps
		'''
            }
        }
        stage('DeployToProduction') {
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
 }
}
}
