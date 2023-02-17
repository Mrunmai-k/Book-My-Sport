pipeline 
{
    agent any
    tools 
    {
        go 'Go 1.19'
        dockerTool 'docker'
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
                
                sh 'ls'
                
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
    }   
}