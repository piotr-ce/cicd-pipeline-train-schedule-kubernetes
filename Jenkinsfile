pipeline {
    agent any
    environment {
        // this is a name of repo (it will be publicly available) that a step below will build
        // and then push
        DOCKER_IMAGE_NAME = "pio2pio/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World! Piotrek'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    // 0b9b73fd*** is jenkins docker_key-id
                    docker.withRegistry('https://registry.hub.docker.com', '0b9b73fd-8b69-457f-94bb-8c3840749dd2') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                // this is interactive part and requires you to click on the box 'Deploy toProduction?' and click 'YES'
                // choosing 'NO' it won;t deploy to K8s cluster. Also note, this Jenkins CD Kubernetes plugin is not
                // more clever than kubectl, removing an object froma a manifest and re-deploying manifest won't remove
                // the deleted object. 
                input 'Deploy to Production?'
                milestone(1)
                //implement Kubernetes deployment here
                kubernetesDeploy(
                    // kubeconfig is jenkins credentials of Kubernetes credentials
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
                
            }
        }
    }
}
