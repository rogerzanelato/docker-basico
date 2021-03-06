# Básico sobre Docker-compose
Resumo e lembretes sobre como criar arquivos docker-compose.yml.

## Links Úteis
- Docker-compose vs dockerfile: https://cursos.alura.com.br/forum/topico-diferenca-entre-o-dockerfile-e-o-docker-compose-30250

## Introdução
Docker-compose é uma ferramenta útil para definir e executar uma aplicação que utilize vários container simultaneamente. Sem o docker-compose, caso nossa aplicação utilize múltiplos container (banco, business, frontend etc..), teríamos que executar um por um estabelecendo os links, para preparar nosso ambiente. Com o docker-compose, podemos resumir isso à um único arquivo.

## Arquivo
A configuração do arquivo docker-compose.yml é bem similar aos comandos que utilizamos ao efetuar o `docker run`. Veja por exemplo:

```dockerfile
version: '2'

services:
    db:
        image: mysql:5.7
        volumes:
            - db_data:/var/lib/mysql
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: somewordpress
            MYSQL_DATABASE: wordpress
            MYSQL_USER: root
            MYSQL_PASSWORD: root
        
    wordpress:
        depends_on:
            - db
        image: wordpress:latest
        ports:
            - "8000:80"
        restart: always
        environment:
            WORDPRESS_DB_HOST: db:3306
            WORDPRESS_DB_USER: root
            WORDPRESS_DB_PASSWORD: root
volumes:
    db_data:
```

**Observação: ** Arquivos yml trabalham com indentação, é importante que as tags estejam corretamente alinhadas.

- `version`: Versão do arquivo
- `services`: Containers que serão criados
- `db/wordpress`: Nome dos containers
- `image`: Docker image
- `volumes`: Mapeamento do volume
- `restart`: Indica quando queremos que o container seja reiniciado:
  - `"no"`: Default, não reinicia o container em nenhuma circunstância.
  - `always`: Tentar reiniciar o container sempre que ele parar, independente do motivo.
    - Queremos utilizar o always em containers que nunca devem parar, como um webserver
  - `unless-stopped`: Igual o `always` porém, ao reiniciarmos o computador, os containers `always` serão reativados quando o docker estiver em execução, enquanto os containers `unless-stopped` permanecerão desativados ([segundo aqui](https://www.reddit.com/r/docker/comments/7iqkk0/what_is_the_purpose_of_restart_always_when_i_just/), não testei).
  - `on-failure`: Reinicia o container apenas se o container foi parado devido à um error code.
    - No Linux o processo é sempre encerrado com um código de resultado da operação, 0 significa que foi bem sucedido e nesse caso, não iria ser reiniciado. Códigos 1-255 indicam um erro, e seriam reiniciado.
    - Queremos utilizar esse formato de `restart` para containers que são levantados para cumprir uma tarefa única e finalizados no fim, ex: importação de dados, leitura de um arquivo csv, pdf, etc...
- `environment`: Variáveis de ambiente que serão criadas no container
- `depends_on`: Indica que o container depende de outro para estar ativo
- `ports`: Mapeamento das portas


## Executar
Para criar os containers, devemos abrir o terminal em uma pasta contendo o arquivo yml, utilizar o utilitário `docker-compose` com os comandos `up` e/ou `down`.
```shell
docker-compose up -d
```

## Network

Por padrão, cada container docker é executado de forma completamente isolada e em sua própria network, isso significa que embora eles consigam se comunicar com o ambiente externo (ex: google.com.br), eles conseguem comunicar entre si ou com a máquina host por default.

Para que os container possam se conectar, é necessário criar uma network e conectá-los.

Embora isso possa ser feito manualmente pela cli do Docker, o docker-compose também efetua esse procedimento automáticamente para nós.

## Rebuildar imagem

Para que o docker-compose rebuilde a imagem ao invés de utilizar o cache, devemos passar a flag `--build`

Ex:
```shell
docker-compose up -d --build
```

Contudo, é importante lembra que o --build irá apenas rebuildar a imagem **sem fazer o pull** para verificar se houve atualizações na imagem, o que ocorre principalmente quando estamos utilizando a tag `latest` da imagem.

Para que a imagem seja atualizada, é necessário executá-lo explícitamente:
```shell
docker-compose pull

# ou efetuar os dois juntos
docker-compose pull && docker-compose up
```