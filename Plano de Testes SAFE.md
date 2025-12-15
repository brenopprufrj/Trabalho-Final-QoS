# Sistema SAFE - Plano de Testes

**Versão:** 1.0.0  
**Organização:** LENS – Laboratório de Engenharia de Software  
**Sistema:** SAFE (Sistema de Apoio à Biossegurança em Ambientes Físicos)

---

## Histórico de Revisão

| Data | Versão | Descrição | Autor |
| :--- | :--- | :--- | :--- |
| 2025-12-15 | 1.0.0 | Criação inicial do plano de testes | Equipe QA |

---

## Sumário

1. Introdução
2. Estratégia de Teste
   - 2.1 Testes Funcionais
   - 2.2 Testes de Requisitos Não Funcionais
3. Resultados dos Testes

---

# 1. Introdução

## 1.1 Finalidade

Este documento tem como finalidade estabelecer a estratégia de testes para o Sistema SAFE, um sistema IoT de missão crítica para monitoramento de biossegurança em ambientes físicos. O plano define os tipos de testes, recursos necessários, cronograma e casos de teste detalhados para execução por equipe externa e independente.

## 1.2 Escopo

O documento cobre o planejamento de testes para os três subsistemas do SAFE:

- **SAFE BIoT:** Hardware/Firmware de coleta de dados ambientais
- **SAFE Manager:** Backend/API de gerenciamento via MQTT
- **SAFE Dashboard:** Frontend de visualização e alertas

Este plano está associado ao projeto SAFE e afeta diretamente:
- Validação dos requisitos funcionais e não funcionais
- Garantia de qualidade para ambientes de produção
- Certificação de segurança e confiabilidade do sistema IoT

## 1.3 Definições, Acrônimos, e Abreviações

| Termo | Descrição |
| :--- | :--- |
| **SAFE** | Sistema de Apoio à Biossegurança em Ambientes Físicos |
| **BIoT** | Dispositivo IoT composto por sensores de temperatura, movimento, CO₂ e VOCs |
| **MQTT** | Message Queuing Telemetry Transport - Protocolo de mensagens para IoT |
| **Broker** | Agente intermediário (RabbitMQ) usando protocolo MQTT |
| **CO₂** | Dióxido de carbono, medido em ppm (partes por milhão) |
| **VOCs** | Compostos orgânicos voláteis, medidos em ppb (partes por bilhão) |
| **RF** | Requisito Funcional |
| **RNF** | Requisito Não Funcional |
| **E2E** | End-to-End (Teste de ponta a ponta) |
| **API** | Application Programming Interface |
| **JSON** | JavaScript Object Notation |
| **Speck** | Algoritmo de criptografia leve usado no sistema |

Para definições completas, consultar documento `SAFE_Requisitos_V2.pdf` (Seção "Informações e Variáveis").

## 1.4 Referências

| Título | Versão | Data | Onde pode ser obtido |
| :--- | :--- | :--- | :--- |
| Requisitos do Sistema SAFE | 1.0 | 2025-12 | `SAFE_Requisitos_V2.pdf` |
| Informações e Variáveis SAFE | 1.0 | 2025-12 | `SAFE_Requisitos_V2.pdf` |
| Template Plano de Teste LENS | 1.0.0 | - | `Template_para_Plano_de_Testes.pdf` |
| Template Casos de Teste LENS | 1.0 | - | `Template_CT_Portugues.pdf` |

## 1.5 Visão geral

Este documento está organizado em três seções principais:

- **Seção 2 - Estratégia de Teste:** Define os tipos de testes a serem realizados, cronograma, recursos e casos de teste para cada categoria
- **Seção 3 - Resultados dos Testes:** Área reservada para registro de resultados após execução

---

# 2. Estratégia de Teste

## Abordagem Geral

A estratégia foi desenvolvida para **maximizar a cobertura de requisitos com esforço mínimo**, seguindo os seguintes princípios:

### Princípios de Otimização

1. **Agrupamento por Fluxos E2E:** Testes que cobrem múltiplos requisitos em um único fluxo de dados (Sensor → Broker → Manager → Dashboard)
2. **Priorização de Criticidade:** Foco em funcionalidades críticas de missão (monitoramento, alertas, segurança)
3. **Testes de Limites (Boundary):** Utilização das constantes definidas como oráculos de teste
4. **Simulação Realista:** Uso de mensagens MQTT reais e simuladores de hardware quando necessário

