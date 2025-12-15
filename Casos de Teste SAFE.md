# Sistema SAFE - Casos de Teste

**Versão:** 1.0  
**Organização:** LENS – Laboratório de Engenharia de Software  
**Data:** 2025-12-15

---

## Histórico de Revisões

| Data | Versão | Descrição | Autor |
| :--- | :--- | :--- | :--- |
| 2025-12-15 | 1.0 | Criação inicial dos casos de teste | Equipe QA |

---

## Índice

1. Introdução
2. Ambiente de Teste
3. Informações de Configuração (Pré-condições gerais)
4. Casos de Teste Funcionais (E2E)
5. Casos de Teste de Requisitos Não Funcionais

---

# 1. Introdução

Este documento especifica os casos de teste para o Sistema SAFE (Sistema de Apoio à Biossegurança em Ambientes Físicos). Os casos foram elaborados para execução por equipe externa e independente, contendo passos detalhados, valores de entrada específicos (incluindo payloads JSON MQTT), e resultados esperados precisos baseados nas constantes do sistema.

## 1.1 Definições, Acrônimos e Abreviações

| Termo | Descrição |
| :--- | :--- |
| **SAFE** | Sistema de Apoio à Biossegurança em Ambientes Físicos |
| **BIoT** | Dispositivo IoT de coleta de dados ambientais |
| **MQTT** | Protocolo de mensagens para IoT |
| **Broker** | RabbitMQ - intermediário de mensagens MQTT |
| **ppm** | Partes por milhão (unidade de CO₂) |
| **ppb** | Partes por bilhão (unidade de VOCs) |
| **E2E** | End-to-End (ponta a ponta) |
| **RBAC** | Role-Based Access Control |

## 1.2 Referências

| Título | Versão | Data | Onde pode ser obtido |
| :--- | :--- | :--- | :--- |
| Requisitos do Sistema SAFE | 1.0 | 2025-12 | `SAFE_Requisitos_V2.pdf` |
| Informações e Variáveis SAFE | 1.0 | 2025-12 | `SAFE_Requisitos_V2.pdf` |
| Template Plano de Teste LENS | 1.0.0 | - | `Template_para_Plano_de_Testes.pdf` |
| Template Casos de Teste LENS | 1.0 | - | `Template_CT_Portugues.pdf` |
| Plano de Testes SAFE | 1.0.0 | 2025-12-15 | `Plano de Testes SAFE.pdf` |

---

# 2. Ambiente de Teste

## 2.1 Ambiente Integrado (E2E)

| Item | Detalhes | Configuração Adicional |
| :--- | :--- | :--- |
| **Máquina Manager** | Servidor Docker | **Diretório BD:** /var/lib/postgresql/data |
| **Máquina Dashboard** | Servidor Web Docker | **BD:** PostgreSQL 14+ |
| **Broker MQTT** | RabbitMQ 3.11+ com plugin MQTT | **Porta:** 1883 (sem TLS), 8883 (TLS) |
| **Dispositivo BIoT** | NodeMCU ESP32 ou Simulador Python | **MAC Address:** registrado no Manager |
| **Client Browser** | Chrome 118+ / Firefox 115+ | **Resolução:** 1920x1080, 768x1024, 360x640 |
| **Testador** | [Nome] | **Data:** [dd/mm/yyyy] |
| **Status** | | (Sucesso/Falha) |

## 2.2 Ambiente de Simulação MQTT

| Item | Detalhes | Configuração Adicional |
| :--- | :--- | :--- |
| **Máquina** | Cliente MQTT (MQTT Explorer ou script Python) | **Broker:** localhost:1883 |
| **Credenciais** | user: biot_test / pwd: [configurado] | **TLS:** Habilitado |
| **Tópicos** | SAFE_IAQ, SAFE_ENTRY_FLOW, SAFE | |

---

# 3. Informações de Configuração (Pré-condições gerais)

## 3.1 Dados de Teste Padrão

### Usuários de Teste

| Papel | Login | Senha | Permissões |
| :--- | :--- | :--- | :--- |
| Administrador | admin@safe.br | Admin@2025! | Todas |
| Gestor Instalação | gestor@ct.ufrj.br | Gestor@2025! | Gerenciar instalações CT |
| Staff | professor@ct.ufrj.br | Prof@2025! | Solicitar uso/manutenção |
| Equipe Manutenção | manut@safe.br | Manut@2025! | Aceitar/concluir manutenção |
| Equipe Limpeza | limpeza@safe.br | Limpeza@2025! | Aceitar/concluir limpeza |

### Instalações de Teste

| ID | Nome | Localização | Tipo | Dispositivo BIoT |
| :--- | :--- | :--- | :--- | :--- |
| INST-001 | Lab Química | CT -> Bloco A -> Andar 2 | Laboratório | BIOT-MAC-001 |
| INST-002 | Sala 201 | CT -> Bloco B -> Andar 2 | Sala de aula | BIOT-MAC-002 |
| INST-003 | Auditório Principal | CT -> Bloco C -> Térreo | Auditório | BIOT-MAC-003 |

### Limites de Biossegurança Padrão (Lab Química)

```json
{
  "idBIoT": "BIOT-MAC-001",
  "temperaturaMin": 18,
  "temperaturaMax": 26,
  "umidadeMin": 30,
  "umidadeMax": 70,
  "co2Max": 1000,
  "numeroPessoasMin": 0,
  "numeroPessoasMax": 15
}
```

