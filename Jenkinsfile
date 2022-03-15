pipeline {
 agent any
 environment {
 mvn = '/usr/share/maven'
 }
 stages {
   stage('Get Current Branch') {
    steps {
		sh "echo Current Branch Name: ${env.BRANCH_NAME}"
     }
   }
   stage('Unit Test') {
    steps {
        sh 'mvn test'
     }
   }
   stage('Build and push Docker Image') {
    steps {
	    withAWS(credentials: '348dc31c-3c36-4e9a-84e1-8a3ad437b866', region: 'us-east-1') {
            sh '''
			 #!/bin/bash
			 IMAGE="dev-promotion-assessment"
			 NOW=`date +"%Y%m%d"`
			 BUILD="$BUILD_NUMBER"
			 TAG=$NOW.$BUILD
			 ECR_LOGINSERVER="186319575019.dkr.ecr.us-east-1.amazonaws.com"
			 aws ecr get-login-password | docker login --username AWS --password-stdin 186319575019.dkr.ecr.us-east-1.amazonaws.com
			 docker build -t ${IMAGE}:${TAG} .
			 docker images
			 docker tag ${IMAGE}:${TAG} ${ECR_LOGINSERVER}/${IMAGE}:${TAG}
			 docker push ${ECR_LOGINSERVER}/${IMAGE}:${TAG}
			 ''' 
	    }
			sh 'ls -lrt'
     }
   }
 post {
    always {
 	echo 'clean up current workspace'
	deleteDir()
  }
 }
}
