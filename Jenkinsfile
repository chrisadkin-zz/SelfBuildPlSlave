properties([
  parameters([
    string(name: 'linuxbuildslave', defaultValue: 'ubuntu-vbox'),
    string(name: 'slaveipaddress' , defaultValue: '192.168.56.101'),
  ])
])

def StartContainer() {
    timeout(time: 20, unit: 'SECONDS') {
        sh "docker run -e \"ACCEPT_EULA=Y\" -e \"SA_PASSWORD=P@ssword1\" --name SQLLinux${env.BRANCH_NAME} -d -i -p ${BranchToPort(env.BRANCH_NAME)}:1433 microsoft/mssql-server-linux:2017-GA && sleep 15"
    }
}

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

def DeployDacpac() {
    def SqlPackage = "C:\\Program Files\\Microsoft SQL Server\\140\\DAC\\bin\\sqlpackage.exe"
    def SourceFile = "SelfBuildPlSlave\\bin\\Release\\SelfBuildPlSlave.dacpac"
    def ConnString = "server=${params.slaveipaddress},${BranchToPort(env.BRANCH_NAME)};database=SsdtDevOpsDemo;user id=sa;password=P@ssword1"
    
    unstash 'theDacpac'
    bat "\"${SqlPackage}\" /Action:Publish /SourceFile:\"${SourceFile}\" /TargetConnectionString:\"${ConnString}\" /p:ExcludeObjectType=Logins"
}

node('master') {
    stage('git checkout') {
        timeout(time: 5, unit: 'SECONDS') {        
            checkout scm
        }        
    }
    stage('build dacpac') {
        bat "\"${tool name: 'Default', type: 'msbuild'}\" /p:Configuration=Release"
        stash includes: 'SelfBuildPlSlave\\bin\\Release\\SelfBuildPlSlave.dacpac', name: 'theDacpac'
    }
}

node( params.linuxbuildslave ) {
    stage('start container') {
        StartContainer()
    }
}

node('master') {
    stage('deploy dacpac') {
        try {
            timeout(time: 60, unit: 'SECONDS') {
                DeployDacpac()
            }
        }
        catch (error) {
            throw error
        }
    }
}

node( params.linuxbuildslave ) {
    sh "docker rm -f SQLLinux${env.BRANCH_NAME}"
}