## 3.2 Constantes de Tempo para Validação

| Constante | Valor | Uso no Teste |
| :--- | :--- | :--- |
| INTERVALO_ATUALIZACAO_DADOS_QUALIDADE_AR | 15s | Verificar publicação periódica |
| INTERVALO_ATUALIZACAO_LIMITES | 30min | Verificar refresh de limites |
| TEMPO_MAXIMO_INATIVIDADE | 1h | Teste de timeout de sessão |
| TEMPO_MAXIMO_ANTES_ALERTA_DASHBOARD | 5min | Alerta de falta de dados |
| INTERVALO_ATUALIZAR_DADOS (Manager) | 15s | Polling do Manager |

---

# 4. Casos de Teste Funcionais (End-to-End)

> **Objetivo:** Validar as funcionalidades do sistema através de fluxos completos de ponta a ponta, garantindo que os requisitos funcionais são atendidos.

## 4.1 Caso de Teste E2E-01: Ciclo Completo de Dados de Qualidade do Ar

### 4.1.1 Descrição

Validar o fluxo completo desde a coleta de dados ambientais pelo BIoT, passando pelo Broker e Manager, até a exibição no Dashboard.

**Cobertura de Requisitos:**  
BIoT: RF03, RF04, RF06, RF07, RNF2, RNF10, RNF17  
Manager: RF13, RNF01, RNF03, RNF07, RNF16, RNF19  
Dashboard: RF01, RF02, RF09, RF10, RNF01, RNF03, RNF05

### 4.1.2 Pré-condições

- Broker MQTT ativo e acessível
- Manager conectado ao Broker e ao Banco de Dados
- Dashboard conectado à API do Manager
- Dispositivo BIoT BIOT-MAC-001 registrado no sistema
- Instalação INST-001 cadastrada com limites configurados

### 4.1.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Publicar no Broker (tópico SAFE_IAQ) o JSON:<br>`{"idBIoT":"BIOT-MAC-001","timestamp":"2025-12-15T14:30:00Z","temperatura":23.5,"umidade":55,"co2":650,"vocs":120,"erro":0}` | Mensagem recebida pelo Broker | | | | |
| 2 | Aguardar 15 segundos (INTERVALO_ATUALIZAR_DADOS) | Manager deve coletar a mensagem | | | | |
| 3 | Consultar Banco de Dados (tabela sensor_data) | Registro inserido com idBIoT=BIOT-MAC-001, temp=23.5, umidade=55, co2=650, vocs=120 | | | | |
| 4 | Abrir Dashboard e acessar card da instalação INST-001 | Exibir: Temp=23.5°C, CO₂=650ppm, Umidade=55%, VOCs=120ppb | | | | |
| 5 | Verificar timestamp da última atualização no Dashboard | Timestamp deve ser ≤15s do horário atual | | | | |
| 6 | Verificar criptografia: capturar pacote MQTT com Wireshark | Dados devem estar criptografados (protocolo Speck) | | | | |

**Status Final:** (Sucesso/Falha)

### 4.1.4 Conjunto de valores

| | Cenário 1 (Valores normais) | Cenário 2 (Temp alta) | Cenário 3 (CO₂ alto) | Cenário 4 (Umidade baixa) | Cenário 5 (VOCs alto) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Temperatura (°C)** | 23.5 | 27 | 22 | 20 | 24 |
| **Umidade (%)** | 55 | 50 | 60 | 25 | 65 |
| **CO₂ (ppm)** | 650 | 700 | 1200 | 500 | 600 |
| **VOCs (ppb)** | 120 | 100 | 150 | 80 | 500 |
| **Resultado Esperado** | Todos valores exibidos corretamente, sem alertas | Temp exibida 27°C, sem alertas (dentro do limite 18-26°C se configurado) | Alerta ⚠ para CO₂ (limite padrão 1000ppm excedido) | Alerta ⚠ para umidade (limite mín padrão 30% não atingido) | Dados exibidos com VOCs=500ppb |
| **Resultado Obtido** | | | | | |
| **Sucesso/Falha** | | | | | |
| **Nº Ambiente** | | | | | |
| **Nº Log** | | | | | |

---

## 4.2 Caso de Teste E2E-02: Contagem de Pessoas e Mudança de Estado

### 4.2.1 Descrição

Validar a contabilização de entrada/saída de pessoas e a mudança de estado da instalação quando o limite é atingido.

**Cobertura de Requisitos:**  
BIoT: RF02, RNF2, RNF11  
Manager: RF13, RF15, RF16, RNF04  
Dashboard: RF03, RF05

### 4.2.2 Pré-condições

- Sistema operacional (Broker, Manager, Dashboard)
- Instalação INST-001 configurada com limite de ocupação = 15 pessoas
- Estado inicial da instalação = "Liberado"
- Número de pessoas inicial = 0