### Fluxos Críticos Identificados

Os seguintes fluxos E2E cobrem a maior parte dos requisitos do sistema:

#### Fluxo 1: Ciclo Completo de Dados de Qualidade do Ar
**Cobertura:** BIoT RF03, RF04, RF06, RF07, RNF2, RNF10 | Manager RF13, RNF01, RNF03, RNF07, RNF16, RNF19 | Dashboard RF01, RF02, RF09, RF10, RNF01, RNF03, RNF05

**Descrição:** Coleta de dados ambientais (temp, CO₂, umidade, VOCs) → Publicação MQTT → Recepção Manager → Armazenamento BD → Disponibilização API → Exibição Dashboard

#### Fluxo 2: Contagem de Pessoas e Controle de Ocupação
**Cobertura:** BIoT RF02, RNF2, RNF11 | Manager RF13, RF15, RF16, RNF04 | Dashboard RF03, RF05

**Descrição:** Detecção entrada/saída → Atualização contador → Verificação limites → Mudança de estado instalação → Notificação usuários

#### Fluxo 3: Gerenciamento de Solicitações de Manutenção
**Cobertura:** Manager RF04, RF05, RF06, RF07, RF14

**Descrição:** Solicitação serviço → Notificação equipe → Aceite → Conclusão → Histórico

#### Fluxo 4: Gerenciamento de Solicitações de Limpeza
**Cobertura:** Manager RF08, RF09, RF10, RF11, RF14

**Descrição:** Similar ao Fluxo 3, mas para serviços de limpeza

#### Fluxo 5: Alertas e Limites de Biossegurança
**Cobertura:** BIoT RF01, RF05, RNF3, RNF4, RNF12 | Manager RNF02, RNF35 | Dashboard RF04, RF11, RF12

**Descrição:** Configuração limites Manager → Publicação Broker → Recepção BIoT → Monitoramento contínuo → Alerta visual/sonoro

---

## 2.1 Testes Funcionais

> **Objetivo:** Validar se as funcionalidades do sistema atendem aos requisitos funcionais especificados, testando os fluxos de negócio e casos de uso principais.

### 2.1.1 Testes End-to-End (E2E)

#### 2.1.1.1 Prazo para realização

**Momento:** Após integração completa dos três subsistemas  
**Duração estimada:** 3 dias  
**Frequência:** A cada versão release do sistema

#### 2.1.1.2 Recursos necessários

##### Hardware
- 1 dispositivo BIoT real (ou simulador MQTT configurado)
- Ambiente de rede local (Wi-Fi isolado para testes)
- Computador para execução do Manager
- Computador/dispositivo móvel para acesso ao Dashboard

##### Software
- Broker MQTT (RabbitMQ ou Mosquitto)
- Simulador de mensagens MQTT (MQTT.fx, MQTT Explorer ou script Python)
- Navegadores: Chrome 118+, Firefox 115+, Edge 118+, Safari 16+
- Banco de dados de teste (configurado)
- Docker (para deploy Manager/Dashboard)

##### Humanos
- 2 testadores (um para executar, um para validar)
- 1 desenvolvedor de suporte (para dúvidas técnicas)

##### Ferramentas de Simulação Requeridas
- **Simulador de Sensor:** Script Python que publica JSON no broker simulando sensores
- **Gerador de Carga:** Para simular múltiplos dispositivos BIoT simultaneamente
- **Mock de Autenticação:** Para simular diferentes papéis de usuário

#### 2.1.1.3 Requisitos Funcionais a serem testados

**Subsistema BIoT:** RF01, RF02, RF03, RF04, RF05, RF06, RF07  
**Subsistema Manager:** RF01, RF02, RF03, RF04, RF05, RF06, RF07, RF08, RF09, RF10, RF11, RF12, RF13, RF14, RF15, RF16, RF17, RF18  
**Subsistema Dashboard:** RF01, RF02, RF03, RF04, RF05, RF06, RF07, RF08, RF09, RF10, RF11, RF12

#### 2.1.1.4 Casos de Teste

Referência: Documento **"Casos de Teste SAFE.pdf"** para scripts detalhados.

