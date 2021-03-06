# LXFramework
Um framework LXD que automatiza o *build machine* através de execução de comandos em série.

## Download e Install

```bash
git clone https://github.com/rauleite/lxf.git && cd lxf
./install.sh
cd ../ && sudo rm -r ./lxf #Opcional
```
*Para maiores detalhes sobre a instalação, paths e desinstação, [aqui](#detalhes-sobre-a-instalação)*

## Get started

Já tendo feito o `sudo lxd init`, como descrito na página de [configurações iniciais do lxd](https://stgraber.org/2016/03/15/lxd-2-0-installing-and-configuring-lxd-212/), basta criar o seguinte arquivo:

Crie o arquivo de exemplo: **`lxf-app.sh`**

```bash
# lxf-app.sh

# Configurações básicas
NETWORK "lxcbr0"
STORAGE_PATH "/var/lib/lxd/storage-pools/zfs/containers/$CONTAINER/rootfs" # coloque o seu storage path

# Configuração avançada
CONFIG "set $CONTAINER security.privileged true"

# Imagem base
FROM ubuntu/zesty/amd64
 
# Executa comandos no container
EXEC ls
```
No mesmo diretório, execute: `lxf app` 

Pronto, sua máquina será montada.

*Mais sobre execução e nome arquivo, [aqui](execução-do-arquivo)*

## Em ação
Além do manual dos comandos mais abaixo, pode também saltar para um [exemplo real e mais complexo](https://github.com/rauleite/lxf/blob/master/example), com bastante interação entre comandos.

## Sessões do manual

### Linha de comando
![help](https://github.com/rauleite/lxf/blob/master/help.png "lxf -h -> brazilian help")

*Mais sobre [opções linha de comando](#opções-de-linha-de-comando)*

### Tabela Rápida
* [Comandos de configuração](#comandos-de-configuração)
* [Comandos de execução](#comandos-de-execução)

### Todos os comandos
* [CONFIG](#config)
* [CONTAINER](#container)
* [COPY](#copy)
* [ENV](#env)
* [EXEC](#exec)
* [FILES](#files)
* [FROM](#from)
* [HOST_EXEC](#host_exec)
* [SIGNAL](#signal)
* [SOURCE](#source)
* [STORAGE](#storage)
* [STORAGE_PATH](#storage_path)
* [IPV4](#ipv4)
* [NETWORK](#network)
* [PRIVILEGED](#privileged)
* [USER_NAME](#user_name)
* [USER_GROUP](#user_group)
* [VAR](#var)
* [VOLUME](#volume)

## Opções de linha de comando
Todas as opções de linha de comando têm precedência sobre os comandos definidos no arquivo.

##### Exemplo
```bash
# lxf-app.sh

NETWORK "lxcbr01"

# FROM ...
```
Arquivo chamado via:

`lxf app --network lxcbr02`

O valor de NETWORK considerado para o container, será **lxcbr02**.

Mesmo que o arquivo defina um valor ao NETWORK, ele será ignorado uma vez que também foi setado como opção de linha de comando.

#### Caso do CONTAINER
Além de via opção de linha de comando e pelo comando CONTAINER do arquivo, o nome do container também pode ser definido simplesmente pelo nome do arquivo.

##### Exemplo
```bash
# lxf-app.sh

# CONTAINER omitido
# ...
# FROM ...
```
Chamando de qualquer de uma das possíveis maneiras:

* `lxf app`
* `lxf lxf-app.sh`
* `lxf lxf-app`
* `lxf app.sh`

O nome do container será **app**. Note que mesmo CONTAINER sendo omitido, e não tendo sido setado a opção `-c app` (ou `--container app`) via linha de comando, o nome do container será atribuido considerando o nome do arquivo (sem lxf- e .sh se houver).




## Tabela rápida de comandos

### Comandos de configuração

São chamado comandos de configuração, aqueles utilizados para configuração pré criação do container. Normalmente ficam ficam dispostos nas linhas antes do comando **FROM**.

Também é possível referenciá-los pelos outros comandos, mais comumente pelo comando **CONFIG**.

##### Exemplo:
```bash
# Comandos de configuração
CONTAINER "app"

CONFIG "set $CONTAINER security.privileged true"

# FROM ...
```

Comando                     | Descrição                             | Sintaxe
:--                         | :--                                   | :--
[CONFIG](#config)           | Qualquer [configuração lxd](https://github.com/lxc/lxd/blob/master/doc/configuration.md)   | `CONFIG <lxc-config>`
[CONTAINER](#container)         | Nome do container                 | `CONTAINER "<name>"`
[STORAGE](#storage)             | Nome do Storage pool              | `STORAGE "<nome>"`
[STORAGE_PATH](#storage_path)   | Path do Storage                   | `STORAGE_PATH "<path>"`
[IPV4](#ipv4)                   | Alias para ipv4 config            | `IPV4 <ip.fixo.do.container>`
[NETWORK](#network)             | Nome do device para brigde        | `NETWORK <nome_do_device>`
[PRIVILEGED](#privileged)       | Alias para privileged config      | `PRIVILEGED "<true>" ou "<false>"`
[USER_NAME](#user_name)         | User assumido para os comandos    | `USER_NAME "<name>"`
[USER_GROUP](#user_group)       | Group assumido para os comandos   | `USER_GROUP "<group>"`
[VAR](#var)                     | Define variáveis                  | `VAR <key> "<value>"`

### Comandos de Execução

São chamados comandos de execução aqueles que rodam logo após o container ser criado. Pode ser entendido que são basicamente aqueles dispostos nas linhas após o comando **FROM**.

```bash
# ...

# FROM ...

# Comandos de execução
EXEC apt-get install -y software-properties-common
```

Comando                     | Descrição                         | Sintaxe
:--                         | :--                               | :--
[COPY](#copy)               | Copia host -> container           | `COPY <host path> <container path>`
[ENV](#env)                 | Cria folder, com nome e grupo     | `<app_path_1> [<app_path_2>] ...[<app_path_10>]`
[EXEC](#exec)               | Executa comandos no container     | `EXEC <command>`
[FILES](#files)             | Executa arquivo no container      | `FILES <path_to_local_file>`
[FROM](#from)               | Imagem a ser usada                | `FROM <image>`
[HOST_EXEC](#host_exec)     | Executa comando no host           | `HOST_EXEC <command>`
[SIGNAL](#signal)           | start, stop ou restart container  | `SIGNAL "start" | "stop" | "restart"`
[SOURCE](#source)           | Reaproveita outro lxf-file        | `SOURCE "<path-to-lxf-file>"`
[VOLUME](#volume)           | Arquivo e diretorio compartilhado | `VOLUME <local-path> <container-path>`


# Comandos Detalhados

## CONFIG
#### Sintaxe
```bash
CONFIG <lxc-config>
```
```bash
CONFIG "set app security.privileged true"
```

* Utilizado para setar qualquer [configuração lxd](https://github.com/lxc/lxd/blob/master/doc/configuration.md), junto à criação do container. Utiliza a mesma sintaxe.
* Pode repetir o commando **CONFIG** quantas vezes forem necessárias, para criação do container desejado.

#### Exemplo:
```bash
CONTAINER "app"
NETWORK "lxcbr0"

# ...

### CONFIG ###
# Setando privileged true
CONFIG "set $CONTAINER security.privileged true"
# Setando IPV4 fixo
CONFIG "device set $CONTAINER $NETWORK ipv4.address 10.99.125.10"

# FROM ...
```

## CONTAINER
```bash
CONTAINER "<name>"
```
```bash
CONTAINER "app"
```

* Nome atribuido ao container
* Se não existir container com este nome, um novo será criado. Caso contrário utilizará o container existente.
* O comando **CONTAINER** também pode ser utilizado como variável no corpo do arquivo.

##### Exemplo:
```bash
CONTAINER "app"

# Usando CONTAINER como referência
CONFIG "set $CONTAINER security.privileged true"

# FROM ...

# Neste ponto container app está criado
```
Omitindo comando CONTAINER
```bash
# CONTAINER "app" --> Omitido

# Ainda assim é possível utilizar o comando CONTAINER como referência.
# Neste caso o nome do container será baseado por duas alternativas:
    # a) pelo nome do arquivo: lxf app;
    # b) pela configuração no caso de ter sido passado: lxf app -c myapp 
CONFIG "set $CONTAINER security.privileged true"

# FROM ...

# Neste ponto container app está criado
```

## COPY
```bash
COPY <host path> <container path>
```
```bash
COPY ./config/proxy/default /etc/nginx/sites-enabled/
```
Copia arquivo ou diretório, do host para o container.

##### Exemplo
```bash

USER_NAME "rleite"

# ...

VAR server_vm "/home/$USER_NAME/server"

# FROM ...

COPY ./package.json $server_vm/
COPY $server_host/config $server_vm/
```

## STORAGE
```bash
STORAGE "<nome>"
```
```bash
STORAGE "meupool"
```
Indique o nome do Storage pool.

Opte por usar STORAGE ao invés de STORAGE_PATH, por ser mais simples.

Obs. Não há sentido semântico usar ambos, porém se acontecer, será considerado o valor de STORAGE_PATH e STORAGE será ignorado.

##### Exemplo
```bash
# Exemplo no caso do ZFS
STORAGE "meupool"

# FROM...
```
A critério de comparação, teria exatamente o mesmo efeito que setar o STORAGE_PATH, como:
```bash
STORAGE_PATH "/var/lib/lxd/storage-pools/meupool/containers/$CONTAINER/rootfs"
```

## STORAGE_PATH
```bash
STORAGE_PATH "<path>"
```
Indique o path personalizado do seu Storage backends.

Prefira por usar STORAGE ao invés de STORAGE_PATH.

Obs. Não há sentido semântico usar ambos, porém se acontecer, será considerado o STORAGE_PATH e STORAGE será ignorado.

##### Exemplo
```bash
# Exemplo no caso do ZFS
STORAGE_PATH "/var/lib/lxd/storage-pools/meupool/containers/$CONTAINER/rootfs"

# FROM...
```

*Compare com [STORAGE](#storage)*

## ENV
```bash
ENV "<app_path_1> [<app_path_2>] ...[<app_path_10>]"
```
```bash
# FROM ...
ENV "/var/www"
```
**ENV** é um comando de conveniência para criação de um diretório (normalmente raiz das suas aplicações). Ele fará o seguinte:

1. Criará <app_path_1>, se ainda não existir...
    * Pseudo code:
        * `sudo mkdir -p <paths>`

1. com usuário e grupo indicado em `$USER_NAME` e `$USER_GROUP`...
    * Pseudo code:
        * `sudo chown $USER_NAME:$USER_GROUP <paths>`

2. e também vai setar este mesmo user e group como padrão, dos demais arquivos criados neste path.
    * Pseudo code:
        * `sudo chmod g+s <paths>`

##### Exemplo
Exemplo prático:

```bash
VAR web_path "/var/www"
VAR server_path "/var/server"
# FROM ...

ENV "$web_path $server_path"
# Neste ponto já existe /var/www e /var/server

# ...

# Referenciando
COPY ./web/build $web_path/
COPY ./server/build $web_path/
```

## EXEC
```bash
EXEC <command>
```
```bash
EXEC ls
```
Executa comandos no container.

##### Exemplo
```bash
EXEC cd /var/server && yarn install

# Linha quebrada e com VAR
EXEC \
    cd $server_vm && \
    yarn start

```

## FILES
```bash
FILES <host_src_file.sh> [<resources_1.sh>] ...[<resources_10.sh>] <dest_path>
```
```bash
FILES "/home/raul/dev/config/lxf/app.sh"
```
**FILES** é um comando de conveniência para execução de um arquivo, diretamente de dentro do próprio container.

O arquivo indicado será compiado para o container, e executado de dentro dele, considerando o devido `$USER_NAME`.

Se for passado mais de um arquivo, será compreendido que o primeiro é para execução, os intermediários são resources (utilizados por este primeiro), e o último é o path de destino dos arquivos.

Normalmente apenas um comando FILES será o suficiente, porém se houver a necessidade de setar mais de um comando FILES em um mesmo arquivo, multiplos comandos FILES são suportados.  

##### Exemplo
```bash
USER_NAME "rauleite"
# ...

VAR server_host "/home/raul/server"
VAR src_dest "/home/$USER_NAME/src_dest"

# FROM ...

FILES \
    "$server_host/config/lxf/app.sh" \ # Este será executado
    "$server_host/config/lxf/src.sh" \ # Resource utilizado pelo app.sh
    "$server_host/config/lxf/lib-color.sh" \ # Idem src.sh
    "$src_dest" # Path destino destes arquivos. Se não existir, será criado
```
Exemplo em ação: [lxf-create.sh](https://github.com/rauleite/lxf/blob/master/example/create_user/lxf-create.sh)

## FROM
```bash
FROM <image>
```
```bash
FROM ubuntu/zesty/amd64
```
Imagem que será usada como base da contrução do seu container. Pode utilizar uma imagem local, publicada ou baixada. Caso não exista, vai procurar online e baixar.


##### Exemplo
```bash
# ...

FROM minha/imagem/nodejs

# ...
```

## HOST_EXEC
```bash
HOST_EXEC <command>
```
```bash
HOST_EXEC ls
```
Executa comandos no host, geralmente utilizado para fazer algum build local, criar, editar ou manipular arquivos e diretórios, antes de enviar ao container via COPY, VOLUME etc


##### Exemplo
```bash
VAR web_host "/home/raul/env/www"
VAR web_vm "/var/www"

# FROM ...

# Faz algum comando que cria arquivos de build
HOST_EXEC [ ! -d $web_host/build ] && yarn build:web

# ...

# Compartilha arquivo build criado 
VOLUME $web_host/build $web_vm/

# ...
```

## IPV4
```bash
IPV4 <ip.fixo.do.container>
```
```bash
# ...

IPV4 "10.99.125.10"

# FROM ...
```
**IPV4** é um comando de conveniência, um alias para: `CONFIG "device set $CONTAINER $NETWORK ipv4.address 10.99.125.10"`.

Cria um container com ipv4 fixo, ao invés do comportamento padrão que seria atribuido um endereço dinâmico via dhcp.

##### Exemplo
```bash
IPV4 "10.99.125.10"

# VAR ...

# FROM ...
```

## NETWORK
```bash
NETWORK <nome_do_device>
```
```bash
NETWORK "lxcbr0"
# VAR ...
# FROM ...
# ...
```
**NETWORK** é um comando de conveniência, um alias para: `lxc network attach $NETWORK $CONTAINER`

## PRIVILEGED
```bash
PRIVILEGED "<true>" ou "<false>"
```
```bash
PRIVILEGED "true"
# VAR ...
# FROM ...
# ...
```
**PRIVILEGED** é um comando de conveniência, um alias para: `CONFIG "set $CONTAINER security.privileged true"`

## SIGNAL
```bash
SIGNAL "start" | "stop" | "restart"
```
```bash
SIGNAL "restart"
```
Envia sinal ao CONTAINER, de start, stop ou restart.

Semelhante à fazer na linha de comando: `lxc restart mycontainer`

## SOURCE
```bash
SOURCE <path-to-lxf-file>
```
```bash
SOURCE ./lxf-file-src.sh
```

Basicamente inclui outro arquivo , no momento em que `SOURCE` é chamado no arquivo corrente.

Normalmente usado para reaproveitar **comandos de execução**, como **EXEC, COPY, VOLUME** etc, considerando as definições de container, assim como definições e valores de variáveis do arquivo corrente.

Funciona exatamente como a ideia da keyword `source` do bash. Outras linguagens se apropriam desta estratégia, se apropriando do termo `include`.

Útil para quando você possui duas ou mais configurações diferentes, para um mesmo processo de execução. Ou seja, um ou mais comandos de configuração, para uma mesma sequência de comandos de execução.

##### Exemplo
1. Arquivo de resources: **`lxf-web-src.sh`**
    ```bash
    # Não é colocado neste arquivo os comandos de configuração, como CONTAINER, IPV4 etc.

    # Variavel $USER_NAME está definido em outro arquivo, que por sua vez referenciará este, via o comando SOURCE.
    VAR server_vm "/home/$USER_NAME/server"

    # FROM ...
    # ENV ...
    # FILES ...
        
    # HOST_EXEC ...

    COPY ./package.json $server_vm/

    # ...

    EXEC "cd $server_vm && yarn install"

    # ...
    ```
1. Arquivo A, que incluirá resources: **`lxf-web-node-a.sh`**
    ```bash
    CONTAINER   "node2"
    # ...
    USER_NAME   "rleite"
    IPV4        "10.99.125.10"

    # Entenda como que se o conteúdo de lxf-web-src.sh, estivesse daqui pra baixo 
    SOURCE ./lxf-web-src.sh
    ```
1. Arquivo B, que incluirá resources: **`lxd-web-node-b.sh`**
    ```bash
    CONTAINER   "node1"
    # ...
    USER_NAME   "rleite"
    IPV4        "10.99.125.20"

    # Entenda como que se o conteúdo de lxf-web-src.sh, estivesse daqui pra baixo
    SOURCE ./lxf-web-src.sh
    ```
## USER_NAME
```bash
USER_NAME "<name>"
```
```bash
USER_NAME "rleite"
```
User existente no container, que será assumido para executar comandos. Caso deseje executar como user root, basta fazer: `USER_NAME "root"`	

## USER_GROUP
```bash
USER_GROUP "<name>"
```
```bash
USER_GROUP "rleite"
```
GROUP existente no container, que será assumido para executar comandos. Caso deseje executar como group root, basta fazer: `USER_GROUP "root"`	

## VAR
```bash
VAR <key> "<value>"
```
```bash
VAR host_path "."
```
Define variáveis, para que seus valores sejam reutilizados durante toda a extensão do arquivo lxf.	

##### Exemplo
```bash

VAR host      "."
# Usando a variável recém criada
VAR web_host        "$host/www"
VAR server_host     "$host/server"
VAR web_vm          "/var/www"
VAR server_vm       "/home/$USER_NAME/server"
VAR src_dest        "/home/$USER_NAME/src_dest"

# FROM ...

ENV $web_vm $server_vm

# ...

FILES \
    "$server_host/config/lxf/app.sh" \
    "$server_host/config/lxf/src.sh" \
    "$src_dest"

# ...

# Nota: Usando no meio da execução no host
HOST_EXEC "[ ! -d $web_host/build ] && yarn build:web"

# ...

COPY $web_host/build $web_vm/

# ...

# Nota: Usando na execução remota
EXEC "cd $server_vm && yarn install"
EXEC "cd $server_vm && yarn start"

EXEC "sudo rm -r $server_vm/src"
```

## VOLUME
```bash
VOLUME <host-path> <container-path>
```
```bash
VOLUME ./web/build /var/www
```
Compartilha arquivo e diretorio. Bastante usado durante a fase de desenvolvimento do projeto.	

##### Exemplo
```bash
VAR web_host "$(pwd)/www"

# FROM ...

VOLUME $web_host/config/proxy/default /etc/nginx/sites-enabled/
VOLUME $web_host/config/proxy/nginx.conf /etc/nginx/nginx.conf
```

## Execução do arquivo
A extensão **.sh** é opcional (use para usufruir da colorização dos editores). Pode deixar sem nenhuma extensão se preferir.

Todas as seguintes maneiras de chamar o arquivo **`lxf-app.sh`**, são válidas:

* `lxf app` (sem lxf e .sh) 
* `lxf lxf-app.sh` (nome do arquivo completo)
* `lxf app.sh` (sem lxf)
* `lxf lxf-app` (sem .sh)

## Detalhes sobre a instalação
A instalação nada mais é do que alguns arquivos que são incluidos em seus respectivos paths. Sem alterações em arquivos como ~/.bashrc ou qualquer outro. A remoção, é a exclusão destes.

Default path do src e bin
* **/usr/local/src**
* **/usr/local/bin**

### Alterar Paths

Para facilitar o manuseio, não é utilizado variáveis de ambiente. Portanto, basta alterar o path em dois arquivos:

1. Em **./src_install.sh**
    ```bash
    declare -r SRC_PATH="/usr/local/src"
    declare -r BIN_PATH="/usr/local/bin"
    ```

1. Em **./bin/lxf.sh**
    ```bash
    declare -r SRC_PATH="/usr/local/src"
    ```

### Uninstall

Para desinstalar: 
```bash
./uninstall
```
ou *(caso não tenha mais os resources baixado)*
```bash
git clone https://github.com/rauleite/lxf.git && cd lxf && ./uninstall.sh && cd ../ && sudo rm -r ./lxf
```
Obs. Mesmo no caso de ter customizado o Path ao instalar, o comando acima é suficiente pra remover os arquivos corretos.

## TODO
* Criar yaml file, como docker-compose com [mikefarah/yaml](https://github.com/mikefarah/yaml)
* Criar Gui para REST API, com React Material e Nodejs no back.
* Criar repo - Lxfhub
* Criar mais configuracoes do tipo alias, de conveniências, como IPV4
* Deixar STORAGE_PATH mais user friendly.
<!-- * Criar mais parametros pra passar via cli. -->