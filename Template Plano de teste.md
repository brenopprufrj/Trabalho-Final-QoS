Assim como no documento anterior, os textos que aparecem em *azul* e entre colchetes no original (instruções de preenchimento) foram formatados como citações (`>`) para facilitar a distinção.

***

# \<Nome do Projeto> - Plano de Testes

**Versão:** <1.0.0>
**Organização:** LENS – Laboratório de Engenharia de Software

> **[Nota explicativa:]**
> Template para Plano de Testes – versão 1.0.0
> Este template foi criado para o LENS para planejar os tipos de testes a serem realizados no projeto, quando serão realizados, os responsáveis e os casos de teste para cada um desses tipos.
> Os textos que aparecem em azul, entre colchetes, são explicações para ajudar no preenchimento de cada seção e devem ser apagados no documento final.
> Preencha, nesta página inicial, o nome do projeto e a versão do documento. A versão inicial é a 1.0.0 e na medida em que vão sendo feitas revisões e alterações, documentadas na seção Histórico, a versão vai sendo alterada para 1.0.1, 1.0.2, etc.

---

## Histórico de Revisão

| Data | Versão | Descrição | Autor |
| :--- | :--- | :--- | :--- |
| | | | |
| | | | |
| | | | |

---

## Sumário

1. Introdução
2. Estratégia de Teste
3. Resultados dos Testes

---

# 1. Introdução

## 1.1 Finalidade
> [Especificar a finalidade desse documento.]

## 1.2 Escopo
> [Descrição do escopo do documento, a qual projeto está associado, o que mais é afetado por esse documento.]

## 1.3 Definições, Acrônimos, e Abreviações
> [Listar aqui todos os termos, definições, siglas, abreviações utilizados no documento, possibilitando sua correta interpretação. Essa informação pode ser substituída por uma referência ao documento de Glossário, se este foi criado para o projeto.]

## 1.4 Referências

| Título | Versão | Data | Onde pode ser obtido |
| :--- | :--- | :--- | :--- |
| | | | |
| | | | |

> [Incluir a identificação de todos os documentos referenciados neste documento. Para cada documento, indicar o título, versão (caso haja), data e onde o documento pode ser encontrado para consulta.]

## 1.5 Visão geral
> [Descrever como o restante do documento está organizado.]

---

# 2. Estratégia de Teste

> [Identificar nesta seção todos os tipos de teste a serem feitos no projeto. Para cada tipo de teste, é necessário identificar quando serão realizados, quais recursos serão necessários para sua realização, quem ficará responsável, quais requisitos devem ser submetidos a esse tipo de teste e os casos de teste.
>
> Existem momentos e níveis distintos de testes a serem realizados. No LENS, foi determinado que devem ser realizados testes de unidade, de integração, testes alfa e beta. Esses nomes indicam momentos e níveis distintos de teste. Em cada um desses momentos, podem ser executados diferentes tipos de testes, como testes funcionais, de desempenho, de stress, etc. O objetivo desse plano é identificar, de acordo com a criticidade e o tamanho do projeto, quanto investir em testes, ou seja, quais testes serão realizados em cada momento.
>
> Os tipos de testes que podem ser feitos são:
> *   testes funcionais
> *   testes de desempenho
> *   testes de stress
> *   testes de carga
> *   testes de tolerância a falhas
> *   testes de controle de acesso e segurança
> *   testes de instalação e configuração
> *   etc]

## 2.1 Teste \<tipo de teste>
> [Identificar o tipo de teste a ser realizado.]

### 2.1.1 Prazo para realização
> [Identificar quando e em que momentos esse tipo de teste será realizado.]

### 2.1.2 Recursos necessários
> [Identificar os recursos necessários para o teste, sejam humanos, de software ou de hardware. Se forem testes que envolvam um ambiente controlado pela equipe de Infraestrutura (ambiente de Homologação ou Produção), é importante repassar as informações do plano para essa equipe.]

### 2.1.3 Requisitos a serem testados
> [Identificar quais requisitos do sistema serão submetidos a esse tipo de teste. Por exemplo, pode ser que exista um requisito de desempenho relacionado a um ou dois casos de uso somente e, sendo assim, não teria sentido submeter todo o sistema ao teste.]

### 2.1.4 Casos de Teste
> [Um caso de teste descreve um cenário de teste, com os dados a serem inseridos e o resultado esperado com esses dados.]

---

# 3. Resultados dos Testes
> [Para cada tipo de teste a ser feito, em cada momento especificado, registrar os resultados dos testes com as análises feitas e as ações tomadas.]