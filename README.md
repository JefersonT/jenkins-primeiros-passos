# Jenkins e Docker
Este repositório foi criado com o objetivo de praticar as atividades do curso da Alura **[Jenkins e Docker: Pipeline de entrega continua](https://cursos.alura.com.br/course/pipeline-ci-jenkins-docker)**, portanto os passo-a-passos são criados de acordo com o material do curso assim como alguns arquivos bases para o desenvolvimento do projeto.

O Jenkins é um servidor de *Integração Contínua* open-source escrito em Java. Ele é o mais popular mas não a única opção. Outros servidores de *Integração Contínua* são TeamCity, Bamboo, Travis CI ou Gitlab CI entre vários outros.

## Configurando o ambiente e conectando máquina virtual:
### Máquina virtual com o vagrant
- Requisitos: http://vagrantup.com e https://www.virtualbox.org/
    - Baixar e descompactar o arquivo 1110-aula-inicial.zip
    - Entendendo o Vagranfile
    - Subindo o ambiente virtualizado
        ```
        $ vagrant plugin install vagrant-disksize
        $ vagrant up
        $ vagrant ssh
        $ ps -ef | grep -i mysql # Verificando se o MySQL esta rodando
        $ mysql -u devops -p # Senha mestre; show databases
        $ mysql -u devops_dev -p # Senha mestre; show databases
        ```
        - Instalando o Jenkins
            ```
            $ cd /vagrant/scripts
            ```
        - Visualizar o conteudo do arquivo de instalacao do jenkins

            ```
            $ sudo ./jenkins.sh
            ```

        - Acessar:  192.168.33.10:8080

            ```
            $ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
            ```

        - Credenciais
            - Nome de usuário: alura
            - Senha: mestre123
            - Nome completo: Jenkins Alura
            - Email: aluno@alura.com.br

        - Reload nas permissoes do docker
            ```
            $ sudo usermod -aG docker $USER
            $ sudo usermod -aG docker jenkins
            $ exit
            ```
        ```
        $ vagrant reload
        ```
        - Observações:
            - No momento da configuração do ambiente pode haver a necessidade de alterar o ip da máquina virtual criado pelo vagrant para um ip aceito pelo range do IP do seu virtual box.
            - O script de instalação do Jankins pode está desatualizado, portanto será necessário atualizar o script conforme a [Documentação do Jankins](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu).
### Passos para a configuracao do git e versionamento do codigo
- Criar uma conta em github.com
    ```
    $ ssh-keygen -t rsa -b 4096 -C "<seu-usuario>@gmail.com"
    $ cat ~/.ssh/id_rsa.pub
    ```
- Configurar a chave no github
    ```
    $ git config --global user.name "<seu-usuario>"
    $ git config --global user.email <seu-usuario>@<seu-providor>
    $ ssh -T git@github.com
    ```
- Fazer download do código nos anexos e copiar para o diretório compartilhado app
    ```
    $ cd /vagrant/jenkins-todo-list
    $ git init
    $ git add .
    $ git commit -m "Meu primeiro commit"
    $ git log
    ```
- Criar um repositório no github: jenkins-todo-list
    ```
    $ git remote add origin git@github.com:<seu-usuario>/jenkins-todo-list.git
    $ git push -u origin master
    ```
### Configurando a chave privada criada no ambiente da VM no Jenkins
- Pegue a chave privada:
    ```
    $ cat ~/.ssh/id_rsa
    ```
- Acessar:  192.168.33.10:8080
    - Perfil -> Credentials -> Jenkins -> Global Credentials -> Add Crendentials -> SSH Username with private key [ github-ssh ]
### Criando o primeiro job que vai monitorar o repositorio
Novo job -> jenkins-todo-list-principal -> Freestyle project:
Esse job vai fazer o build do projeto e registrar a imagem no repositório.
- Gerenciamento de código fonte:
    - Git: git@github.com:rafaelvzago/treinamento-devops-alura.git [SSH]
    - Credentials: git (github-ssh)
    - Branch: master
- Trigger de builds
    - Pool SCM: * * * * *
- Ambiente de build
    - Delete workspace before build starts
- Salvar
- Validar o log em: Git Log de consulta periódica

#### Passos para configurar a app e subir manualmente
- Criando o arquivo .env (temporário)
    ```
    $ cd /vagrant/jenkins-todo-list/to_do/
    $ vi .env
    ```
        [config]
        - Secret configuration
        SECRET_KEY = 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'

        - conf
        DEBUG=True

        - Database
        DB_NAME = "todo_dev"
        DB_USER = "devops_dev"
        DB_PASSWORD = "mestre"
        DB_HOST = "localhost"
        DB_PORT = "3306"
- Instalando o venv
    ```
    $ sudo pip3 install virtualenv nose coverage nosexcover pylint
    ```
- Criando e ativando o venv (dev)
    ```
    $ cd ../    
    $ virtualenv  --always-copy  venv-django-todolist
    $ source venv-django-todolist/bin/activate
    $ pip install -r requirements.txt
    ```
- Fazendo a migracao inicial dos dados
    ```
    $ python manage.py makemigrations
    $ python manage.py migrate
    ```
- Criando o superuser para acessar a app
    ```
    $ python manage.py createsuperuser
    ```
- Repetir o processo de migracaoção para o ambiente de produção:
    ```
    $ vi to_do/.env
    ```
        [config]
        - Secret configuration
        SECRET_KEY = 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'

        - conf
        DEBUG=True

        - Database
        DB_NAME = "todo"
        DB_USER = "devops"
        DB_PASSWORD = "mestre"
        DB_HOST = "localhost"
        DB_PORT = "3306"

- Fazendo a migracao inicial dos dados
    ```
    $ python manage.py makemigrations
    $ python manage.py migrate
    ```
- Criando o superuser para acessar a app
    ```
    $ python manage.py createsuperuser
    ```
- Verificar o ip do servidor
    ```
    $ ip addr
    ```
- Rodando a app
    ```
    $ python manage.py runserver 0:8000
    ```
- Acesse http://192.168.33.10:8000.
### Expor o deamon do docker
```
$ sudo mkdir -p /etc/systemd/system/docker.service.d/
$ sudo vi /etc/systemd/system/docker.service.d/override.conf
    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker.service
```
### Instalando os plugins Docker
- Gerenciar Jenkins -> Gerenciar Plugins -> Disponíveis
    - docker
- Install without restart -> Depois reiniciar o jenkins
- Gerenciar Jenkins -> Configurar o sistema -> Nuvem
    - Name: docker
    - URI: tcp://127.0.0.1:2376
    - Enabled
- This project is parameterized: 
    - DOCKER_HOST
    - tcp://127.0.0.1:2376
- Voltar no job criado na aula anterior
    - Manter a mesma configuracao do GIT para desenvolvimento
    - Build step 1: Executar Shell
- Validando a sintaxe do Dockerfile
    - docker run --rm -i hadolint/hadolint < Dockerfile
    - Build step 2: Build/Publish Docker Image
        - Directory for Dockerfile: ./
        - Cloud: docker
        - Image: rafaelvzago/django_todolist_image_build

### Instalar o plugin Config File Provider
- Gerenciar Jenkins -> Gerenciar Plugins -> Disponíveis
    - Config File Provider
- Install without restart

### Configurar o Managed Files para Dev
- Gerenciar Jenkins -> Gerenciar arquivos
    - Adicionar uma nova Config
        - Name : .env-dev
            ```
            [config]
            # Secret configuration
            SECRET_KEY = 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'
            # conf
            DEBUG=True
            # Database
            DB_NAME = "todo_dev"
            DB_USER = "devops_dev"
            DB_PASSWORD = "mestre"
            DB_HOST = "localhost"
            DB_PORT = "3306"
            ```
### Configurar o Managed Files para Prod
- Gerenciar Jenkins -> Gerenciar arquivos
    - Adicionar uma nova Config
        - Name : .env-prod
            ```
            [config]
            # Secret configuration
            SECRET_KEY = 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'
            # conf
            DEBUG=True
            # Database
            DB_NAME = "todo"
            DB_USER = "devops"
            DB_PASSWORD = "mestre"
            DB_HOST = "localhost"
            DB_PORT = "3306"
            ```
- No job: jenkins-todo-list-principal importar o env de dev para teste:

    - Adicionar passo no build: Provide configuration Files
    - File: .env-dev
    - Target: ./to_do/.env
    - Adicionar passo no build: Executar Shell
        - Criando o Script para Subir o container com o arquivo de env e testar a app:
            ```
            #!/bin/sh

            # Subindo o container de teste
            docker run -d -p 82:8000 -v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock -v /var/lib/jenkins/workspace/jenkins-todo-list-principal/to_do/.env:/usr/src/app/to_do/.env --name=todo-list-teste django_todolist_image_build

            # Testando a imagem
            docker exec -i todo-list-teste python manage.py test --keep
            exit_code=$?

            # Derrubando o container velho
            docker rm -f todo-list-teste

            if [ $exit_code -ne 0 ]; then
                exit 1
            fi
            ```
    - Execute o projeto para testar.
### Instalar o plugin: Parameterized Trigger 
- Gerenciar Jenkins -> Gerenciar Plugins -> Disponíveis
    - Parameterized Trigger
- Install without restart

### Modificar o Job para startar com 2 parametros:
- Geral:
    - Este build é parametrizado com 2 parametros de string
        - Nome: image
            - Valor padrão: <seu-usuario-no-dockerhub>/django_todolist_image_build

        - Nome: DOCKER_HOST
            - Valor padrão: tcp://127.0.0.1:2376

- No build step: Build / Publish Docker Image
    - Mudar o nome da imagem para: <seu-usuario-no-dockerhub>/django_todolist_image_build
    - Marcar: Push Image e configurar **suas credenciais** no dockerhub

- Mudar no job de teste a imagem para: ${image}
    ```
    docker run -d -p 82:8000 -v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock -v /var/lib/jenkins/workspace/jenkins-todo-list-principal/to_do/.env:/usr/src/app/to_do/.env --name=todo-list-teste ${image}
    ```
### Criar app no slack:
- Defina o nome do canal de notificação de: pipeline-todolist
- Guardar as seguintes informações:
    - Workspace: <Subdomínio da equipe>
    - Token de integração: <ID da credencial do token de integração do Jenkins app no seu canal do Slack>

### Instalar o plugin do slack: 
- Gerenciar Jenkins > Gerenciar Plugins > Disponíveis: Slack Notification
    - Configurar no jenkins: Gerenciar Jenkins > Configuraçao o sistema > Global Slack Notifier Settings
        - Workspace: <Subdomínio da equipe>
        - Integration Token Credential ID : ADD > Jenkins > Secret Text
            - Secret: <ID da credencial do token de integração do Jenkins app no seu canal do Slack>
            - ID: slack-token
        - Channel or Slack ID: pipeline-todolist

### Novo Job: 
- **Nome**: todo-list-desenvolvimento:
    - **Tipo**: Pipeline
    - Este build é parametrizado com 2 Builds de Strings:
        - Nome: image
            - Valor padrão: - Vazio, pois o valor sera recebido do job anterior.

        - Nome: DOCKER_HOST
            - Valor padrão: tcp://127.0.0.1:2376
- Script groover:
    ```
    pipeline {
        environment {
            dockerImage = "${image}"
        }
        agent any

        stages {
            stage('Carregando o ENV de desenvolvimento') {
                steps {
                    configFileProvider([configFile(fileId: '<id do seu arquivo de desenvolvimento>', variable: 'env')]) {
                        sh 'cat $env > .env'
                    }
                }
            }
            stage('Derrubando o container antigo') {
                steps {
                    script {
                        try {
                            sh 'docker rm -f django-todolist-dev'
                        } catch (Exception e) {
                            sh "echo $e"
                        }
                    }
                }
            }        
            stage('Subindo o container novo') {
                steps {
                    script {
                        try {
                            sh 'docker run -d -p 81:8000 -v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock -v /var/lib/jenkins/workspace/jenkins-todo-list-desenvolvimento/.env:/usr/src/app/to_do/.env --name=django-todolist-dev ' + dockerImage + ':latest'
                        } catch (Exception e) {
                            slackSend (color: 'error', message: "[ FALHA ] Não foi possivel subir o container - ${BUILD_URL} em ${currentBuild.duration}s", tokenCredentialId: 'slack-token')
                            sh "echo $e"
                            currentBuild.result = 'ABORTED'
                            error('Erro')
                        }
                    }
                }
            }
            stage('Notificando o usuario') {
                steps {
                    slackSend (color: 'good', message: '[ Sucesso ] O novo build esta disponivel em: http://192.168.33.10:81/ ', tokenCredentialId: 'slack-token')
                }
            }
        }
    }
    ```

### todo-list-principal > Configurar
- Definir post build: image=$image
- Salvar
- Executar o projeto.

### Subindo o container com o Sonarcube
- Na máquina devops (Vagrant):
    ```
    $ docker run -d --name sonarqube -p 9000:9000 sonarqube:lts
    ```
- Acessar: http://192.168.33.10:9000
    - Usuário: admin
    - Senha: admin
    - Redefina a senha como preferir
    - Add Project
        - Manually
        - Name: jenkins-todolist
        - Provide a token: enkins-todolist e anotar o seu token
        - Run analysis on your project > Other (JS, Python, PHP, ...) > Linux > django-todo-list
        - Copie o shell script fornecido
            ```
            sonar-scanner \
                -Dsonar.projectKey=enkins-todolist \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://192.168.33.10:9000 \
                -Dsonar.login=<seu token>
            ```
# Criar um job para Coverage com o nome: todo-list-sonarqube
- Gerenciamento de código fonte > Git
    - git: git@github.com:alura-cursos/jenkins-todo-list.git (Selecione as mesmas credenciais)
    - branch: master
    - Pool SCM: * * * * *
    - Delete workspace before build starts
    - Execute Script:
        - Build > Adicionar passo no build > Executar Shell
        ```
        #!/bin/bash
        # Baixando o Sonarqube
        wget https://s3.amazonaws.com/caelum-online-public/1110-jenkins/05/sonar-scanner-cli-3.3.0.1492-linux.zip

        # Descompactando o scanner
        unzip sonar-scanner-cli-3.3.0.1492-linux.zip

        # Rodando o Scanner
        ./sonar-scanner-3.3.0.1492-linux/bin/sonar-scanner   -X \
            -Dsonar.projectKey=jenkins-todolist \
            -Dsonar.sources=. \
            -Dsonar.host.url=http://192.168.33.10:9000 \
            -Dsonar.login=<seu token>
        ```
    - Salvar.