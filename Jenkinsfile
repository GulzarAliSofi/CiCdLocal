pipeline {
    agent any
    stages {
        stage('Restore') {
            steps {
                bat 'dotnet restore CiCdLocal.sln'
            }
        }
        stage('Build') {
            steps {
                bat 'dotnet build CiCdLocal.sln --no-restore'
            }
        }
        stage('Test') {
            steps {
                bat 'dotnet test tests/SimpleWebApi.Test/SimpleWebApi.Test.csproj --no-build --no-restore --collect "XPlat Code Coverage"'
            }
            post {
                always {
                    recordCoverage(tools: [[parser: 'COBERTURA', pattern: '**/coverage.cobertura.xml']])
                }
            }
        }
        // stage('Run API') {
        //     steps {
        //         bat 'dotnet run --project src/CiCdLocal/CiCdLocal.csproj --no-build --no-restore'
        //     }
        // }
    }
}