**CT-E2E-01: Ciclo Completo de Dados de Qualidade do Ar**  
**Cenário:** Validar fluxo completo de coleta → transmissão → processamento → exibição de dados ambientais  
**Dados de Entrada:** JSON MQTT com temperatura=23.5°C, umidade=55%, CO₂=650ppm, VOCs=120ppb  
**Resultado Esperado:** (1) Dados armazenados no BD do Manager, (2) Dashboard exibe valores corretos em ≤15s, (3) Dados criptografados no trânsito (Speck)

**CT-E2E-02: Contagem de Pessoas e Mudança de Estado**  
**Cenário:** Simular 16 entradas em instalação com limite de 15 pessoas  
**Dados de Entrada:** 16 eventos de entrada (entry_flow=1) via MQTT  
**Resultado Esperado:** (1) Contador atualizado para 16 pessoas, (2) Estado da instalação muda para "Não Liberado (Em uso)", (3) Dashboard exibe alerta "🔴 Bloqueado", (4) Gestor recebe notificação de uso não autorizado, (5) Ao publicar 8 saídas, estado volta para "Liberado 🟢"

**CT-E2E-03: Solicitação e Conclusão de Manutenção**  
**Cenário:** Staff solicita manutenção → Equipe aceita → Gestor conclui  
**Dados de Entrada:** Solicitação de manutenção para INST-001 com motivo "Ar condicionado com defeito"  
**Resultado Esperado:** (1) Equipe de manutenção recebe e-mail, (2) Estado da instalação = "Bloqueado (Aguardando manutenção)", (3) Após conclusão, histórico registra todas as datas/responsáveis, (4) Estado volta para "Liberado"

**CT-E2E-04: Solicitação e Conclusão de Limpeza**  
**Cenário:** Staff solicita limpeza → Equipe aceita → Gestor conclui  
**Dados de Entrada:** Solicitação de limpeza para INST-002  
**Resultado Esperado:** (1) Equipe de limpeza recebe notificação, (2) Estado = "Bloqueado (Aguardando limpeza)", (3) Histórico completo armazenado, (4) Estado final = "Liberado"

**CT-E2E-05: Configuração e Alertas de Limites**  
**Cenário:** Gestor altera limite de CO₂ para 800ppm → BIoT recebe novo limite → Sensor envia valor 850ppm  
**Dados de Entrada:** Novo limite CO₂=800ppm (via Manager), leitura de sensor CO₂=850ppm  
**Resultado Esperado:** (1) BIoT atualiza limite interno em ≤30min, (2) Dashboard exibe alerta "⚠ Limite máximo atingido", (3) LED de alerta aceso no BIoT, (4) Classificação de risco atualizada no Dashboard

**CT-E2E-06: Uso Não Autorizado de Instalação**  
**Cenário:** Pessoas acessam instalação sem agendamento  
**Dados de Entrada:** Evento de entrada (entry_flow=1) em instalação sem agendamento  
**Resultado Esperado:** (1) Manager detecta ocupação não autorizada, (2) Gestor recebe notificação, (3) Evento registrado no log de auditoria

**CT-E2E-07: Exibição de Histórico no Dashboard**  
**Cenário:** Visualizar gráficos temporais da última hora  
**Dados de Entrada:** Dados históricos de INST-001 (30 pontos de intervalo de 2min)  
**Resultado Esperado:** (1) Dashboard exibe 5 gráficos lineares (temp, CO₂, umidade, VOCs, pessoas), (2) Valores máximos exibidos para cada medida, (3) Intervalo temporal = 2 minutos

---

## 2.2 Testes de Requisitos Não Funcionais

> **Objetivo:** Validar atributos de qualidade do sistema como desempenho, segurança, confiabilidade, usabilidade e manutenibilidade, garantindo que o sistema atende aos requisitos não funcionais especificados.
>
> **Nota Importante:** Os testes de requisitos não funcionais devem ser executados **APÓS** a conclusão e aprovação dos testes funcionais, garantindo que a base funcional do sistema está estável antes de avaliar seus atributos de qualidade.

### 2.2.1 Testes de Integração e Comunicação (MQTT/Broker)

#### 2.2.1.1 Prazo para realização

**Momento:** Após aprovação dos testes funcionais  
**Duração estimada:** 2 dias  
**Frequência:** A cada modificação na estrutura de mensagens MQTT ou atualização de protocolo

