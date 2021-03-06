Esse exemplo contempla como seria a configuração de uma aplicação com imagem docker em React para desenvolvimento e produção, com CI/CD no Travis e deployment para o ElasticBeanstalk na AWS.

Quando em desenvolvimento, queremos que atualizações no código fonte, atualizem automáticamente a aplicação (como seria se estivéssemos executando o npm start no local), para isso acontecer, precisamos "linkar" o código fonte do nosso computador com o container, através da flag `-v` de volume.

Além disso, queremos que a bateria de teste também execute em container.

Quando em produção, queremos enviar todo nosso código fonte para o container de forma que ele fique embutido na imagem.

## Executando como dev

Irá rodar a aplicação com o auto-reload funcionando, instalar a aplicação e executar a bateria de teste.

```shell
docker-compose up
```

Infelizmente não irá executar os testes automáticamente, quando uma alteração for efetuada.

## Executando em produção

O arquivo `Dockerfile` é utilizado para executar o sistema com as configurações de produção. Para isso, é quebrado o processo com uma técnica de `multistage build` (perceba os dois FROM no arquivo), o motivo disso é que precisamos do node e outras dependências apenas durante o processo de desenvolvimento, para o sistema executar em produção, tudo que ele precisa é um arquivo `.html` e `.js`, portanto, não faz sentido subirmos para produção uma imagem com dependências desnecessárias.

Utilizando o `multistage build` podemos puxar temporariamente uma imagem do node, efetuar o processo de compilação da aplicação, e gerar a imagem de produção apenas com o que realmente queremos: os arquivos gerados pelo build, e o servidor `nginx` para servi-lo.

Perceba que para servir o sistema em produção, utilizamos o `nginx` como Webserver da aplicação. O servidor node utilizado na imagem dev é apenas para fins de desenvolvimento.

Para execução:

```shell
docker build -t rogerzanelato/react-app  .
docker run -p 8085:80 rogerzanelato/react-app
```

## Travis CI

Os steps definidos no yml de configuração do travis esperam que os comandos retornem um status code com o resultado, lembrando das regras de error code do Linux (0 sucesso / 1-255 erro).

Ao executar os testes como normalmente fazemos não há status codes retornados, e a execução fica pausada esperando novos comandos. Para que seja retornado o resultado, precisamos passar as flags `-- --coverage` para gerar o coverage, acompanhado da varávelmente de ambiente `-e CI=true`, assim será retornando 0 quando nenhum teste quebrar, e um error code caso quebre.

```shell
docker run -e CI=true rogerzanelato/docker-react npm run test -- --coverage
```

## Deployment AWS

Antigamente o Elastic Beanstalk dava preferência à procurar um `Dockerfile` na raiz do projeto para efetuar deploy da aplicação. Porém, hoje a AWS possuí suporte completo à `docker-compose` e portanto ele primeiro procura um `docker-compose` na raíz do projeto, para em sequência procuar por um `Dockerfile`.

Para que não tenhamos problema com isso (em vista que nosso docker-compose é para desenvolvimento local), ao selecionar o ambiente de execução na AWS, precisamos marcar a opção: **Docker running on 64bit Amazon Linux**

No lugar da "Docker running on 64bit Amazon Linux 2".

Para que o CD funcione, é necessário criar um usuário IAM com a permissão AdministratorAccess-AWSElasticBeanstalk, gerar uma chave de acesso, e configurá-las nas seguintes variáveis de ambeinte no Travis:
- AWS_ACCESS_KEY
- AWS_SECRET_KEY

Obs: Esse modelo de implantar aplicação em produção não é recomendável, pois dependemos da AWS gerar a imagem do nosso Dockerfile e executá-la, dificultando a possibilidade de migrar de cloud, e processamento desnecessário. Uma forma mais recomendada, seria gerar a imagem pelo CI, publicá-la no docker.hub (ou outro lugar), e deixar para a cloud apenas fazer pull da imagem, e executar a aplicação (conforme feito no repositório fib-multiplos-container-aws-travis).

## Disclaimers

1. 
A seguinte linha no docker-compose:
volumes:
    - /app/node_modules

Serve para "favoritar" a pasta `node_modules` dentro do container de forma que ela não seja sobrescrita pelo nosso mapeamento da pasta local ao container. Caso essa linha seja removida, o container iria entender que deveríamos usar a pasta `node_modules` do nosso local, e como ela não existe, a aplicação não iria funcionar.

2. 
A seguinte linha foi necessária para o autoreload do container funcionar:
    environment:
      - CHOKIDAR_USEPOLLING=true