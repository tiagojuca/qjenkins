// Parametros de entrada que nao precisam ser configurados via UI, mudam raramente
def SVN_BASE_URL = "https://svn.altoqi.com.br/svn/Eberick/"
def CHECKOUT_FOLDER_PREFIX = "eb"
def SCRIPTS_FOLDER = "Scripts"
def APP_7ZA_REL_PATH = "${SCRIPTS_FOLDER}\\Utils\\7za.exe"
def PYTHON_COMMAND = "python"
def PYTHON_BUILD_SCRIPT = "build.py"
def CMAKE_BINARY_DIR = "exeobj_cmake\\VC"
def OWL_VERSION = "6.42"
def BUNDLE_OUT_DIR = "D:\\jenkins\\jobs\\${env.JOB_NAME}\\builds\\${env.BUILD_NUMBER}"
def BUNDLE_PASSWD = "AltoQi12"

// Variaveis de saida populadas nas etapas de configuracao e espelhamento do Pipeline
def SVN_URL = ""
def SVN_REVISION = ""
def CHECKOUT_FOLDER = ""
def CHECKOUT_PATH = ""
def SCRIPTS_PATH = ""
def APP_7ZA_PATH = ""
def CMAKE_PROJECT_FOLDER = ""
def CMAKE_INSTALL_FOLDER = ""
def CMAKE_INSTALL_PREFIX = ""
def CMAKE_PDB_OUTPUT_DIRECTORY = ""
def PYTHON_BUILD_SCRIPT_FLAGS = ""
def BUNDLE_BASENAME = ""
def BUNDLE_NAME_EXEOBJ = ""
def BUNDLE_NAME_PDBS = ""
def BUNDLE_PATH_EXEOBJ = ""
def BUNDLE_PATH_PDBS = ""