#### 2.2.1.2 Recursos necessários

##### Software
- Broker MQTT de teste
- Cliente MQTT (MQTT Explorer ou similar)
- Scripts de publicação/subscrição MQTT
- Analisador de rede (Wireshark) para verificar criptografia

##### Humanos
- 1 testador com conhecimento de MQTT e protocolos de comunicação

#### 2.2.1.3 Requisitos Não Funcionais a serem testados

**BIoT:** RNF1, RNF2, RNF10, RNF11, RNF15, RNF17  
**Manager:** RNF01, RNF03, RNF07, RNF08, RNF16, RNF17, RNF19

#### 2.2.1.4 Casos de Teste

Referência: Documento **"Casos de Teste SAFE.pdf"** para scripts detalhados.

**CT-INT-01: Validação de Formato JSON (SAFE_IAQ)**  
**Cenário:** Publicar mensagens MQTT com diferentes formatos JSON (válido, campo faltando, tipo errado, valor NULL, JSON inválido)  
**Dados de Entrada:** 5 payloads JSON variando de válido a completamente inválido  
**Resultado Esperado:** (1) Payload válido aceito e armazenado, (2) Payloads inválidos rejeitados com alerta "DADOS DO DISPOSITIVO [ID] EM FORMATO ERRADO"

**CT-INT-02: Validação de Formato JSON (SAFE_ENTRY_FLOW)**  
**Cenário:** Testar formato JSON para eventos de entrada/saída  
**Dados de Entrada:** entry_flow=1 (entrada), entry_flow=-1 (saída), entry_flow="entrada" (inválido)  
**Resultado Esperado:** (1) Entrada: contador +1, (2) Saída: contador -1, (3) Inválido: alerta de formato errado

**CT-INT-03: Criptografia Speck nos Dados**  
**Cenário:** Capturar pacotes MQTT com Wireshark durante transmissão de dados  
**Dados de Entrada:** BIoT publica dados no tópico SAFE_IAQ  
**Resultado Esperado:** (1) Payload do pacote MQTT criptografado (não legível em texto plano), (2) Manager descriptografa e processa corretamente

---

### 2.2.2 Testes de Limites (Boundary Testing)

#### 2.2.2.1 Prazo para realização

**Momento:** Após testes de integração e comunicação  
**Duração estimada:** 1 dia  
**Frequência:** Uma vez por release

#### 2.2.2.2 Recursos necessários

##### Software
- Simulador de tempo (para acelerar/desacelerar intervalos)
- Scripts para injetar valores extremos nos sensores

##### Humanos
- 1 testador especializado em testes de borda

#### 2.2.2.3 Requisitos Não Funcionais a serem testados

Todos os requisitos não funcionais que dependem de **constantes de tempo e valores limite** (ver tabela de variáveis no documento `SAFE_Requisitos_V2.pdf`)

**Requisitos específicos:**  
**BIoT:** RNF3, RNF4, RNF5, RNF10, RNF12  
**Manager:** RNF07, RNF15  
**Dashboard:** RNF04, RNF05, RNF06

#### 2.2.2.4 Casos de Teste

Referência: Documento **"Casos de Teste SAFE.pdf"** para scripts detalhados.

**CT-LIM-01: Timeout de Sessão**  
**Cenário:** Usuário autenticado permanece inativo por período superior a TEMPO_MAXIMO_INATIVIDADE (1 hora)  
**Dados de Entrada:** Login às T0, inatividade por 61 minutos  
**Resultado Esperado:** (1) Aos 59min: sessão ainda ativa, (2) Aos 61min: sessão encerrada automaticamente, (3) Redirecionamento para tela de login

**CT-LIM-02: Intervalo de Atualização de Dados (15s)**  
**Cenário:** Monitorar publicações do BIoT no tópico SAFE_IAQ  
**Dados de Entrada:** 10 publicações consecutivas do BIoT  
**Resultado Esperado:** Intervalo entre publicações = 15s ± 1s (INTERVALO_ATUALIZACAO_DADOS_QUALIDADE_AR)

