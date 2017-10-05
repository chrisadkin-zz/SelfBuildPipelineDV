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
    bat "docker -ti --volume-driver=pure -v ${env.BRANCH_NAME}:/data run -e \"ACCEPT_EULA=Y\" -e \"SA_PASSWORD=P@ssword1\" --name SQLLinux${env.BRANCH_NAME} -d -i -p ${BranchToPort(env.BRANCH_NAME)}:1433 microsoft/mssql-server-linux"
}
 
def DeployDacpac() {
    def SqlPackage = "C:\\Program Files\\Microsoft SQL Server\\140\\DAC\\bin\\sqlpackage.exe"
    def SourceFile = "SelfBuildPipeline\\bin\\Release\\SelfBuildPipeline.dacpac"
    def ConnString = "server=localhost,${BranchToPort(env.BRANCH_NAME)};database=SsdtDevOpsDemo;user id=sa;password=P@ssword1"
 
    unstash 'theDacpac'
    bat "\"${SqlPackage}\" /Action:Publish /SourceFile:\"${SourceFile}\" /TargetConnectionString:\"${ConnString}\" /p:ExcludeObjectType=Logins"
}
 
node('master') {
    stage('git checkout') {
        checkout scm
    }
    stage('build dacpac') {
        bat "\"${tool name: 'Default', type: 'msbuild'}\" /p:Configuration=Release"
        stash includes: 'SelfBuildPipeline\\bin\\Release\\SelfBuildPipeline.dacpac', name: 'theDacpac'
    }
}

node('linux-slave') {
    stage('start container') {
        StartContainer()
    }
}

node('master') {
    stage('deploy dacpac') {
}

node('linux-slave') {
    sh "docker rm -f SQLLinux${env.BRANCH_NAME}"
    sh "docker rm -f ${env.BRANCH_NAME}"
}