pipeline {
  agent any

  parameters {
    string(name: "REVISAO_SVN", defaultValue: "HEAD", description: "Revisao do SVN a ser compilada")
    string(name: "BRANCH_SVN", defaultValue: "trunk", description: "Branch do SVN a ser compilada")
    choice(name: "TIPO_BUILD", choices: ["RELEASE", "DEBUG"], description: "Tipo de construcao")
    choice(name: "TRADUCAO", choices: ["DESABILITADA", "PT-BR", "ES-MX", "EN-US"], description: "Suporte a traducao")
    choice(name: "PROTECAO", choices: ["OAUTH", "RMS", "DEMO", "SL"], description: "Tipo de protecao")
    booleanParam(name: "COTIRE", defaultValue: true, description: "Compilar com Unity Build?")
    booleanParam(name: "VLD", defaultValue: false, description: "Compilar com Visual Leak Detector?")
    booleanParam(name: "PDB", defaultValue: false, description: "Distribuir simbolos de depuracao MSVC?")
  }

  environment {
    SERVER_PROTECAO = "http://aq000043:8080/"
  }

  stages {

    stage('Configurar') {
      steps {

        // Popula dados do workspace com base nos parametros de SVN
        script {
          def rota = params.BRANCH_SVN == "trunk" ? "trunk" : "branches/${params.BRANCH_SVN}"
          SVN_URL = "${SVN_BASE_URL}${rota}"

          CHECKOUT_FOLDER = "${CHECKOUT_FOLDER_PREFIX}_${params.BRANCH_SVN}"
          CHECKOUT_PATH = "${env.WORKSPACE}\\${CHECKOUT_FOLDER}"
          SCRIPTS_PATH = "${CHECKOUT_PATH}\\${SCRIPTS_FOLDER}"
          APP_7ZA_PATH = "${CHECKOUT_PATH}\\${APP_7ZA_REL_PATH}"
        }

        // Gera o comando de build com base nas opcoes selecionadas
        script {
          if (params.TIPO_BUILD == "DEBUG") {
            PYTHON_BUILD_SCRIPT_FLAGS += "-B=deb"
          } else {
            PYTHON_BUILD_SCRIPT_FLAGS += "-B=rel"
          }

          PYTHON_BUILD_SCRIPT_FLAGS += " -V=22 -TS=143"

          PYTHON_BUILD_SCRIPT_FLAGS += " "
          if (params.PROTECAO == "SL") {
            PYTHON_BUILD_SCRIPT_FLAGS += "-SL"
          } else if (params.PROTECAO == "DEMO") {
            PYTHON_BUILD_SCRIPT_FLAGS += "-D"
          } else if (params.PROTECAO == "RMS") {
            PYTHON_BUILD_SCRIPT_FLAGS += "-R"
          } else {
            PYTHON_BUILD_SCRIPT_FLAGS += "-O"
          }

          if (params.TRADUCAO != "DESABILITADA") {
            PYTHON_BUILD_SCRIPT_FLAGS += " -T -Lang="
            if (params.TRADUCAO == "EN-US") {
              PYTHON_BUILD_SCRIPT_FLAGS += "en-US"
            } else if (params.TRADUCAO == "ES-MX") {
              PYTHON_BUILD_SCRIPT_FLAGS += "es-MX"
            } else {
              PYTHON_BUILD_SCRIPT_FLAGS += "pt-BR"
            }
          }

          if (params.COTIRE) {
            PYTHON_BUILD_SCRIPT_FLAGS += " -C"
          }

          if (params.VLD) {
            PYTHON_BUILD_SCRIPT_FLAGS += " -L"
          }

          PYTHON_BUILD_SCRIPT_FLAGS += " -P2"
        }

        // Monta nome da pasta a ser gerada pelo comando de build
        script {
          if (params.TIPO_BUILD == "DEBUG") {
            CMAKE_PROJECT_FOLDER += "deb"
          } else {
            CMAKE_PROJECT_FOLDER += "rel"
          }

          CMAKE_PROJECT_FOLDER += "_vs22_msvc143"

          if (params.PROTECAO == "SL") {
            CMAKE_PROJECT_FOLDER += "_sl"
          } else if (params.PROTECAO == "DEMO") {
            CMAKE_PROJECT_FOLDER += "_demo"
          } else if (params.PROTECAO == "RMS") {
            CMAKE_PROJECT_FOLDER += "_rms"
          } else {
            CMAKE_PROJECT_FOLDER += "_oauth"
          }

          if (params.TRADUCAO != "DESABILITADA") {
            CMAKE_PROJECT_FOLDER += "_tr"
          }

          if (params.COTIRE) {
            CMAKE_PROJECT_FOLDER += "_cotire"
          }

          if (params.VLD) {
            CMAKE_PROJECT_FOLDER += "_vld"
          }
        }

        // Monta nome da pasta do comando de instalacao do CMake
        script {
          if (params.TIPO_BUILD == "DEBUG") {
            CMAKE_INSTALL_FOLDER += "Db"
          } else {
            CMAKE_INSTALL_FOLDER += "Rel"
          }

          CMAKE_INSTALL_FOLDER += "_OWL${OWL_VERSION}"

          if (params.VLD) {
            CMAKE_INSTALL_FOLDER += "_VLD"
          }

          if (params.PROTECAO != "DEMO") {
            if (params.PROTECAO == "SL") {
              CMAKE_INSTALL_FOLDER += "_SL"
            } else if (params.PROTECAO == "RMS") {
              CMAKE_INSTALL_FOLDER += "_RMS"
            } else {
              CMAKE_INSTALL_FOLDER += "_OAUTH"
            }
          }

          CMAKE_INSTALL_FOLDER += "_x64"

          if (params.PROTECAO == "DEMO") {
            CMAKE_INSTALL_FOLDER += "_Demo"
          }

          if (params.COTIRE) {
            CMAKE_INSTALL_FOLDER += "_COTIRE"
          }

          if (params.TRADUCAO != "DESABILITADA") {
            CMAKE_INSTALL_FOLDER += "_TRA"
          }

          CMAKE_INSTALL_PREFIX = "${CHECKOUT_PATH}\\${CMAKE_BINARY_DIR}\\${CMAKE_INSTALL_FOLDER}"
          CMAKE_PDB_OUTPUT_DIRECTORY = "${CMAKE_INSTALL_PREFIX}_pdb"

          env.EXEOBJ = "${CMAKE_INSTALL_PREFIX}"
        }

      }
    }

    stage("Espelhar") {
      steps {

        script {
          bat "svn checkout -r ${params.REVISAO_SVN} ${SVN_URL} ${CHECKOUT_FOLDER}"

          if (params.REVISAO_SVN != "HEAD") {
            SVN_REVISION = params.REVISAO_SVN
          } else {
            SVN_REVISION = bat(
              script: """
                @echo off
                for /f "tokens=2" %%i in ('svn info "${CHECKOUT_PATH}" ^| findstr /R "^Revision:"') do @echo %%i
              """,
              returnStdout: true
            ).trim()
          }
        }

        script {
          BUNDLE_BASENAME += "${SVN_REVISION}"

          if (params.BRANCH_SVN != "trunk") {
            BUNDLE_BASENAME += "_${params.BRANCH_SVN}"
          }

          if (params.TIPO_BUILD == "DEBUG") {
            BUNDLE_BASENAME += "_Debug"
          }

          if (params.VLD) {
            BUNDLE_BASENAME += "_VLD"
          }

          if (params.TRADUCAO != "DESABILITADA") {
            BUNDLE_BASENAME += "_Traducao"
          }

          if (params.PROTECAO == "SL") {
            BUNDLE_BASENAME += "_SL"
          } else if (params.PROTECAO == "DEMO") {
            BUNDLE_BASENAME += "_Demo"
          } else if (params.PROTECAO == "RMS") {
            BUNDLE_BASENAME += "_RMS"
          } else {
            BUNDLE_BASENAME += "_Cloud"
          }

          BUNDLE_NAME_EXEOBJ = "exeobj_${BUNDLE_BASENAME}.zip"
          BUNDLE_NAME_PDBS = "pdbs_${BUNDLE_BASENAME}.zip"

          BUNDLE_PATH_EXEOBJ = "${BUNDLE_OUT_DIR}\\${BUNDLE_NAME_EXEOBJ}"
          BUNDLE_PATH_PDBS = "${BUNDLE_OUT_DIR}\\${BUNDLE_NAME_PDBS}"
        }

      }
    }

    // stage("Analisar") {

    //   steps {
    //     echo "Analise Encoding"
    //   }

    //   steps {
    //     echo "Analise Estatica"
    //   }

    //   steps {
    //     echo "Analise Resources"
    //   }

    //   steps {
    //     echo "Analise Coverage"
    //   }

    //   script {
    //     if (params.COTIRE) {
    //       echo "Analise Cpps Isolados"
    //     }
    //   }

    // }

    stage("Construir") {
      steps {
        dir(SCRIPTS_PATH) {
          bat "${PYTHON_COMMAND} ${PYTHON_BUILD_SCRIPT} ${PYTHON_BUILD_SCRIPT_FLAGS}"
        }
      }
    }

    stage("Envelopar") {
      steps {
        dir(SCRIPTS_PATH) {
          bat "${SCRIPTS_PATH}\\ProtegeEberick.bat ENVELOPE_THEMIDA HASH"
          bat "type ${CMAKE_INSTALL_PREFIX}\\Eberick_themida.log"
          bat "del ${CMAKE_INSTALL_PREFIX}\\Eberick_themida.log"
          bat "del ${CMAKE_INSTALL_PREFIX}\\EberickAdmin_themida.log"
          bat "del ${CMAKE_INSTALL_PREFIX}\\SecureEngineSDK64.dll"
        }
      }
    }

    stage("Distribuir") {
      steps {
        script {
          def cmd = "${APP_7ZA_PATH} a -r -tzip -p${BUNDLE_PASSWD}"

          bat "${cmd} ${BUNDLE_PATH_EXEOBJ} ${CMAKE_INSTALL_PREFIX}\\*"
          if (params.PDB) {
            bat "${cmd} ${BUNDLE_PATH_PDBS} ${CMAKE_PDB_OUTPUT_DIRECTORY}\\*"
          }
        }
      }
    }

    // stage("Testar") {
    //   steps {
    //     echo "Testando"
    //   }
    // }

  }
}