### 4.2.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Publicar 10 eventos de entrada no tópico SAFE_ENTRY_FLOW:<br>`{"idBIoT":"BIOT-MAC-001","timestamp":"2025-12-15T14:35:00Z","entry_flow":1,"erro":0}` | Manager incrementa contador para 10 pessoas | | | | |
| 2 | Verificar Dashboard card INST-001 | Exibir "Pessoas: 10" e estado "Liberado" (🟢) | | | | |
| 3 | Publicar 6 eventos de entrada adicionais | Contador atualizado para 16 pessoas | | | | |
| 4 | Verificar estado da instalação no Manager | Estado muda para "Não Liberado (Em uso)" | | | | |
| 5 | Verificar Dashboard | Exibir "Pessoas: 16" e estado "Bloqueado" (🔴) | | | | |
| 6 | Verificar alertas no Dashboard | Alerta visual: "⚠ Limite máximo atingido" para ocupação | | | | |
| 7 | Verificar notificação para gestor | Gestor recebe notificação de "Uso não autorizado" (RF16 Manager) | | | | |
| 8 | Publicar 8 eventos de saída:<br>`{"idBIoT":"BIOT-MAC-001","timestamp":"...","entry_flow":-1,"erro":0}` | Contador volta para 8 pessoas | | | | |
| 9 | Verificar estado no Dashboard | Estado volta para "Liberado" (🟢) | | | | |

**Status Final:** (Sucesso/Falha)

### 4.2.4 Conjunto de valores

| | Cenário 1 (Abaixo limite) | Cenário 2 (No limite) | Cenário 3 (Acima limite) | Cenário 4 (Muito acima) | Cenário 5 (Volta ao normal) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Nº de Entradas** | 10 | 15 | 16 | 25 | 8 (saídas de 16) |
| **Limite Máx** | 15 | 15 | 15 | 15 | 15 |
| **Pessoas Totais** | 10 | 15 | 16 | 25 | 8 |
| **Resultado Esperado** | Estado "Liberado" 🟢, Pessoas: 10 | Estado "Liberado" 🟢, Pessoas: 15 (ainda no limite) | Estado "Bloqueado" 🔴, Pessoas: 16, Alerta "⚠ Limite máximo atingido", Notificação para gestor | Estado "Bloqueado" 🔴, Pessoas: 25, Alerta mantido | Estado "Liberado" 🟢, Pessoas: 8 |
| **Resultado Obtido** | | | | | |
| **Sucesso/Falha** | | | | | |
| **Nº Ambiente** | | | | | |
| **Nº Log** | | | | | |

---

## 4.3 Caso de Teste E2E-03: Solicitação e Conclusão de Manutenção

### 4.3.1 Descrição

Validar o ciclo completo de solicitação, aceite e conclusão de manutenção.

**Cobertura de Requisitos:**  
Manager: RF04, RF05, RF06, RF07, RF14

### 4.3.2 Pré-condições

- Usuário Staff autenticado (professor@ct.ufrj.br)
- Usuário Equipe Manutenção autenticado (manut@safe.br)
- Usuário Gestor autenticado (gestor@ct.ufrj.br)
- Instalação INST-001 cadastrada

### 4.3.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Staff acessa Manager e solicita manutenção para INST-001:<br>Motivo: "Ar condicionado com defeito" | Solicitação criada com status "Aguardando atendimento" | | | | |
| 2 | Verificar envio de e-mail | Equipe de manutenção recebe e-mail de notificação (RF05) | | | | |
| 3 | Verificar estado da instalação no Dashboard | Estado = "Bloqueado (Aguardando manutenção)" 🔴 | | | | |
| 4 | Equipe Manutenção acessa Manager e aceita a solicitação | Status muda para "Em andamento" | | | | |
| 5 | Gestor da instalação marca manutenção como concluída | Status muda para "Atendida" | | | | |
| 6 | Verificar histórico no Manager | Registro contém: nome solicitante (Staff), data/hora solicitação, nome atendente, data/hora atendimento, data/hora conclusão, instalação INST-001 (RF14) | | | | |
| 7 | Verificar estado da instalação no Dashboard | Estado volta para "Liberado" 🟢 (se não houver outros bloqueios) | | | | |

**Status Final:** (Sucesso/Falha)

---

## 4.4 Caso de Teste E2E-04: Solicitação e Conclusão de Limpeza

### 4.4.1 Descrição

Validar o ciclo completo de solicitação, aceite e conclusão de limpeza.

**Cobertura de Requisitos:**  
Manager: RF08, RF09, RF10, RF11, RF14

### 4.4.2 Pré-condições

- Usuário Staff autenticado
- Usuário Equipe Limpeza autenticado (limpeza@safe.br)
- Usuário Gestor autenticado
- Instalação INST-002 cadastrada

### 4.4.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Staff solicita limpeza para INST-002 | Solicitação criada com status "Aguardando atendimento" | | | | |
| 2 | Verificar notificação | Equipe de limpeza recebe notificação (RF09) | | | | |
| 3 | Verificar Dashboard | Estado = "Bloqueado (Aguardando limpeza)" 🔴 | | | | |
| 4 | Equipe Limpeza aceita | Status = "Em andamento" | | | | |
| 5 | Gestor marca como concluída | Status = "Atendida" | | | | |
| 6 | Verificar histórico | Registro completo armazenado (RF14) | | | | |
| 7 | Verificar Dashboard | Estado = "Liberado" 🟢 | | | | |

**Status Final:** (Sucesso/Falha)

---

## 4.5 Caso de Teste E2E-05: Configuração e Alertas de Limites

### 4.5.1 Descrição

Validar a configuração de limites pelo Manager, propagação para BIoT via Broker, e acionamento de alertas.

**Cobertura de Requisitos:**  
BIoT: RF01, RF05, RNF3, RNF4, RNF12  
Manager: RNF02, RNF35  
Dashboard: RF04, RF11, RF12

### 4.5.2 Pré-condições

