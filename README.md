# Jenkins e Docker
Este repositório foi criado com o objetivo de praticar as atividades do curso da Alura **[Jenkins e Docker: Pipeline de entrega continua](https://cursos.alura.com.br/course/pipeline-ci-jenkins-docker)**, portanto os passo-a-passos são criados de acordo com o material do curso assim como alguns arquivos bases para o desenvolvimento do projeto.
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
## Anotações
-  Jenkins é um servidor de *Integração Contínua* open-source escrito em Java. Ele é o mais popular mas não a única opção. Outros servidores de *Integração Contínua* são TeamCity, Bamboo, Travis CI ou Gitlab CI entre vários outros.