**CT-LIM-03: Valores Extremos de CO₂**  
**Cenário:** Publicar leituras de CO₂ em valores extremos (limite configurado = 1000ppm)  
**Dados de Entrada:** CO₂ = 0ppm, 500ppm, 999ppm, 1000ppm, 5000ppm  
**Resultado Esperado:** (1) Valores ≤999ppm: aceitos sem alerta, (2) Valores ≥1000ppm: aceitos com alerta ⚠, (3) Dashboard exibe valores corretamente

**CT-LIM-06: Alerta de Falta de Dados (5 minutos)**  
**Cenário:** Interromper publicações do BIoT por período superior a TEMPO_MAXIMO_ANTES_ALERTA_DASHBOARD  
**Dados de Entrada:** Última publicação em T0, sem novas publicações  
**Resultado Esperado:** (1) T0+4min: sem alerta, (2) T0+5min: Dashboard exibe "OS DADOS NÃO ESTÃO SENDO ATUALIZADOS", (3) Ao retomar publicação: alerta desaparece

---

### 2.2.3 Testes de Segurança e Controle de Acesso

#### 2.2.3.1 Prazo para realização

**Momento:** Após testes de limites  
**Duração estimada:** 1 dia  
**Frequência:** Uma vez por release e obrigatoriamente após qualquer mudança de segurança

#### 2.2.3.2 Recursos necessários

##### Software
- Múltiplas contas de teste (uma para cada papel de usuário)
- Ferramenta de análise de requisições (Postman, Burp Suite)
- Scanner de vulnerabilidades (opcional)

##### Humanos
- 1 testador de segurança certificado ou com experiência comprovada

#### 2.2.3.3 Requisitos Não Funcionais a serem testados

**Manager:** RNF14, RNF15, RNF16, RNF17  
**BIoT:** RNF15, RNF16, RNF17, RNF19

#### 2.2.3.4 Casos de Teste

Referência: Documento **"Casos de Teste SAFE.pdf"** para scripts detalhados.

**CT-SEG-01: Login com Credenciais**  
**Cenário:** Testar autenticação com credenciais válidas e inválidas  
**Dados de Entrada:** (1) Email/senha válidos, (2) Senha errada, (3) Email inexistente, (4) Campos vazios  
**Resultado Esperado:** (1) Login bem-sucedido, (2-4) Mensagens de erro apropriadas

**CT-SEG-02: Troca de Senha no Primeiro Acesso**  
**Cenário:** Novo usuário faz primeiro login  
**Dados de Entrada:** Login com credenciais temporárias, tentativa de nova senha fraca e forte  
**Resultado Esperado:** (1) Redirecionamento obrigatório para alteração de senha, (2) Senha fraca rejeitada com critérios exibidos, (3) Senha forte aceita (>6 caracteres, números, maiúsculas, minúsculas, especiais)

**CT-SEG-04: Controle de Acesso por Papel (RBAC)**  
**Cenário:** Usuários de diferentes papéis tentam acessar recursos  
**Dados de Entrada:** Login como Administrador, Gestor, Staff, Equipe Manutenção  
**Resultado Esperado:** (1) Administrador: acesso total, (2) Gestor: acesso apenas à sua hierarquia, (3) Staff: apenas solicitações, sem configurações, (4) Tentativas de acesso não autorizado: bloqueadas

---

### 2.2.4 Testes de Confiabilidade e Tolerância a Falhas

#### 2.2.4.1 Prazo para realização

**Momento:** Após testes de segurança  
**Duração estimada:** 1 dia  
**Frequência:** Uma vez por release

#### 2.2.4.2 Recursos necessários

##### Software/Hardware
- Ferramenta para simular queda de rede (software ou controle físico do roteador)
- Simulador de queda de energia (para BIoT)
- Script para derrubar Broker temporariamente
- Ferramenta de monitoramento de uptime

##### Humanos
- 1 testador especializado em testes de confiabilidade

#### 2.2.4.3 Requisitos Não Funcionais a serem testados

**BIoT:** RNF6, RNF7, RNF8, RNF9, RNF13  
**Manager:** RNF05, RNF06, RNF09, RNF11  
**Dashboard:** RNF04, RNF08, RNF10

#### 2.2.4.4 Casos de Teste

Referência: Documento **"Casos de Teste SAFE.pdf"** para scripts detalhados.