- Gestor autenticado no Manager
- Dispositivo BIoT-MAC-001 online
- Instalação INST-001 cadastrada

### 4.5.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Gestor altera limite de CO₂ da INST-001 para 800ppm (máx) | Novo limite salvo no Manager | | | | |
| 2 | Manager publica no Broker (tópico SAFE):<br>`{"idBIoT":"BIOT-MAC-001","temperaturaMin":18,"temperaturaMax":26,"umidadeMin":30,"umidadeMax":70,"co2Max":800,"numeroPessoasMin":0,"numeroPessoasMax":15}` | Mensagem publicada | | | | |
| 3 | Aguardar até 30 minutos (INTERVALO_ATUALIZACAO_LIMITES) | BIoT consome a mensagem e atualiza limites internos (RF01 BIoT) | | | | |
| 4 | Publicar leitura de CO₂ = 850ppm (acima do limite):<br>`{"idBIoT":"BIOT-MAC-001","timestamp":"...","temperatura":24,"umidade":60,"co2":850,"vocs":100,"erro":0}` | Manager processa dados | | | | |
| 5 | Verificar Dashboard | Alerta visual "⚠ Limite máximo atingido" para CO₂ (RF11) | | | | |
| 6 | Verificar legenda do Dashboard | Exibir legenda: 🔴 Bloqueado, 🟢 Liberado, ⚠ Limite máximo atingido (RF12) | | | | |
| 7 | Verificar painel sinalizador físico do BIoT | LED de alerta aceso (RF05 BIoT) | | | | |
| 8 | Verificar classificação de risco no Dashboard | Exibir classificação baseada no Guia de Biossegurança (RF04 Dashboard) | | | | |

**Status Final:** (Sucesso/Falha)

### 4.5.4 Conjunto de valores

| | Cenário 1 (CO₂ normal) | Cenário 2 (CO₂ limite) | Cenário 3 (Temp alta) | Cenário 4 (Umidade baixa) | Cenário 5 (Múltiplos limites) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Limite CO₂ (ppm)** | 800 | 800 | 1000 | 1000 | 800 |
| **Limite Temp (°C)** | 18-26 | 18-26 | 18-25 | 18-26 | 18-24 |
| **Limite Umidade (%)** | 30-70 | 30-70 | 30-70 | 40-70 | 35-65 |
| **Leitura CO₂** | 750 | 850 | 600 | 500 | 850 |
| **Leitura Temp** | 23 | 24 | 26 | 22 | 25 |
| **Leitura Umidade** | 55 | 60 | 55 | 35 | 32 |
| **Resultado Esperado** | Sem alertas, valores normais | Alerta ⚠ CO₂ (850 > 800), LED BIoT aceso | Alerta ⚠ Temp (26 > 25), classificação de risco ajustada | Alerta ⚠ Umidade (35 < 40) | Alertas para CO₂ e Umidade |
| **Resultado Obtido** | | | | | |
| **Sucesso/Falha** | | | | | |
| **Nº Ambiente** | | | | | |
| **Nº Log** | | | | | |

---

## 4.6 Caso de Teste E2E-06: Uso Não Autorizado de Instalação

### 4.6.1 Descrição

Validar notificação de uso não autorizado quando pessoas acessam instalação sem agendamento.

**Cobertura de Requisitos:**  
Manager: RF16

### 4.6.2 Pré-condições

- Instalação INST-001 sem agendamento para o horário atual
- Nenhuma solicitação de uso aprovada
- Gestor autenticado e com notificações habilitadas

### 4.6.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Verificar agenda da INST-001 | Nenhum agendamento ativo | | | | |
| 2 | Publicar evento de entrada de pessoas:<br>`{"idBIoT":"BIOT-MAC-001","timestamp":"...","entry_flow":1,"erro":0}` | Manager detecta ocupação sem autorização | | | | |
| 3 | Verificar notificações do gestor | Gestor recebe notificação de "Uso não autorizado de INST-001" (RF16) | | | | |
| 4 | Verificar log de auditoria | Evento registrado com timestamp e instalação | | | | |

**Status Final:** (Sucesso/Falha)

---

## 4.7 Caso de Teste E2E-07: Exibição de Histórico no Dashboard

### 4.7.1 Descrição

Validar a exibição de gráficos temporais de evolução dos parâmetros ambientais.

**Cobertura de Requisitos:**  
Dashboard: RF07, RF08

### 4.7.2 Pré-condições

- Dashboard operacional
- Dados históricos de INST-001 para última hora
- Intervalo de coleta: 2 minutos (30 pontos na última hora)

### 4.7.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Acessar Dashboard e selecionar INST-001 | Card da instalação exibido | | | | |
| 2 | Visualizar gráfico de temperatura (última hora) | Gráfico linear temporal com intervalo de 2min (RF07) | | | | |
| 3 | Visualizar gráfico de CO₂ (última hora) | Gráfico linear temporal com intervalo de 2min | | | | |
| 4 | Visualizar gráfico de umidade (última hora) | Gráfico linear temporal com intervalo de 2min | | | | |
| 5 | Visualizar gráfico de VOCs (última hora) | Gráfico linear temporal com intervalo de 2min | | | | |
| 6 | Visualizar gráfico de pessoas (última hora) | Gráfico linear temporal com intervalo de 2min | | | | |
| 7 | Verificar valores máximos exibidos | Dashboard mostra: Temp Máx=XX°C, CO₂ Máx=XXXppm, Umidade Máx=XX%, VOCs Máx=XXppb, Pessoas Máx=XX (RF08) | | | | |

