pipeline 
{
    agent any
    tools 
    {
        go 'Go 1.19'
    }
    
    stages 
    {
        stage('Build Project') 
        {
            steps 
            {
                checkout scmGit(branches: [[name: '*/dev']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/amancooks08/Book-My-Sport.git/']])
                sh 'go build -o server'
                
                stash includes: 'server', name: 'server'
                stash includes: 'migrations/*', name:'migrations'
            }
        }
        
        stage('Test')
        {
            steps
            {
                withEnv(['CGO_ENABLED=0']) 
                {
                    sh'go test'
                }
            }
        }
        stage('Deployment') 
        {
            agent {node {label"deployment"}}
            steps 
            {
                sh 'pwd'
                
                // Remove previous deployment if any
                sh """
                    if pidof server ; then
                        killall server ;
                    fi
                """
                
                unstash name:'server' 
                unstash name:'migrations'
                
                //sh 'ls'
                
                sh './server migrate'
                
                // Create logs directory
                sh "mkdir -p logs"
                
                // Deploy server
                withEnv(['BUILD_ID=dontKillMe', 'JENKINS_NODE_COOKIE=dontKillMe']) 
                {
                    sh """
                        ./server start > logs/stdout.log 2> logs/error.log &
                    """
                }
            }
        }
        stage('Backup binary')
        {
            steps 
            {
                sh 'ls'    
                withAWS(region:"ap-south-1", credentials:"aws-account-id")
                {
                    s3Upload(file:'server', bucket:'book-my-sport-binary-files', path:'binaries/')
                }
                
                
            }
        }
    }
    
    post {
        always {
            mail bcc: '', body: " Hi Team \n I have forwarded the build status of $JOB_NAME  \n Build : $BUILD_NUMBER  ${currentBuild.currentResult}.\n \n \n Check the console output at ${env.BUILD_URL} to view results\n Thanks and Regards ", cc: 'mrunmai.kudale@joshsoftware.com', from: '', replyTo: '', subject: 'Email From Jenkins Job', to: 'omkar.khedkar@joshsoftware.com'
        }
    }
}
