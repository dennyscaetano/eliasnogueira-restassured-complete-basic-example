# Exemplo Básico Completo com Rest-Assured  
[![Status das Ações](https://github.com/eliasnogueira/restassured-complete-basic-example/workflows/Build%20and%20Test/badge.svg)](https://github.com/eliasnogueira/restassured-complete-basic-example/actions)

Não esqueça de deixar uma ⭐ neste projeto!

* [Softwares Requisitos](#softwares-requisitos)  
* [Como executar os testes](#como-executar-os-testes)  
   * [Executando a API backend](#executando-a-api-backend)  
   * [Executando os test suites](#executando-os-test-suites)  
   * [Gerando o relatório de testes](#gerando-o-relatório-de-testes)  
* [Sobre a Estrutura do Projeto](#sobre-a-estrutura-do-projeto)  
* [Bibliotecas](#bibliotecas)  
* [Padrões aplicados](#padrões-aplicados)  
* [Pipeline](#pipeline)  
* [Quer ajudar?](#quer-ajudar)

Este projeto foi criado para iniciar os passos iniciais com automação de testes para uma API REST utilizando o Rest-Assured.  
Ele testa a API: [combined-credit-api](https://github.com/eliasnogueira/combined-credit-api)

> :warning: **Aviso**  
>  
> Este projeto tem objetivo educacional e **não** segue as melhores práticas que poderiam ser aplicadas.  
>
> Algumas práticas vão te ajudar a melhorar sua arquitetura de testes, mas o ponto central deste repositório é demonstrar um exemplo de execução de testes para API em um pipeline.

## Softwares Requisitos

* Java JDK 24+  
* Maven instalado e presente no seu classpath  
* Clone/baixe a API backend [combined-credit-api](https://github.com/eliasnogueira/combined-credit-api)

> :notebook: **Nota**  
>  
> Você pode usar o Java 17 se preferir

## Como executar os testes

Você pode abrir cada classe de teste em `src\test\java` e executá-las, mas recomendo que rode pela linha de comando.  
Isso nos permite executar em diferentes estratégias e também em pipelines, que é o propósito deste repositório.

### Executando a API backend

Antes de executar qualquer teste, **inicie a API backend**.  
Após clonar o projeto:

1. Navegue até a pasta do projeto via Terminal / Prompt de Comando  
2. Execute: `./mvnw spring-boot:run`  
3. Aguarde até ver algo como: _Application has started! Happy tests!_  
4. A API estará pronta e ouvindo requisições em `http://localhost:8088`

### Executando os test suites

Os test suites podem ser executados pela sua IDE ou pela linha de comando.  
Se você rodar `./mvnw test`, todos os testes serão executados porque é o ciclo padrão do Maven.

Para rodar diferentes suites com base nos grupos definidos para cada teste, use a propriedade `-Dgroups` com os nomes dos grupos.  
Exemplo para cada estágio do pipeline:

| estágio do pipeline  | comando                                 |
|----------------------|------------------------------------------|
| health check tests   | `./mvnw test -Dgroups="health"`          |
| testes de contrato   | `./mvnw test -Dgroups="contract"`        |
| testes funcionais    | `./mvnw test -Dgroups="functional"`      |
| testes e2e           | `./mvnw test -Dgroups="e2e"`             |

### Gerando o relatório de testes

Este projeto utiliza o **Allure Report** para gerar os relatórios automaticamente.  
Há algumas configurações para isso acontecer:

* configuração `aspectj` no arquivo `pom.xml`  
* arquivo `allure.properties` em `src/test/resources`

Você pode gerar o relatório via linha de comando de duas formas:

* `./mvnw allure:serve`: abre o relatório HTML diretamente no navegador  
* `./mvnw allure:report`: gera o HTML na pasta `target/site/allure-maven-plugin`

## Sobre a Estrutura do Projeto

### src/main/java

#### test  
Classe base de testes que define os aspectos iniciais para fazer requisições com RestAssured.  
Também configura o tratamento de retornos com `BigDecimal` e configurações de SSL.

#### client  
Classes que executam ações em seus respectivos endpoints. Usadas no `FullSimulationE2ETest` para demonstrar um cenário e2e.

#### commons  
Contém uma classe que formata a URL esperada ao criar um novo recurso no endpoint `simulation`.  
Você pode adicionar classes reutilizáveis aqui.

#### config  
A classe `Configuration` faz a conexão com o arquivo `api.properties` localizado em `src/test/resources/`.

O `@Config.Sources` carrega o arquivo de propriedades e associa os atributos com `@Key`, recuperando os valores automaticamente.  
Você verá duas fontes:
```java
@Config.Sources({
    "system:properties",
    "classpath:api.properties"})
```

A variável de ambiente é lida pela classe `ConfiguratorManager`, que reduz a quantidade de código para acessar informações do arquivo de propriedades.

Essa estratégia usa a biblioteca [Owner](https://matteobaccan.github.io/owner/)

#### data

##### factory  
Classes de Fábrica de Dados de Teste usando [java-faker](https://github.com/DiUS/java-faker) para gerar dados falsos, e [Lombok] com o padrão Builder.

Em alguns casos, há dados personalizados como:
 * lista de restrições e simulações existentes
 * geração de CPF
 * dados retornados pela API

##### provider  
Arguments do JUnit 5 para reduzir código e manutenção nos testes funcionais em `SimulationsFunctionalTest`.

##### suite  
Contém uma classe com os dados relacionados aos grupos de testes.

##### support  
Gerador personalizado de CPF.

#### model  
Classe de modelo e builder para [mapear objetos via serialização e desserialização](https://github.com/rest-assured/rest-assured/wiki/Usage#object-mapping) com uso do Rest-Assured.

#### specs  
Especificações de requisição e resposta usadas pelos clients e testes e2e.  
A classe `InitialStepsSpec` define `basePath`, `baseURI` e `port` para as specs.  
As classes `RestrictionsSpecs` e `SimulationsSpecs` contêm as implementações das specs.

### src/test/java

#### e2e  
Teste End-to-End usando ambos os endpoints para simular a jornada do usuário pela API.

#### general  
Teste de health check para garantir que o endpoint está disponível.

#### restrictions  
Testes de contrato e funcionais para o endpoint de restrições.

#### simulations  
Testes de contrato e funcionais para o endpoint de simulações.

### src/test/resources  
Contém a pasta `schemas` com os JSON Schemas usados nos testes de contrato.  
Também o arquivo de propriedades para configurar facilmente a URI da API.

## Bibliotecas

* [RestAssured](http://rest-assured.io/) – testar APIs REST  
* [JUnit 5](https://junit.org/junit5/) – suporte aos testes  
* [Owner](https://matteobaccan.github.io/owner/) – gerenciamento de arquivos de propriedades  
* [java-faker](https://github.com/DiUS/java-faker) – geração de dados fictícios  
* [Log4J2](https://logging.apache.org/log4j/2.x/) – estratégia de logging  
* [Allure Report](https://docs.qameta.io/allure/) – estratégia de relatório de testes

## Padrões aplicados

* Fábrica de Dados de Teste (Test Data Factory)  
* Provedor de Dados (Data Provider)  
* Builder  
* Especificações de Requisição e Resposta  
* Teste Base (Base Test)

## Pipeline

Este projeto usa o [GitHub Actions](https://github.com/features/actions) para executar todos os testes em um pipeline.  
Você pode ver o arquivo em: https://github.com/eliasnogueira/restassured-complete-basic-example/blob/master/.github/workflows/test-execution.yml

Etapas do pipeline:
```
build -> health check -> contract -> e2e -> funcional
```

Exceto pelo build (build padrão do Maven), os outros estágios usam parâmetros para determinar o tipo de teste e o SUT (Sistema sob Teste).  
Parâmetros usados:

* `-Dgroups`: especifica o tipo de teste  
* `-Dapi.base.uri`: define uma nova base URI  
* `-Dapi.base.path`: define um novo base path  
* `-Dapi.port`: define uma nova porta  
* `-Dapi.health.context`: define um novo contexto de health check

Todos os parâmetros (exceto `-Dgroups`) apontam para o Heroku, pois não conseguimos rodar localmente.  
É um ótimo exemplo de como configurar valores diferentes para rodar seus testes.

## Quer ajudar?

Por favor, leia o [guia de contribuição](CONTRIBUTING.md)