**Status Final:** (Sucesso/Falha)

---

# 5. Casos de Teste de Requisitos Não Funcionais

> **Objetivo:** Validar os atributos de qualidade do sistema, incluindo desempenho, segurança, confiabilidade, usabilidade e manutenibilidade.
>
> **⚠ IMPORTANTE:** Estes testes devem ser executados **APÓS** a conclusão e aprovação de todos os testes funcionais (Seção 4). A base funcional do sistema deve estar estável antes de avaliar seus requisitos não funcionais.

## 5.1 Testes de Integração e Comunicação

### 5.1.1 Caso de Teste INT-01: Validação de Formato JSON (SAFE_IAQ)

#### 5.1.1.1 Descrição

Validar que o Manager aceita apenas mensagens MQTT no formato JSON correto para SAFE_IAQ.

**Cobertura de Requisitos:**  
BIoT: RNF2  
Manager: RNF19, RNF06

#### 5.1.1.2 Pré-condições

- Broker MQTT ativo
- Manager conectado e escutando tópico SAFE_IAQ

#### 5.1.1.3 Conjunto de valores

| | Cenário 1 (Válido) | Cenário 2 (Campo faltando) | Cenário 3 (Tipo errado) | Cenário 4 (Valor NULL) | Cenário 5 (JSON inválido) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Payload MQTT** | `{"idBIoT":"BIOT-MAC-001","timestamp":"2025-12-15T14:30:00Z","temperatura":23.5,"umidade":55,"co2":650,"vocs":120,"erro":0}` | `{"idBIoT":"BIOT-MAC-001","timestamp":"2025-12-15T14:30:00Z","temperatura":23.5,"umidade":55,"vocs":120,"erro":0}` (sem co2) | `{"idBIoT":"BIOT-MAC-001","timestamp":"2025-12-15T14:30:00Z","temperatura":"vinte","umidade":55,"co2":650,"vocs":120,"erro":0}` | `{"idBIoT":"BIOT-MAC-001","timestamp":"2025-12-15T14:30:00Z","temperatura":null,"umidade":55,"co2":650,"vocs":120,"erro":0}` | `{idBIoT:BIOT-MAC-001,temperatura:23.5}` |
| **Resultado Esperado** | Dados aceitos e armazenados | Manager exibe alerta "DADOS DO DISPOSITIVO BIOT-MAC-001 EM FORMATO ERRADO" (RNF06) | Manager exibe alerta "DADOS DO DISPOSITIVO BIOT-MAC-001 EM FORMATO ERRADO" | Manager exibe alerta "DADOS DO DISPOSITIVO BIOT-MAC-001 EM FORMATO ERRADO" | Manager exibe alerta "DADOS DO DISPOSITIVO BIOT-MAC-001 EM FORMATO ERRADO" |
| **Resultado Obtido** | | | | | |
| **Sucesso/Falha** | | | | | |
| **Nº Ambiente** | | | | | |
| **Nº Log** | | | | | |

---

### 5.1.2 Caso de Teste INT-02: Validação de Formato JSON (SAFE_ENTRY_FLOW)

#### 5.1.2.1 Descrição

Validar formato JSON para tópico de contagem de pessoas.

**Cobertura de Requisitos:**  
BIoT: RNF2  
Manager: RNF19

#### 5.1.2.2 Pré-condições

- Broker MQTT ativo
- Manager conectado

#### 5.1.2.3 Conjunto de valores

| | Cenário 1 (Entrada) | Cenário 2 (Saída) | Cenário 3 (Inválido) |
| :--- | :--- | :--- | :--- |
| **Payload** | `{"idBIoT":"BIOT-MAC-001","timestamp":"2025-12-15T14:30:00Z","entry_flow":1,"erro":0}` | `{"idBIoT":"BIOT-MAC-001","timestamp":"2025-12-15T14:30:00Z","entry_flow":-1,"erro":0}` | `{"idBIoT":"BIOT-MAC-001","entry_flow":"entrada"}` |
| **Resultado Esperado** | Contador +1 | Contador -1 | Alerta de formato errado |
| **Resultado Obtido** | | | |
| **Sucesso/Falha** | | | |

---

### 5.1.3 Caso de Teste INT-03: Criptografia Speck nos Dados

#### 5.1.3.1 Descrição

Validar que os dados transmitidos entre BIoT e Manager são criptografados com protocolo Speck.

**Cobertura de Requisitos:**  
BIoT: RNF17  
Manager: RNF16, RNF17

#### 5.1.3.2 Pré-condições

- BIoT autenticado no Broker
- Wireshark instalado para captura de pacotes
- Chaves de criptografia configuradas

#### 5.1.3.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Iniciar captura de pacotes com Wireshark na interface de rede do BIoT | Captura iniciada | | | | |
| 2 | BIoT publica dados no tópico SAFE_IAQ | Pacote MQTT capturado | | | | |
| 3 | Analisar payload do pacote MQTT | Dados do payload estão criptografados (não legíveis em texto plano) | | | | |
| 4 | Verificar Manager | Manager descriptografa e processa dados corretamente | | | | |

**Status Final:** (Sucesso/Falha)

---

## 5.2 Testes de Limites (Boundary Testing)

### 5.2.1 Caso de Teste LIM-01: Timeout de Sessão

#### 5.2.1.1 Descrição