**CT-TOL-01: Queda e Retorno de Internet (BIoT)**  
**Cenário:** BIoT perde conexão Wi-Fi e reconecta após 2 minutos  
**Dados de Entrada:** Desabilitar Wi-Fi, aguardar 2min, reabilitar Wi-Fi  
**Resultado Esperado:** (1) Durante offline: BIoT armazena dados localmente, (2) Reconexão automática em <30s, (3) Publicação de backlog + dados novos, (4) Manager recebe todos os dados

**CT-TOL-06: Dados em Formato Errado**  
**Cenário:** Publicar JSON malformado no Broker  
**Dados de Entrada:** `{temperatura: vinte, co2: abc}` (JSON inválido)  
**Resultado Esperado:** (1) Manager detecta formato inválido, (2) Alerta na tela: "DADOS DO DISPOSITIVO [ID] EM FORMATO ERRADO", (3) Erro registrado em log com timestamp

**CT-TOL-07: Disponibilidade 99,9%**  
**Cenário:** Monitoramento contínuo do Manager e Dashboard por 30 dias  
**Dados de Entrada:** Ferramenta de monitoramento de uptime  
**Resultado Esperado:** Disponibilidade ≥99,9% ao mês (downtime máximo ≈43 minutos), exceto janelas de manutenção programadas

---

### 2.2.5 Testes de Desempenho e Eficiência

#### 2.2.5.1 Prazo para realização

**Momento:** Após testes de confiabilidade  
**Duração estimada:** 1 dia  
**Frequência:** Uma vez por release e obrigatoriamente antes do deploy em produção

#### 2.2.5.2 Recursos necessários

##### Software/Hardware
- Gerador de carga (JMeter, Locust ou script customizado)
- Monitor de recursos do sistema (CPU, Memória, Rede, Disco)
- Equipamento de referência para teste de precisão de sensores

##### Humanos
- 1 testador de performance com experiência em sistemas IoT

#### 2.2.5.3 Requisitos Não Funcionais a serem testados

**BIoT:** RNF5 (Precisão 90%), RNF10, RNF13  
**Manager:** RNF07, RNF09 (Disponibilidade 99,9%)  
**Dashboard:** RNF05, RNF07

#### 2.2.5.4 Casos de Teste

Referência: Documento **"Casos de Teste SAFE.pdf"** para scripts detalhados.

**CT-PERF-01: Carga com 50 Dispositivos BIoT Simultâneos**  
**Cenário:** Simular 50 dispositivos BIoT publicando dados simultaneamente a cada 15s  
**Dados de Entrada:** Gerador de carga publicando 50 streams MQTT paralelos  
**Resultado Esperado:** (1) Manager processa todos os dados sem perda, (2) Latência de processamento <5s, (3) CPU/Memória do Manager dentro de limites aceitáveis

**CT-PERF-02: Precisão de Leitura dos Sensores (≥90%)**  
**Cenário:** Comparar 100 leituras do BIoT com equipamento de referência  
**Dados de Entrada:** 100 leituras simultâneas de temperatura, CO₂, umidade e 50 eventos de contagem de pessoas  
**Resultado Esperado:** Erro médio ≤10% para cada medida (precisão ≥90% conforme RNF5)

**CT-PERF-03: Latência de Atualização Dashboard (≤15s)**  
**Cenário:** Medir tempo entre publicação BIoT e exibição no Dashboard  
**Dados de Entrada:** Publicação de dados no Broker com timestamp  
**Resultado Esperado:** Dashboard atualiza valores em ≤15s após publicação (INTERVALO_ATUALIZAR_DADOS)

---

### 2.2.6 Testes de Usabilidade (Interface Responsiva e Compatibilidade)

#### 2.2.6.1 Prazo para realização

**Momento:** Após testes de desempenho  
**Duração estimada:** 0.5 dia  
**Frequência:** Uma vez por release

#### 2.2.6.2 Recursos necessários

##### Software/Hardware
- Dispositivos físicos: Smartphone, Tablet, Desktop
- Ferramenta de teste responsivo (Browser DevTools)
- Navegadores: Chrome 118+, Firefox 115+, Edge 118+, Safari 16+

##### Humanos
- 1 testador de UI/UX com conhecimento em design responsivo

#### 2.2.6.3 Requisitos Não Funcionais a serem testados

**Manager:** RNF13, RNF18  
**Dashboard:** RNF12, RNF13, RNF14

#### 2.2.6.4 Casos de Teste

