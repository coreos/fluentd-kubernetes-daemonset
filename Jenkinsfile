podTemplate(
    cloud: 'kubernetes',
    containers: [
        containerTemplate(
            alwaysPullImage: false,
            envVars: [],
            command: 'dockerd-entrypoint.sh',
            args: '--storage-driver=overlay',
            volumes: [
                emptyDirVolume(
                    memory: false,
                    mountPath: '/var/lib/docker',
                ),
            ],
            image: 'docker:dind',
            name: 'docker',
            privileged: true,
            resourceRequestCpu: '1000m',
            resourceRequestMemory: '500Mi',
            resourceLimitCpu: '2000m',
            resourceLimitMemory: '2048Mi',
            ttyEnabled: true,
        ),
    ],
    instanceCap: 5,
    label: 'tectonic-fluentd-kubernetes',
    name: 'tectonic-fluentd-kubernetes',
) {
    node('tectonic-fluentd-kubernetes') {
        def isTectonicLoggingBranch = env.BRANCH_NAME == "tectonic_logging"

        stage('Checkout') {
            container('docker'){
                sh 'apk add --no-cache git make'
                checkout scm
            }
        }

        stage('Build') {
            withCredentials([
                usernamePassword(credentialsId: 'quay-coreos-jenkins-push', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME'),
            ]) {
                container('docker'){
                    echo "Authenticating to docker registry"
                    sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD quay.io"
                    if (isTectonicLoggingBranch) {
                        echo "Branch is ${env.BRANCH_NAME}, building, tagging, and pushing images"
                        sh """
                        make release-all \
                            no-cache=yes \
                            IMAGE_NAME=quay.io/coreos/fluentd-kubernetes
                        """
                    } else {
                        echo "Building images, skipping tag/push"
                        sh """
                        make image-all \
                            no-cache=yes \
                            IMAGE_NAME=quay.io/coreos/fluentd-kubernetes
                        """
                    }
                }
            }
        }
    }
}