Validar que o Manager encerra sessão após tempo máximo de inatividade.

**Cobertura de Requisitos:**  
Manager: RNF15

#### 5.2.1.2 Pré-condições

- Usuário Staff autenticado no Manager
- TEMPO_MAXIMO_INATIVIDADE = 1 hora (3600 segundos)

#### 5.2.1.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Usuário faz login no Manager | Sessão iniciada | | | | |
| 2 | Anotar timestamp do login | T0 = [timestamp] | | | | |
| 3 | Não realizar nenhuma ação por 59 minutos | Sessão ainda ativa | | | | |
| 4 | Tentar acessar página interna do Manager | Acesso permitido (sessão ainda válida) | | | | |
| 5 | Não realizar ação por mais 2 minutos (total 61min) | Sessão encerrada automaticamente (RNF15) | | | | |
| 6 | Tentar acessar página interna | Redirecionado para tela de login | | | | |

**Status Final:** (Sucesso/Falha)

---

### 5.2.2 Caso de Teste LIM-02: Intervalo de Atualização de Dados (15s)

#### 5.2.2.1 Descrição

Validar que o BIoT publica dados de qualidade do ar a cada 15 segundos.

**Cobertura de Requisitos:**  
BIoT: RNF10

#### 5.2.2.2 Pré-condições

- BIoT operacional
- Cliente MQTT subscrito no tópico SAFE_IAQ

#### 5.2.2.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Inscrever-se no tópico SAFE_IAQ com MQTT Explorer | Inscrição realizada | | | | |
| 2 | Anotar timestamp da primeira mensagem recebida | T1 = [timestamp] | | | | |
| 3 | Aguardar próxima mensagem | T2 = [timestamp] | | | | |
| 4 | Calcular intervalo: T2 - T1 | Intervalo = 15s ± 1s (RNF10) | | | | |
| 5 | Repetir para 10 mensagens consecutivas | Todos os intervalos = 15s ± 1s | | | | |

**Status Final:** (Sucesso/Falha)

---

### 5.2.3 Caso de Teste LIM-03: Valores Extremos de CO₂

#### 5.2.3.1 Descrição

Validar comportamento do sistema com valores extremos de CO₂.

**Cobertura de Requisitos:**  
BIoT: RF04, RNF5  
Manager: RF13

#### 5.2.3.2 Pré-condições

- Sistema operacional
- Limite de CO₂ configurado = 1000ppm

#### 5.2.3.3 Conjunto de valores

| | Cenário 1 (0 ppm) | Cenário 2 (500 ppm) | Cenário 3 (999 ppm) | Cenário 4 (1000 ppm) | Cenário 5 (5000 ppm) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Valor CO₂** | 0 | 500 | 999 | 1000 | 5000 |
| **Resultado Esperado** | Aceito, sem alerta | Aceito, sem alerta | Aceito, sem alerta | Aceito, alerta ⚠ (limite exato) | Aceito, alerta ⚠ |
| **Dashboard** | Exibir 0ppm | Exibir 500ppm | Exibir 999ppm | Exibir 1000ppm + alerta | Exibir 5000ppm + alerta |
| **Resultado Obtido** | | | | | |
| **Sucesso/Falha** | | | | | |

---

### 5.2.4 Caso de Teste LIM-04: Valores Extremos de Temperatura

#### 5.2.4.1 Descrição

Validar comportamento do sistema com valores extremos de temperatura.

**Cobertura de Requisitos:**  
BIoT: RF03, RNF5  
Manager: RF13

#### 5.2.4.2 Pré-condições

- Sistema operacional
- Limites de temperatura configurados = 18°C (mín) / 26°C (máx)

#### 5.2.4.3 Conjunto de valores

| | Cenário 1 (Muito baixo) | Cenário 2 (Mín limite) | Cenário 3 (Normal) | Cenário 4 (Máx limite) | Cenário 5 (Muito alto) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Temperatura (°C)** | 10 | 18 | 22 | 26 | 35 |
| **Resultado Esperado** | Aceito, alerta ⚠ (< 18°C) | Aceito, sem alerta (limite exato) | Aceito, sem alerta | Aceito, sem alerta (limite exato) | Aceito, alerta ⚠ (> 26°C) |
| **Dashboard** | Exibir 10°C + alerta | Exibir 18°C | Exibir 22°C | Exibir 26°C | Exibir 35°C + alerta |
| **LED BIoT** | Aceso (alerta) | Apagado | Apagado | Apagado | Aceso (alerta) |
| **Resultado Obtido** | | | | | |
| **Sucesso/Falha** | | | | | |

**Status Final:** (Sucesso/Falha)

---

### 5.2.5 Caso de Teste LIM-05: Valores Extremos de Umidade

#### 5.2.5.1 Descrição

Validar comportamento do sistema com valores extremos de umidade.

**Cobertura de Requisitos:**  
BIoT: RF06, RNF5  
Manager: RF13

#### 5.2.5.2 Pré-condições

- Sistema operacional
- Limites de umidade configurados = 30% (mín) / 70% (máx)

#### 5.2.5.3 Conjunto de valores

