pipeline {
    agent any
    environment {
        BRANCH = 'master'
        HELM_STATE_GIT_REPO = 'github.com/infracloudio/app-mono-helmstate.git'
        HELM_STATE_REPO = 'app-mono-helmstate'
        DOCKERHUB_HOOK_SECRET = "dockerhub-webhook-secret"
        //GIT = credentials('github-credentials')
        GIT_USR = ""
        GIT_PSW = ""
    }

    stages {
        stage('configure webook') {
            steps {
                script {
                    setupWebhook()
	        }
	    }
        }
        stage('Update values.yaml in app-mono-helmstate') {
            steps {
                updateValuesfile()
            }
        }
    }
    post {
        cleanup {
            deleteDir()
        }
    }
}

def setupWebhook() {
    properties([
        pipelineTriggers([
            [$class: 'GenericTrigger',
                genericVariables: [
                    [key: 'TAG', value: '$.push_data.tag'],
                    [key: 'IMAGE', value: '$.repository.name'],
                    [key: 'REPONAME', value: '$.repository.repo_name'],
                ],
                causeString: 'Triggered on docker push',
                token: env.DOCKERHUB_HOOK_SECRET,
                printContributedVariables: true,
                printPostContent: true,
            ]
        ])
    ])
}

def updateValuesfile() {
    if (IMAGE != "") {
        env.IMAGE = IMAGE
        env.TAG = TAG
        sh '''
        git clone https://${GIT_USR}:${GIT_PSW}@${HELM_STATE_GIT_REPO}
        cd ${HELM_STATE_REPO}
        git checkout ${BRANCH}
        git reset --hard origin/${BRANCH}
        ls -l
        '''

        sh "sed -i 's/\\btag.*\\b/tag: $TAG/g' ${HELM_STATE_REPO}/$IMAGE/values.yaml"

        sh '''
        cd ${HELM_STATE_REPO}
        git branch
        git diff
        git status
        git config user.name ${GIT_USR}
        git config user.email ${GIT_USR}@users.noreply.github.com
        git add $IMAGE/values.yaml
        git commit -m \"Jenkins: Update image tag in $IMAGE/values.yaml to $TAG\"
        git push https://${GIT_USR}:${GIT_PSW}@${HELM_STATE_GIT_REPO}
        '''
    }
}
