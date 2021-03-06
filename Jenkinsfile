podTemplate(
    containers: [
        containerTemplate(
            args: 'cat', 
            command: '/bin/sh -c', 
            image: 'docker', 
            livenessProbe: containerLivenessProbe(
                execArgs: '', 
                failureThreshold: 0, 
                initialDelaySeconds: 0, 
                periodSeconds: 0, 
                successThreshold: 0, 
                timeoutSeconds: 0
                ), 
            name: 'docker-container',  
            ttyEnabled: true, 
            workingDir: '/home/jenkins/agent'
        ),
        containerTemplate(
            args: 'cat', 
            command: '/bin/sh -c', 
            image: 'lopesall/kubectl:v.01', 
            livenessProbe: containerLivenessProbe(
                execArgs: '', 
                failureThreshold: 0, 
                initialDelaySeconds: 0, 
                periodSeconds: 0, 
                successThreshold: 0, 
                timeoutSeconds: 0, 
                ),
            name: 'deploy-container',  
            ttyEnabled: true, 
            workingDir: '/home/jenkins/agent'
        )],  
      label: 'codemaster', 
      name: 'codemaster', 
      namespace: 'jenkins',    
      volumes: [
        hostPathVolume(
          hostPath: '/run/docker.sock', 
          mountPath: '/run/docker.sock'
            )         
        ]
    ) 
    {
    //Start PIPELINE
    node('codemaster') {
      def REPOS
      def IMAGE_NAME = "frontend"
      def ENVIRONMENT
      def IMAGE_VERSION
      def KUBE_NAMESPACE
      def GIT_BRANCH
         
      stage('Checkout') {
        echo "Iniciando Pipeline"  //Clone do Repositório"
        REPOS=checkout([$class: 'GitSCM', 
                           branches: [[name: '*/master'], [name: '*/develop']],
                           extensions: [], 
                           userRemoteConfigs: [[credentialsId: 'Gitlab', 
                                                url: 'git@gitlab.codemaster.com.br:alexandre/frontend.git']]
                          ])
        GIT_BRANCH = REPOS.GIT_BRANCH
        if (GIT_BRANCH.equals("origin/develop")){
          KUBE_NAMESPACE = "staging"
          ENVIRONMENT = "staging"
        } else if(GIT_BRANCH.equals("origin/master")){
          KUBE_NAMESPACE = "production"
          ENVIRONMENT = "production"
        } 
         //else {
          //echo "Não existe pipeline para a Branch ${GIT_BRANCH}."
          //exit 0
        //}         
        //sh label: '',
           // script: "git checkout ${GIT_BRANCH}" //REPOS = git credentialsId: 'gitlab', url: 'git@gitlab.codemaster.com.br:codemaster/frontend.git'
        IMAGE_VERSION = sh returnStdout: true, script: 'sh read-package-version.sh'
        IMAGE_VERSION = IMAGE_VERSION.trim()
        
        echo "BRANCH: ${GIT_BRANCH}"
        echo "AMBIENTE: ${ENVIRONMENT}"
        sh 'sleep 2'
        input message: "Seguir com a Pipeline?"

      }
      stage('Package') {
        container('docker-container') {
          echo "Iniciando empacotamento com Docker"
           withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USER')]) {
          sh label: '', 
            script: "docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}"
          sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_VERSION} . --build-arg NPM_ENV=${ENVIRONMENT}"
          sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_VERSION}"
          }
        }
      }
      stage('Deploy') {
        container('deploy-container') {
          repos = "git credentialsId: 'Gitlab', url: 'git@gitlab.codemaster.com.br:alexandre/frontend.git', branch: '${GIT_BRANCH}'"
          //sh label: '',
          //  script: "git checkout ${GIT_BRANCH}"
          sh returnStdout: true, script: 'sh version.sh'

          //sh "git checkout ${GIT_BRANCH}"
          sh 'cat frontend-deployment.yaml'
          sh 'sleep 1'
          sh "kubectl apply -f frontend-deployment.yaml -n ${KUBE_NAMESPACE}"
        }
      }
    }  // end pipeline
}