| | Cenário 1 (Muito baixo) | Cenário 2 (Mín limite) | Cenário 3 (Normal) | Cenário 4 (Máx limite) | Cenário 5 (Muito alto) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Umidade (%)** | 15 | 30 | 50 | 70 | 90 |
| **Resultado Esperado** | Aceito, alerta ⚠ (< 30%) | Aceito, sem alerta (limite exato) | Aceito, sem alerta | Aceito, sem alerta (limite exato) | Aceito, alerta ⚠ (> 70%) |
| **Dashboard** | Exibir 15% + alerta | Exibir 30% | Exibir 50% | Exibir 70% | Exibir 90% + alerta |
| **LED BIoT** | Aceso (alerta) | Apagado | Apagado | Apagado | Aceso (alerta) |
| **Resultado Obtido** | | | | | |
| **Sucesso/Falha** | | | | | |

**Status Final:** (Sucesso/Falha)

---

### 5.2.6 Caso de Teste LIM-06: Alerta de Falta de Dados (5 minutos)

#### 5.2.6.1 Descrição

Validar que o Dashboard exibe alerta se não receber dados em 5 minutos.

**Cobertura de Requisitos:**  
Dashboard: RNF04

#### 5.2.6.2 Pré-condições

- Dashboard operacional
- Manager operacional (mas sem receber dados do BIoT)

#### 5.2.6.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | BIoT publica última mensagem | Dashboard atualizado | | | | |
| 2 | Interromper publicação do BIoT (simular falha) | BIoT para de enviar dados | | | | |
| 3 | Aguardar 4 minutos | Dashboard ainda sem alerta | | | | |
| 4 | Aguardar mais 1 minuto (total 5min) | Dashboard exibe alerta "OS DADOS NÃO ESTÃO SENDO ATUALIZADOS" (RNF04) | | | | |
| 5 | BIoT volta a publicar | Alerta desaparece após 15s | | | | |

**Status Final:** (Sucesso/Falha)

---

## 5.3 Testes de Segurança e Controle de Acesso

### 5.3.1 Caso de Teste SEG-01: Login com Credenciais

#### 5.3.1.1 Descrição

Validar autenticação no Manager com credenciais válidas e inválidas.

**Cobertura de Requisitos:**  
Manager: RF17

#### 5.3.1.2 Pré-condições

- Manager acessível via browser
- Usuários cadastrados no sistema

#### 5.3.1.3 Conjunto de valores

| | Cenário 1 (Válido) | Cenário 2 (Senha errada) | Cenário 3 (Email não existe) | Cenário 4 (Campos vazios) |
| :--- | :--- | :--- | :--- | :--- |
| **Email** | admin@safe.br | admin@safe.br | inexistente@safe.br | (vazio) |
| **Senha** | Admin@2025! | SenhaErrada123 | Qualquer@123 | (vazio) |
| **Resultado Esperado** | Login bem-sucedido, redirecionado para dashboard interno | Erro: "Credenciais inválidas" | Erro: "Credenciais inválidas" | Erro: "Preencha todos os campos" |
| **Resultado Obtido** | | | | |
| **Sucesso/Falha** | | | | |

---

### 5.3.2 Caso de Teste SEG-02: Troca de Senha no Primeiro Acesso

#### 5.3.2.1 Descrição

Validar que o sistema obriga mudança de senha no primeiro login.

**Cobertura de Requisitos:**  
Manager: RF18

#### 5.3.2.2 Pré-condições

- Usuário novo criado: novousuario@safe.br / SenhaTemp123!
- Usuário nunca fez login antes

#### 5.3.2.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Fazer login com novousuario@safe.br | Login aceito | | | | |
| 2 | Após login | Sistema redireciona para tela de "Alterar Senha" obrigatória | | | | |
| 3 | Tentar acessar outras páginas sem alterar senha | Acesso bloqueado, redirecionado para alteração | | | | |
| 4 | Inserir nova senha: "senha" (fraca) | Erro: "Senha deve ter >6 caracteres, números, maiúsculas, minúsculas e caracteres especiais" (RF18) | | | | |
| 5 | Inserir nova senha: "NovaSenha@2025!" | Senha aceita, acesso liberado | | | | |

**Status Final:** (Sucesso/Falha)

---

### 5.3.3 Caso de Teste SEG-04: Controle de Acesso por Papel (RBAC)

#### 5.3.3.1 Descrição

Validar que usuários só acessam recursos permitidos para seu papel.

**Cobertura de Requisitos:**  
Manager: RF03, RNF14

#### 5.3.3.2 Pré-condições

- Usuários de diferentes papéis autenticados

#### 5.3.3.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Login como Administrador | Acesso total a todas as funcionalidades | | | | |
| 2 | Login como Gestor (gestor@ct.ufrj.br) | Pode gerenciar instalações do CT, mas não de outros centros (RF03) | | | | |
| 3 | Gestor tenta gerenciar instalação do CT-2 (fora da hierarquia) | Acesso negado (RNF14, RF03) | | | | |
| 4 | Login como Staff | Pode solicitar uso/manutenção/limpeza, mas não gerenciar instalações | | | | |
| 5 | Staff tenta acessar painel de configuração de limites | Acesso negado | | | | |
| 6 | Login como Equipe Manutenção | Pode aceitar/concluir manutenções, mas não gerenciar usuários | | | | |

**Status Final:** (Sucesso/Falha)

---

## 5.4 Testes de Confiabilidade e Tolerância a Falhas

### 5.4.1 Caso de Teste TOL-01: Queda e Retorno de Internet (BIoT)

#### 5.4.1.1 Descrição

Validar reconexão automática do BIoT após queda de rede.

**Cobertura de Requisitos:**  
BIoT: RNF7, RNF9

#### 5.4.1.2 Pré-condições

