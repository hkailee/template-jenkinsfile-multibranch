def imageName = 'mlabouardy/movies-loader'
def registry = '305929695733.dkr.ecr.eu-west-3.amazonaws.com'
def region = 'eu-west-3'

node('workers'){
    try {
        stage('Checkout'){
            checkout scm
        }

        stage('Unit Tests'){
            def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")
            imageTest.inside{
                sh "python test_main.py"
            }
        }

        stage('Build'){
            docker.build(imageName)
        }

        stage('Push'){
            sh "\$(aws ecr get-login --no-include-email --region ${region}) || true"
            docker.withRegistry("https://${registry}") {
                docker.image(imageName).push(commitID())

                if (env.BRANCH_NAME == 'develop') {
                    docker.image(imageName).push('develop')
                }

                if (env.BRANCH_NAME == 'preprod') {
                    docker.image(imageName).push('preprod')
                }

                if (env.BRANCH_NAME == 'master') {
                    docker.image(imageName).push('latest')
                }
            }
        }

        stage('Analyze'){
            def scannedImage = "${registry}/${imageName}:${commitID()} ${workspace}/Dockerfile"
            writeFile file: 'images', text: scannedImage
            anchore name: 'images'
        }

        stage('Deploy'){
            if(env.BRANCH_NAME == 'develop' || env.BRANCH_NAME == 'preprod'){
                build job: "watchlist-deployment/${env.BRANCH_NAME}"
            }

            if(env.BRANCH_NAME == 'master'){
                timeout(time: 2, unit: "HOURS") {
                    input message: "Approve Deploy?", ok: "Yes"
                }
                build job: "watchlist-deployment/master"
            }
        }
    } catch(e){
        currentBuild.result = 'FAILED'
        throw e
    } finally {
        echo "Build ${currentBuild.result}"
    }
}



def commitAuthor(){
    sh 'git show -s --pretty=%an > .git/commitAuthor'
    def commitAuthor = readFile('.git/commitAuthor').trim()
    sh 'rm .git/commitAuthor'
    commitAuthor
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}

def commitMessage() {
    sh 'git log --format=%B -n 1 HEAD > .git/commitMessage'
    def commitMessage = readFile('.git/commitMessage').trim()
    sh 'rm .git/commitMessage'
    commitMessage
}