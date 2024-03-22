pipeline {
    agent any

    parameters {
        choice(name: 'account', choices: ['dev', 'qa', 'stage', 'prod'], description: 'Select the environment.')
        //string(name: 'commit_id', defaultValue: 'latest', description: 'provide commit id.')
    }

    environment {
        COMMITID            = "${params.commit_id}"
        dev_dh_creds        = 'docker-cred'
        dev_registry        = 'asadis7171/dev'
        dev_registry_endpoint = 'https://' + "${env.registryURI}" + "${env.dev_registry}"
        dev_image           = "${env.dev_registry}" + ":$GIT_COMMIT"
        prod_dh_creds       = 'dh_cred_prod'
        prod_image          = "${env.registryURI}" + "${env.prod_registry}" + ':' + "${env.COMMITID}"
        prod_registry       = 'asadis7171/prod'
        prod_registry_endpoint = 'https://' + "${env.registryURI}" + "${env.prod_registry}"
        qa_dh_creds         = 'dh_cred_qa'
        qa_image            = "${env.registryURI}" + "${env.qa_registry}" + ':' + "${env.COMMITID}"
        qa_registry         = 'asadis7171/qa'
        qa_registry_endpoint = 'https://' + "${env.registryURI}" + "${env.qa_registry}"
        registryURI         = 'registry.hub.docker.com/'
        stage_dh_creds      = 'dh_cred_stage'
        stage_image         = "${env.registryURI}" + "${env.stage_registry}" + ':' + "${env.COMMITID}"
        stage_registry      = 'asadis7171/stage'
        stage_registry_endpoint = 'https://' + "${env.registryURI}" + "${env.stage_registry}"
    }

    stages {

        stage('Docker Image Build IN Dev') {
            when {
                expression {
                    params.account == 'dev'
                }
            }

            steps {
                echo "Building Docker Image Logging in to Docker Hub & Pushing the Image"
                script {
                    def app = docker.build(dev_image)
                    docker.withRegistry(dev_registry_endpoint, dev_dh_creds) {
                        app.push()
                    }
                }
            }

            /*post {
                always {
                    sh 'echo Cleaning docker Images from Jenkins.'
                    sh "docker rmi ${env.dev_image}"
                }
            }*/
        }

        stage('Pull Tag push to QA') {
            when {
                expression {
                    params.account == 'qa'
                }
            }

            steps {
                script {
                    docker.withRegistry(dev_registry_endpoint, dev_dh_creds) {
                        docker.image(dev_image).pull()
                    }

                    sh 'echo Image pulled from DEV'
                    sh 'echo Tagging Docker image from Dev to QA'
                    sh "docker tag ${env.dev_image} ${env.qa_image}"

                    docker.withRegistry(qa_registry_endpoint, qa_dh_creds) {
                        docker.image(env.qa_image).push()
                    }

                    sh 'echo Image pushed'
                }
            }

           /* post {
                always {
                    sh 'echo Cleaning docker Images from Jenkins.'
                    sh "docker rmi ${env.dev_image}"
                    sh "docker rmi ${env.qa_image}"
                }
            }*/
        }

        stage('Pull Tag push to stage') {
            when {
                expression {
                    params.account == 'stage'
                }
            }

            steps {
                script {
                    docker.withRegistry(qa_registry_endpoint, qa_dh_creds) {
                        docker.image(qa_image).pull()
                    }
                    sh 'echo Image pulled from QA'
                    sh 'echo Tagging Docker image from QA to Stage'
                    sh "docker tag ${env.qa_image} ${env.stage_image}"

                    docker.withRegistry(stage_registry_endpoint, stage_dh_creds) {
                        docker.image(env.stage_image).push()
                    }

                    sh 'echo Image Pushed to STAGE'
                }
            }

          /*  post {
                always {
                    sh 'echo Cleaning docker Images from Jenkins.'
                    sh "docker rmi ${env.qa_image}"
                    sh "docker rmi ${env.stage_image}"
                }
            } */
        }

        stage('Pull Tag push to Prod') {
            when {
                expression {
                    params.account == 'prod'
                }
            }

            steps {
                script {
                    docker.withRegistry(stage_registry_endpoint, stage_dh_creds) {
                        docker.image(stage_image).pull()
                    }

                    sh 'echo Image pulled from QA'
                    sh 'echo Tagging Docker image from stage to prod'
                    sh "docker tag ${env.stage_image} ${env.prod_image}"

                    docker.withRegistry(prod_registry_endpoint, prod_dh_creds) {
                        docker.image(env.prod_image).push()
                    }

                    sh 'echo Image Pushed to Prod'
                }
            }

            /*post {
                always {
                    sh 'echo Cleaning docker Images from Jenkins.'
                    sh "docker rmi ${env.stage_image}"
                    sh "docker rmi ${env.prod_image}"
                }
            }*/
        }

        stage('Deploy to Dev K8S cluster ') {
            when {
                expression {
                    params.account == 'dev' //&& env.BRANCH_NAME == 'myBranch'
                }
            }
            environment {
                        kube_config  = "${params.account}" + '-kube-config'
                        account_name = "${params.account}"
                    }

            steps {
                script {
                    echo 'Deploying on Dev K8S Cluster.'
                    withKubeConfig(credentialsId: "${env.account}-kube-config", restrictKubeConfigAccess: true) {
                        sh "sed -i -e 's/{{ACCOUNT}}/${env.account_name}/g' -e 's/{{COMMITID}}/${GIT_COMMIT}/g' KUBE/deployment.yaml"
                        sh 'echo deployment.yaml file after replace with sed'
                        sh 'cat KUBE/deployment.yaml'
                        sh 'kubectl apply -f KUBE/deployment.yaml'
                        sh 'kubectl apply -f KUBE/service.yaml'
                    }
                }
            }
        }

        stage('Deploy to QA K8S cluster ') {
            when {
                expression {
                    params.account == 'qa' //&& env.BRANCH_NAME == 'myBranch'
                }
            }
            environment {
                        kube_config  = "${params.account}" + '-kube-config'
                        account_name = "${params.account}"
                    }

            steps {
                script {
                    echo 'Deploying on Dev K8S Cluster.'
                    withKubeConfig(credentialsId: "${env.account}-kube-config", restrictKubeConfigAccess: true) {
                        sh "sed -i -e 's/{{ACCOUNT}}/${env.account_name}/g' -e 's/{{COMMITID}}/${GIT_COMMIT}/g' KUBE/deployment.yaml"
                        sh 'echo deployment.yaml file after replace with sed'
                        sh 'cat KUBE/deployment.yaml'
                        sh 'kubectl apply -f KUBE/deployment.yaml'
                        sh 'kubectl apply -f KUBE/service.yaml'
                    }
                }
            }
        }

        stage('Deploy to STAGE K8S cluster ') {
            when {
                expression {
                    params.account == 'stage' //&& env.BRANCH_NAME == 'myBranch'
                }
            }
            environment {
                        kube_config  = "${params.account}" + '-kube-config'
                        account_name = "${params.account}"
                    }

            steps {
                script {
                    echo 'Deploying on Dev K8S Cluster.'
                    withKubeConfig(credentialsId: "${env.account}-kube-config", restrictKubeConfigAccess: true) {
                        sh "sed -i -e 's/{{ACCOUNT}}/${env.account_name}/g' -e 's/{{COMMITID}}/${GIT_COMMIT}/g' KUBE/deployment.yaml"
                        sh 'echo deployment.yaml file after replace with sed'
                        sh 'cat KUBE/deployment.yaml'
                        sh 'kubectl apply -f KUBE/deployment.yaml'
                        sh 'kubectl apply -f KUBE/service.yaml'
                    }
                }
            }
        }

        stage('Deploy to QA K8S cluster ') {
            when {
                expression {
                    params.account == 'prod' //&& env.BRANCH_NAME == 'myBranch'
                }
            }
            environment {
                        kube_config  = "${params.account}" + '-kube-config'
                        account_name = "${params.account}"
                    }

            steps {
                script {
                    echo 'Deploying on Dev K8S Cluster.'
                    withKubeConfig(credentialsId: "${env.account}-kube-config", restrictKubeConfigAccess: true) {
                        sh "sed -i -e 's/{{ACCOUNT}}/${env.account_name}/g' -e 's/{{COMMITID}}/${GIT_COMMIT}/g' KUBE/deployment.yaml"
                        sh 'echo deployment.yaml file after replace with sed'
                        sh 'cat KUBE/deployment.yaml'
                        sh 'kubectl apply -f KUBE/deployment.yaml'
                        sh 'kubectl apply -f KUBE/service.yaml'
                    }
                }
            }
        }
    }



    post {
        always {
            echo 'Deleting Workspace from shared Lib'
            //emailext(body: '${DEFAULT_CONTENT}', subject: '${DEFAULT_SUBJECT}', to: '$DEFAULT_RECIPIENTS')
            deleteDir() /* clean up our workspace */
        }
    }
}