- BIoT operacional e conectado
- Controle sobre Wi-Fi (para desabilitar/habilitar)

#### 5.4.1.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Verificar BIoT publicando dados normalmente | Publicações a cada 15s | | | | |
| 2 | Desabilitar Wi-Fi do roteador | BIoT perde conexão | | | | |
| 3 | Aguardar 2 minutos (BIoT armazena dados localmente - RNF9) | BIoT continua coletando, mas não publica | | | | |
| 4 | Reativar Wi-Fi | BIoT reconecta automaticamente em <30s (RNF7) | | | | |
| 5 | Verificar Broker | BIoT publica dados armazenados localmente (backlog) + dados novos | | | | |
| 6 | Verificar Manager/BD | Todos os dados (inclusive do período offline) foram recebidos | | | | |

**Status Final:** (Sucesso/Falha)

---

### 5.4.2 Caso de Teste TOL-06: Dados em Formato Errado

#### 5.4.2.1 Descrição

Validar que o Manager exibe alerta ao receber dados inválidos.

**Cobertura de Requisitos:**  
Manager: RNF06

#### 5.4.2.2 Pré-condições

- Manager operacional
- Acesso ao Broker para publicar dados

#### 5.4.2.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Publicar JSON inválido no tópico SAFE_IAQ:<br>`{temperatura: vinte, co2: abc}` | Manager detecta formato inválido | | | | |
| 2 | Verificar tela do Manager | Exibir alerta: "DADOS DO DISPOSITIVO [ID] EM FORMATO ERRADO" (RNF06) | | | | |
| 3 | Verificar log do Manager | Erro registrado com timestamp e payload recebido | | | | |

**Status Final:** (Sucesso/Falha)

---

## 5.5 Testes de Desempenho e Eficiência

### 5.5.1 Caso de Teste PERF-02: Precisão de Leitura dos Sensores (90%)

#### 5.5.1.1 Descrição

Validar que os sensores do BIoT têm precisão mínima de 90%.

**Cobertura de Requisitos:**  
BIoT: RNF5

#### 5.5.1.2 Pré-condições

- BIoT com sensores calibrados
- Equipamento de referência (termômetro, medidor de CO₂ profissional)

#### 5.5.1.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Posicionar equipamento de referência ao lado do BIoT | Ambos no mesmo ambiente | | | | |
| 2 | Coletar 100 leituras de temperatura | Leituras coletadas | | | | |
| 3 | Comparar cada leitura BIoT com referência | Erro médio ≤10% (Precisão ≥90% - RNF5) | | | | |
| 4 | Repetir para CO₂ (100 leituras) | Erro médio ≤10% | | | | |
| 5 | Repetir para umidade (100 leituras) | Erro médio ≤10% | | | | |
| 6 | Contagem de pessoas: simular 50 entradas/saídas | Contador BIoT correto em ≥45 eventos (90%) | | | | |

**Status Final:** (Sucesso/Falha)

---

## 5.6 Testes de Usabilidade (Interface Responsiva e Compatibilidade)

### 5.6.1 Caso de Teste RESP-01: Layout Responsivo Pequeno (≤640px)

#### 5.6.1.1 Descrição

Validar interface responsiva em dispositivos pequenos.

**Cobertura de Requisitos:**  
Manager: RNF18  
Dashboard: RNF13

#### 5.6.1.2 Pré-condições

- Dashboard e Manager acessíveis
- Browser com DevTools para simular resolução

#### 5.6.1.3 Cenário (Script)

| Passo | Descrição | Resultado Esperado | Resultado Obtido | Sucesso/Falha | Nº Ambiente | Nº Log |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Abrir Dashboard no Chrome | Página carregada | | | | |
| 2 | DevTools: configurar resolução 360x640 (mobile) | Layout ajustado para mobile | | | | |
| 3 | Verificar cards de instalações | Cards empilhados verticalmente (RNF14 Dashboard) | | | | |
| 4 | Verificar navegação | Menu hamburguer ou navegação mobile visível | | | | |
| 5 | Testar scroll e interações | Todos os elementos acessíveis e funcionais | | | | |
| 6 | Repetir para Manager (com login) | Layout responsivo funcionando | | | | |

**Status Final:** (Sucesso/Falha)

---

### 5.6.2 Caso de Teste RESP-04: Compatibilidade de Navegadores

#### 5.6.2.1 Descrição

Validar compatibilidade com navegadores específicos.

**Cobertura de Requisitos:**  
Manager: RNF13  
Dashboard: RNF12

#### 5.6.2.2 Pré-condições

- Navegadores instalados: Chrome 118+, Firefox 115+, Edge 118+, Safari 16+

#### 5.6.2.3 Conjunto de valores

| | Chrome 118 | Firefox 115 | Edge 118 | Safari 16 |
| :--- | :--- | :--- | :--- | :--- |
| **Dashboard carrega** | Sim | Sim | Sim | Sim |
| **Gráficos exibidos** | Sim | Sim | Sim | Sim |
| **Alertas funcionam** | Sim | Sim | Sim | Sim |
| **Manager login** | Sim | Sim | Sim | Sim |
| **Resultado Obtido** | | | | |
| **Sucesso/Falha** | | | | |

---

> **Nota:** Casos de teste adicionais para Fluxos E2E-03, E2E-04 (já especificados acima como CT 4.3 e 4.4) e outros requisitos específicos podem ser adicionados conforme necessário durante a execução.
