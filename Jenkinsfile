def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT"
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh "rm -f GIT_COMMIT"
    return gitCommit
}

node {
    // Checkout source code from Git
    stage 'Checkout'
    checkout scm

    // Build Docker image
    stage 'Build'
    sh "docker build -t lionmint/dcos-nuremberg-meetup:${gitCommit()} ."

    // Log in and push image to Docker Hub
    stage 'Publish'
    withCredentials(
        [[
            $class: 'UsernamePasswordMultiBinding',
            credentialsId: 'Lukaesch',
            passwordVariable: 'DOCKERHUB_PASSWORD',
            usernameVariable: 'DOCKERHUB_USERNAME'
        ]]
    ) {
        sh "docker login -u '${env.DOCKERHUB_USERNAME}' -p '${env.DOCKERHUB_PASSWORD}' -e lukas.schmyrczyk@lionmint.com"
        sh "docker push lionmint/dcos-nuremberg-meetup:${gitCommit()}"
    }

    // Deploy
    stage 'Deploy'

    marathon(
        url: 'http://marathon.mesos:8080',
        forceUpdate: false,
        filename: 'marathon.json',
        appid: 'nginx-test',
        docker: "lionmint/dcos-nuremberg-meetup:${gitCommit()}".toString()
    )

}
