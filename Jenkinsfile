@Library('sharedlibrary')_

pipeline {
    environment {
        ecrRegistry = "768649629888.dkr.ecr.eu-north-1.amazonaws.com"
        imageName = "${ecrRegistry}/model"
        branchName = sh(script: 'echo $BRANCH_NAME | sed "s#/#-#"', returnStdout: true).trim()
        gitCommit = "${GIT_COMMIT[0..6]}"
        dockerTag = "${branchName}-${gitCommit}-${env.BUILD_NUMBER}"
        gitRepoURL = "https://github.com/nithinvarma-git/project-movie-analyzer-model.git"
    }
    agent {
        label 'agent'
    }
    stages {
        stage('Git Checkout') {
            steps {
                gitCheckout("$gitRepoURL", "$BRANCH_NAME", "githubCred")
            }
        }

        stage('Docker Build') {
            steps {
                dockerImageBuild('$imageName', '$dockerTag')
            }
        }

        stage('Docker Push') {
            steps {
                dockerECRImagePush('$imageName', '$dockerTag', 'model', 'awsCred', 'eu-north-1')
            }
        }

        stage('Update Helm Values') {
            steps {
                dir('helm') {
                    sh """
                        sed -i 's|__MODEL_IMAGE_REPOSITORY__|${imageName}|g' values.yaml
                        sed -i 's|__MODEL_IMAGE_TAG__|${dockerTag}|g' values.yaml
                    """
                }
            }
        }

        stage('Kubernetes Deploy - DEV') {
            when { branch 'dev' }
            steps { kubernetesEKSHelmDeploy('$imageName', '$dockerTag', 'movie-analyzer-model', 'awsCred', 'eu-north-1', 'movie-eks', 'dev') }
        }



        stage('Kubernetes Deploy - STAGING') {
            when { branch 'staging' }
            steps { kubernetesEKSHelmDeploy('movie-analyzer-model', 'staging') }
        }

        stage('Kubernetes Deploy - PROD') {
            when { branch 'master' }
            steps { kubernetesEKSHelmDeploy('movie-analyzer-model', 'prod') }
        }
    }
}
