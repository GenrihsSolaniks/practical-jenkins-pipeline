def clonePythonGreetings() {
    dir('python-greetings') {
        deleteDir()
    }
    dir('python-greetings') {
        git url: 'https://github.com/mtararujs/python-greetings.git', branch: 'main'
    }
}

def cloneJsApiFramework() {
    dir('course-js-api-framework') {
        deleteDir()
    }
    dir('course-js-api-framework') {
        git url: 'https://github.com/mtararujs/course-js-api-framework.git', branch: 'main'
    }
}

def installPythonDeps() {
    dir('python-greetings') {
        bat 'py -3 venv venv'
        bat 'venv\\Scripts\\python.exe -m pip install --upgrade pip'
        bat 'venv\\Scripts\\python.exe -m pip install -r requirements.txt'
    }
}

def deployStage(envName, port) {
    echo "Deploying application to ${envName} environment..."

    clonePythonGreetings()

    bat """
pm2 delete greetings-app-${envName}
EXIT /B 0
"""

    bat """
if not exist C:\\greetings-deploy mkdir C:\\greetings-deploy
if exist C:\\greetings-deploy\\${envName} rmdir /S /Q C:\\greetings-deploy\\${envName}
xcopy /E /I /Y python-greetings C:\\greetings-deploy\\${envName}
"""

    dir("C:/greetings-deploy/${envName}") {
        bat 'py -3 venv venv'
        bat 'venv\\Scripts\\python.exe -m pip install --upgrade pip'
        bat 'venv\\Scripts\\python.exe -m pip install -r requirements.txt'
    }

    bat """
set PORT=${port}
pm2 start C:\\greetings-deploy\\${envName}\\app.py --name greetings-app-${envName} --interpreter C:\\greetings-deploy\\${envName}\\venv\\Scripts\\python.exe
"""

    bat 'pm2 list'
}

def testStage(envName) {
    echo "Running API tests on ${envName} environment..."

    cloneJsApiFramework()

    dir('course-js-api-framework') {
        bat 'npm install'
        bat "npm run greetings greetings_${envName}"
    }
}

pipeline {
    agent any

    stages {
        stage('install-pip-deps') {
            steps {
                echo 'Installing all required dependencies...'
                clonePythonGreetings()
                dir('python-greetings') {
                    bat 'dir'
                }
                installPythonDeps()
            }
        }

        stage('deploy-to-dev') {
            steps {
                script {
                    deployStage('dev', '7001')
                }
            }
        }

        stage('tests-on-dev') {
            steps {
                script {
                    testStage('dev')
                }
            }
        }

        stage('deploy-to-stg') {
            steps {
                script {
                    deployStage('stg', '7002')
                }
            }
        }

        stage('tests-on-stg') {
            steps {
                script {
                    testStage('stg')
                }
            }
        }

        stage('deploy-to-preprod') {
            steps {
                script {
                    deployStage('preprod', '7003')
                }
            }
        }

        stage('tests-on-preprod') {
            steps {
                script {
                    testStage('preprod')
                }
            }
        }

        stage('deploy-to-prod') {
            steps {
                script {
                    deployStage('prod', '7004')
                }
            }
        }

        stage('tests-on-prod') {
            steps {
                script {
                    testStage('prod')
                }
            }
        }
    }
}