Referência: Documento **"Casos de Teste SAFE.pdf"** para scripts detalhados.

**CT-RESP-01: Layout Pequeno (≤640px)**  
**Cenário:** Acessar Dashboard em dispositivo móvel (360x640px)  
**Dados de Entrada:** DevTools configurado para resolução mobile  
**Resultado Esperado:** (1) Cards de instalações empilhados verticalmente, (2) Menu responsivo (hamburguer), (3) Todos os elementos acessíveis e funcionais

**CT-RESP-04: Compatibilidade de Navegadores**  
**Cenário:** Acessar Manager e Dashboard em diferentes navegadores  
**Dados de Entrada:** Chrome 118+, Firefox 115+, Edge 118+, Safari 16+  
**Resultado Esperado:** (1) Dashboard carrega corretamente em todos, (2) Gráficos exibidos, (3) Alertas funcionam, (4) Login do Manager funcional

---

### 2.2.7 Testes de Manutenibilidade (Instalação e Atualização)

#### 2.2.7.1 Prazo para realização

**Momento:** Após testes de usabilidade, antes de cada deploy  
**Duração estimada:** 0.5 dia  
**Frequência:** Obrigatoriamente a cada atualização de versão

#### 2.2.7.2 Recursos necessários

##### Software/Hardware
- Ambiente limpo (máquina virtual ou container sem instalação prévia)
- Docker instalado
- Acesso ao repositório de imagens Docker

##### Humanos
- 1 testador de infraestrutura ou DevOps

#### 2.2.7.3 Requisitos Não Funcionais a serem testados

**Manager:** RNF10, RNF11, RNF12  
**Dashboard:** RNF09, RNF10, RNF11  
**BIoT:** RNF14, RNF20, RNF21

#### 2.2.7.4 Casos de Teste

**CT-INST-01: Instalação Inicial via Docker (Manager)**  
**Cenário:** Instalar Manager em ambiente limpo usando Docker  
**Dados de Entrada:** Imagem Docker do Manager, ambiente sem instalação prévia  
**Resultado Esperado:** (1) Container inicia sem erros, (2) Banco de dados criado automaticamente, (3) Manager acessível via browser, (4) Logs sem erros críticos

**CT-INST-03: Atualização de Versão sem Perda de Dados (Rollout)**  
**Cenário:** Atualizar Manager da v1.0 para v1.1 com dados existentes  
**Dados de Entrada:** Sistema v1.0 com dados de produção, nova imagem Docker v1.1  
**Resultado Esperado:** (1) Atualização bem-sucedida, (2) Sem perda de dados do BD, (3) Sistema funcional após atualização, (4) Possibilidade de rollback

**CT-INST-04: Acesso Direto ao Servidor em Caso de Falha Sistêmica**  
**Cenário:** Sistema com falha sistêmica, Administrador precisa acessar servidor  
**Dados de Entrada:** Credenciais de administrador, acesso SSH/terminal  
**Resultado Esperado:** (1) Administrador consegue acesso direto ao servidor, (2) Logs acessíveis, (3) Comandos de diagnóstico funcionais

---

# 3. Resultados dos Testes

> Esta seção será preenchida após a execução dos testes. Para cada tipo de teste, registrar:
> 
> - Data de execução
> - Testador responsável
> - Ambiente utilizado
> - Casos de teste executados
> - Taxa de sucesso/falha
> - Defeitos encontrados (IDs de rastreamento)
> - Ações corretivas tomadas
> - Observações e recomendações

**Formato sugerido:**

## 3.1 Resultados - Testes Funcionais E2E

| Data | Testador | Ambiente | Casos Executados | Sucesso | Falha | Observações |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| | | | | | | |

**Defeitos Encontrados:**
- [Lista de bugs com IDs e severidade]

**Ações Tomadas:**
- [Descrição das correções aplicadas]

---

## 3.2 Resultados - Testes de Integração MQTT

*(Repetir formato acima para cada tipo de teste)*

---

## 3.3 Resultados - Testes de Limites

---

## 3.4 Resultados - Testes de Segurança

---

## 3.5 Resultados - Testes de Tolerância a Falhas

---

## 3.6 Resultados - Testes de Desempenho

---

## 3.7 Resultados - Testes de Responsividade

---

## 3.8 Resultados - Testes de Instalação
