JOB_NAME = "$JOB_BASE_NAME"
ENVIRONMENT = env.env

def AUTOMOTIVE = 'automotive'
def DIGITAL = 'digital'
def FGL = 'fgl'
def FRONTEND = 'frontend'
def GTIN = 'gtin'
def MDM = 'mdm'
def MARKS = 'marks'
def VENDOR = 'vendormdm'
def CUSTOM_JOB = 'custom'

def executeGradleTask(taskName) {
    try {
        currentBuild.result = 'SUCCESS'
        def GRADLE_HOME = tool name: 'Gradle 4.10.2', type: 'hudson.plugins.gradle.GradleInstallation'
        println "Gradle params: $taskName\nSKU range: $env.skuLowBoundary - $env.skuHighBoundary\nEnvironment: $ENVIRONMENT" +
                "\nBranch: $env.git_branch\nNode: $Node_Name\nHeadLess mode: $env.headless"

        bat "$GRADLE_HOME\\bin\\gradle.bat clean -i --continue $taskName"
    } catch (Exception ex) {
        println("Test execution failed: $ex")
        currentBuild.result = 'FAILED'
    }
}

pipeline {
    agent { node { label "$Node_Name" } }
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    println "Clearing The current workspace folder is $env.WORKSPACE"
                    try {
                        bat "taskkill /IM chromedriver.exe /F > NUL"
                        bat "taskkill /IM chrome.exe /F > NUL"
                    } catch (Exception ex) {
                        println("No active chromedriver: $ex")
                    }
                    bat "DEL /F /Q /S ${env.WORKSPACE}\\*.* > NUL & RMDIR /Q /S ${env.WORKSPACE}\\ > NUL & dir ${env.WORKSPACE}"
                }
                checkout([
                        $class           : 'GitSCM',
                        branches         : [[name: '*/${git_branch}']],
                        gitTool          : 'Git-P5CPAJ00001JKWS',
                        userRemoteConfigs: [
                                [
                                        credentialsId: '0252520b-3a6c-4b94-8e1c-ff657d89be92',
                                        url          : 'ssh://git@bitbucket.corp.ad.ctc:22/mdmdevops/auto.git'
                                ]
                        ]
                ])
            }
        }
        stage('Check Portals') {
            when {
                not {
                    expression { return JOB_NAME.startsWith("custom") }
                }
            }
            steps {
                script {
                    def result
                    if (ENVIRONMENT == 'qa')
                        base_url = 'http://stepmdmqa.labcorp.ad.ctc'
                    else if (ENVIRONMENT == 'tst3' || ENVIRONMENT == 'test3')
                        base_url = 'http://stepmdmtst3.labcorp.ad.ctc'
                    else if (ENVIRONMENT == 'dev3')
                        base_url = 'http://d9lcwdcmdmap03.labcorp.ad.ctc'
                    else if (ENVIRONMENT == 'test2')
                        base_url = 'http://q9icwpemdm01.idm.ad.ctc:8443'
                    testSuite = JOB_NAME.substring(0, JOB_NAME.indexOf('-'))
                    result = bat "portal_check.cmd $base_url $testSuite exit %ERRORLEVEL%"
                    currentBuild.result = result == null ? 'SUCCESS' : 'FAILED'
                }
            }
        }
        stage('ExecuteTests') {
            steps {
                script {
                    if (JOB_NAME.startsWith(AUTOMOTIVE))
                        executeGradleTask('runAutomotiveTests')
                    else if (JOB_NAME.startsWith(DIGITAL))
                        executeGradleTask('runDigitalTests')
                    else if (JOB_NAME.startsWith(FGL))
                        executeGradleTask('runFglTests')
                    else if (JOB_NAME.startsWith(FRONTEND))
                        executeGradleTask('runFrontendTests')
                    else if (JOB_NAME.startsWith(GTIN))
                        executeGradleTask('runGtinTests')
                    else if (JOB_NAME.startsWith(MDM))
                        executeGradleTask('runMdmTests')
                    else if (JOB_NAME.startsWith(MARKS))
                        executeGradleTask('runMarksTests')
                    else if (JOB_NAME.startsWith(VENDOR))
                        executeGradleTask('runVendorMdmTests')
                    else if (JOB_NAME.startsWith(CUSTOM_JOB)) {
                        def testsToRun = 'test'
                        for (String testId : "$test_name".split(' ')) {
                            testsToRun += " --tests *$testId"
                        }
                        executeGradleTask(testsToRun)
                    } else
                        println 'Unable to start gradle task. Unknown job name: ' + JOB_NAME
                }
            }
        }
        stage('Allure') {
            steps {
                script {
                    try {
                        allure([includeProperties: false, jdk: '', results: [[path: 'build/allure-results']]])
                    } catch (Exception ex) {
                        println("Allure exception occurred: $ex")
                    } finally {
                        println("Allure report generated")
                    }
                }
            }
        }
        stage('Clean Test Data') {
            when {
                anyOf {
                    expression { return JOB_NAME.contains('automotive') }
                    expression { return JOB_NAME.contains('digital') }
                    expression { return JOB_NAME.contains('gtin') }
                    expression { return JOB_NAME.contains('mdm') }
                    expression { return JOB_NAME.contains('custom') }
                }
            }
            steps {
                script {
                    try {
                        def GRADLE_HOME = tool name: 'Gradle 4.10.2', type: 'hudson.plugins.gradle.GradleInstallation'
                        bat "$GRADLE_HOME\\bin\\gradle.bat -i deleteProducts"
                    } catch (Exception ex) {
                        println("Clean exception occurred: $ex")
                    }
                }
            }
        }
    }
}
