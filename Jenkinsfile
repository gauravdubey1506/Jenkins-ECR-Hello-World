pipeline {
 agent any
 environment {
 mvn = '/usr/bin/mvn'
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
   stage('Maven Clean Install') {
    when {
		anyOf {
			branch 'feature'; branch 'develop'; branch 'main'
		}
    } 	
    steps {
       sh 'mvn clean install'
    }
   }
   stage('Build and push Docker Image') {
	when {
		anyOf {
			branch 'develop'; branch 'main'
	   }
    }
    steps {
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
   }
   stage('Deploy on EKS cluster') {
	when {
		anyOf {
			branch 'develop'; branch 'master'
	   }
    }
     steps {
            sh '''
            #!/bin/bash
            NOW=`date +"%Y%m%d"`
			BUILD="$BUILD_NUMBER"
			TAG=$NOW.$BUILD
			echo $TAG
            helm_path=/usr/local/bin
			RELEASE="hello-world"
			IMAGE="dev-promotion-assessment"
			NAMESPACE="default"
            aws eks --region us-east-1 update-kubeconfig --name Terraform-EKS-Cluster
            $helm_path/helm upgrade --install hello-world hello-world --set image.tag=$TAG
            '''
     }
   }
 }
 post {
    always {
 	echo 'clean up current workspace'
	deleteDir()
  }
 }
}
