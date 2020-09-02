pipeline{

agent {
    kubernetes {
      label "docker-img-builder-${UUID.randomUUID().toString()}"
      yamlFile "pipelines/podTemplate.yaml"
    }
  }

  environment {
      registry_uri = "670053613788.dkr.ecr.us-east-1.amazonaws.com"
      ecr_repo = "sample"
  }

stage ('Docker build') {
        steps {
            script {
                container(name: 'utilities') {
                    sh """
                        wget https://tomcat.apache.org/tomcat-8.0-doc/appdev/sample/sample.war 
                        docker build --no-cache -f ${WORKSPACE}/Dockerfile -t ${env.registry_uri}/${env.ecr_repo}:${BRANCH_NAME} .
                    """
                }
            }
        }
    }
    stage ('Docker publish') {
        when { branch "master" }
        steps {
            script {
                container(name: 'utilities') {
                    sh """
                         aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${env.registry_uri}
                         docker tag ${env.registry_uri}/${env.ecr_repo}:${BRANCH_NAME} ${env.registry_uri}/${env.ecr_repo}:latest
                         docker push ${env.registry_uri}/${env.ecr_repo}:latest
                    """
                }
            }
        }
    }
    stage ('Release application') {
        when { branch "master" }
        steps {
            script {
                container(name: 'utilities') {
                    dir ('helm-charts/sampleapp'){
                        sh """
                            helm init 
                            helm upgrade sampleapp-release --namespace samplens --install ."
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}