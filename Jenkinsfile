def BranchToPort(String branchName) {
    def BranchPortMap = [
        [branch: 'master'   , port: 15565],
        [branch: 'Release'  , port: 15566],
        [branch: 'Feature'  , port: 15567],
        [branch: 'Prototype', port: 15568],
        [branch: 'HotFix'   , port: 15569]
    ]
    BranchPortMap.find { it['branch'] ==  branchName }['port']
}
 
def StartContainer() {
    sh "docker volume create --driver=pure -o size=4GB ${env.BRANCH_NAME}"
    docker.image('microsoft/mssql-server-linux:2017-latest').run("-v ${env.BRANCH_NAME}:/var/opt/mssql -e ACCEPT_EULA=Y -e SA_PASSWORD=P@ssword1 --name SQLLinux${env.BRANCH_NAME} -d -i -p  ${BranchToPort(env.BRANCH_NAME)}:1433")
}
 
def DeployDacpac() {
    def SqlPackage = "C:\\Program Files\\Microsoft SQL Server\\140\\DAC\\bin\\sqlpackage.exe"
    def SourceFile = "SelfBuildPipelineDV\\bin\\Release\\SelfBuildPipelineDV.dacpac"
    def ConnString = "server=linux-slave,${BranchToPort(env.BRANCH_NAME)};database=SsdtDevOpsDemo;user id=sa;password=P@ssword1"
 
    unstash 'theDacpac'
    bat "\"${SqlPackage}\" /Action:Publish /SourceFile:\"${SourceFile}\" /TargetConnectionString:\"${ConnString}\" /p:ExcludeObjectType=Logins"
}
 
node {
    stage('git checkout') {
        checkout scm
    }
    stage('build dacpac') {
        bat "\"${tool name: 'Default', type: 'msbuild'}\" /p:Configuration=Release"
        stash includes: 'SelfBuildPipelineDV\\bin\\Release\\SelfBuildPipelineDV.dacpac', name: 'theDacpac'
    }
}

node('linux-slave') {
    stage('start container') {
        StartContainer()
    }
}

node {
    stage('Deploy DACPAC') {
        DeployDacpac()
    }
}

node ('linux-slave') {
    stage('Clean up') {
        sh "docker rm -f SQLLinux${env.BRANCH_NAME}"
        sh "docker volume rm ${env.BRANCH_NAME}"
    }
}
