# DESAFIO DE DOCKER
Exercício para aplicar conhecimentos introdutórios de Docker

## OBJETIVO GERAL
O objetivo desse desafio é colocar em prática os conhecimentos introdutórios de Docker vistos em nosso curso introdutório, trabalhando com um cenário real e bem interessante: monitoramento de um container com uma instância do MongoDB!

## FERRAMENTAS
Quando lidamos com aplicações rodando em ambientes de produção, é necessário analisarmos como as aplicações (e as máquinas em que estão rodando) estão trabalhando: saber se um banco de dados está realizando muitas leituras em disco, saber se a rede está ficando sobrecarregada, saber se alguma aplicação está usando mais memória do que devia, etc. Duas tecnologias que são muito utilizadas para esta tarefa são:

* [Prometheus](https://prometheus.io/), uma ferramenta open source para armazenamento e consulta de séries temporais (medidas coletadas com o passar do tempo);
* [Grafana](https://grafana.com/), uma ferramenta para visualização e monitoramento de dados de séries temporais;

Uma arquitetura comum quando utilizamos essas ferramentas é a seguinte:

![Diagrama Prometheus/Grafana](https://github.com/InsightLab/docker-introduction-challenge/raw/master/Diagrama%20Prometheus_Grafana.png)

O Grafana entra na parte de visualização, com seus painéis (*dashboards*). Eles são configurados com consultas e gráficos para apresentar os dados do Prometheus. Este último, por sua vez, armazena as métricas coletadas das aplicações e responde às consultas do Grafana. Por fim, existe o **exporter**, o qual é responsável por coletar as métricas de uma aplicação e disponibilizá-las ao Prometheus. O *exporter* pode aparecer, pelo menos, de três maneiras (da esquerda para a direita): encapsulando uma aplicação, rodando dentro da aplicação ou analisando externamente a aplicação.

Para este desafio, utilizaremos o seguinte ecossistema: haverá uma instância do MongoDB rodando. O Prometheus será responsável por colher métricas dessa instância (número de inserções, consultas, memória utilizada, etc.) e o Grafana irá conter o dashboard para visualizar as métricas do Prometheus. Porém, o Prometheus não se comunica diretamente com o MongoDB, ele precisa coletar as métricas já estruturadas em um formato que ele seja capaz de compreender. Para isso, precisamos usar um exporter: um serviço responsável por analisar a instância do MongoDB e prover as métricas para o Prometheus.

## TUTORIAL

Como o foco do desafio é trabalhar com Docker, este repositório contm um pequeno arcabouço com a estrutura do Grafana e Prometheus configuradas.

Esse repositório contém um arquivo *docker-compose.yml* que traz as configurações necessárias para subir o Grafana e o Prometheus. A primeira coisa que podemos notar é que uma rede chamada **main** é definida no começo do arquivo. Precisamos atribuir um nome para ela. Crie uma rede chamada **mongo**.

Vamos ver o que temos até então: suba a aplicação utilizando o *docker-compose*. Agora, poderemos acessar a porta **9090**, que conterá a interface web do Prometheus. Não entraremos em detalhes sobre seu funcionamento, mas no cabeçalho da página, vá em **status -> targets**. Essa seção mostra as fontes de onde o Prometheus coleta suas métricas.

Opa, parece que tem algo de errado! Ele está tentando consumir métricas de um serviço chamado **mongodb-exporter** na porta **9216**, mas o serviço está **DOWN**, ou sejá, fora do ar! Desse jeito a aplicação não vai funcionar.

Precisamos subir a aplicação do exporter para podermos coletar as métricas, porém, sua imagem não está disponível publicamente! Como podemos proceder?

Bom, os desenvolvedores do *Perconalabs* disponibilizaram o código fonte de um [exporter do MongoDB para o Prometheus](https://github.com/percona/mongodb_exporter) no github. Podemos clonar esse repositório e construir uma imagem docker a partir dele! E olha que sensacional: ele já tem um **Dockerfile** E uma diretiva para construir a imagem: `make docker`. Se ficar mais fácil do que isso, perde a graça, certo?

Agora que temos a imagem (*mongodb-exporter:master*), podemos adicionar o serviço do exporter no *docker-compose.yml*. Lembre-se de informar as seguintes informações:

* rede em que o serviço irá rodar;
* nome do serviço de acordo com o esperado no Prometheus;
* porta para o Prometheus consumir;
* endereço da instância do MongoDB.

Mas espera, ainda não temos uma instância do MongoDB! Aproveite que está editando o arquivo do docker compose e adicione um serviço para o MongoDB utilizando a imagem **mongo:latest**. Uma vez que tenha seu serviço configurado, adicione o comando `--mongodb.uri=mongodb://<nome do serviço do mongo>:27017` na configuração do exporter. Lembre-se de checar se todas as informações (portas, rede, etc.) estão corretas!

Agora que está tudo pronto, vamos subir o docker compose novamente e verificar o status no Prometheus!

Perfeito! O exporter está funcionando! Se quiser, acesse a porta 9216 para ver como que o exporter envia as métricas. Agora vamos ao Grafana!

Na porta 3000, podemos acessar o Grafana utilizando o usuário **admin** e deixando o **campo de senha em branco**. Assim que entramos, podemos identificar um símbolo em forma de + no painel esquerdo. Nele, selecionamos a opção **Import**. Agora, precisamos carregar nosso dashboard (o qual já está corretamente configurado no repositório) no Grafana. Existem 3 maneiras de fazer isso:

* Informando o ID do dashboard, caso esteja disponível no catálogo aberto do Grafana;
* Colando o JSON de um dashboard gerado pelo Grafana;
* Importando o arquivo JSON de um painel.

No repositório, já contém um arquivo chamado MongoDB-Dashboard.json com nosso dashboard configurado. Basta importá-lo!

Pronto! Agora conseguimos monitorar nossa instância do MongoDB!

**TAREFA EXTRA:** faça o mongo exporter monitorar uma intância do MongoDB que esteja rodando em uma máquina diferente!

