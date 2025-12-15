### Extração Complementar (Informações de Domínio)

Adicione estas seções ao seu planejamento para dar insumo aos casos de teste:

#### A. Tabela de Variáveis e Constantes (Para verificação de tempos e limites)
*Essenciais para testes de desempenho e comportamento temporal.*

| Nome da Variável | Valor Definido | Descrição (Para o Testador) |
| :--- | :--- | :--- |
| **INTERVALO_ATUALIZACAO_DADOS_BROKER** | 15 segundos | Tempo entre acessos ao broker para coletar dados IoT. |
| **INTERVALO_ATUALIZACAO_DADOS_CO2** | 5 segundos | Tempo de coleta do sensor CJMCU-811. |
| **INTERVALO_ATUALIZACAO_DADOS_TEMPERATURA** | 7,5 segundos | Tempo de coleta do sensor DHT11. |
| **INTERVALO_ATUALIZACAO_ESTADO** | 10 minutos | Frequência de verificação do estado da instalação (Livre/Ocupada). |
| **INTERVALO_ATUALIZACAO_LIMITES** | 30 minutos | Frequência de envio de novos limites de biossegurança para o BIoT. |
| **INTERVALO_ATUALIZACAO_NOTIFICACOES** | 10 segundos | Tempo para buscar novas notificações e verificar Wi-Fi. |
| **INTERVALO_ATUALIZACAO_NUMERO_PESSOAS** | 1 segundo | Tempo de coleta dos sensores HC-SR04 (entrada/saída). |
| **INTERVALO_ATUALIZAR_DADOS** (Manager) | 15 segundos | Tempo para o Manager coletar dados disponíveis. |
| **TEMPO_MAXIMO_ANTES_ALERTA_DASHBOARD** | 5 minutos | Se o Dashboard não receber dados nesse tempo, exibe alerta. |
| **TEMPO_MAXIMO_INATIVIDADE** | 1 hora | Tempo para *logout* automático do usuário. |
| **PERIODO_OBSERVACAO_HISTORICO_DADOS** | 4 horas | Janela de tempo padrão exibida nos gráficos. |

#### B. Definição de Papéis e Permissões (Para testes de segurança/acesso)
*Essenciais para criar casos de teste de "Tentativa de acesso não autorizado".*

*   **Administrador do Sistema:** Superusuário. Gerencia instalações, gestores e dispositivos. Vê problemas técnicos.
*   **Gestor das Instalações:** Gerencia usuários (Equipes e Staff), define limites de biossegurança, aprova solicitações de uso.
*   **Equipe de Manutenção:** Pode receber e atender solicitações de manutenção. (Não pode solicitar uso de sala, por exemplo).
*   **Equipe de Limpeza:** Pode receber e atender solicitações de limpeza.
*   **Staff (Professores/Funcionários):** Pode solicitar uso de instalação, limpeza e manutenção.
*   **Usuário não-cadastrado (Alunos/Visitantes):** Acesso **apenas** visual ao Dashboard (leitura).

#### C. Estrutura de Hardware (Para simulação/Mocks do BIoT)
*Essencial se o time de teste não tiver o hardware real e precisar simular via software (scripts Python enviando MQTT).*

*   **Processador:** ESP32 (Dual-core).
*   **Sensor CO2/VOCs:** CJMCU-811 ou ENS160.
*   **Sensor Temp/Umidade:** DHT11 ou AHT21.
*   **Contador de Pessoas:** HC-SR04 (Ultrassônico).
*   **Protocolo:** MQTT sobre Wi-Fi.
*   **Tópico Default:** "SAFE" (se não configurado outro).


### D. Glossário Técnico e Definições de Estados
*(Adicione esta seção ao final do bloco [DOMÍNIO] do prompt)*

| Termo | Descrição / Definição |
| :--- | :--- |
| **AHT21** | Sensor de Temperatura e Umidade. |
| **AP** | Quantidade de pessoas numa instalação específica. |
| **Arquitetura dual-core** | Arquitetura com dois cores de processamento oferecida pelo ESP32. |
| **BioT / BIoT** | Dispositivo IoT composto por sensor de temperatura, movimento, CO2. Propriedades: Id, versão software, porta, data instalação. Integra sensores para monitorar qualidade do ar e ocupação. |
| **Bloco** | Espaço físico pertencente a uma ou mais unidades que possui um conjunto de instalações. Pode ter um ou mais andares. |
| **Broker** | Agente intermediário (RabbitMQ) que utiliza protocolo MQTT. Responsável pela comunicação entre dados do dispositivo IoT e o sistema. |
| **C++** | Linguagem de programação utilizada no firmware. |
| **CJMCU-811** | Sensor de CO2 e VOCs. |
| **CO2** | Dióxido de carbono. Unidade de medida: ppm. |
| **CT / CT-2** | Centro de Tecnologia / Centro de Gestão Tecnológica (exemplos de Centros). |
| **DHT11** | Sensor de Temperatura e Umidade. |
| **Docker** | Tecnologia para encapsulamento de aplicações (usada para deploy/atualização). |
| **ENS160** | Sensor de CO2 (alternativo ao CJMCU-811). |
| **ESP32** | Processador do BIoT (Microcontrolador). |
| **FreeRTOS** | Sistema operacional multi-core utilizado no BIoT. |
| **HC-SR04** | Sensor de presença ultrassônico (para contagem de pessoas). |
| **Instalação** | Qualquer ambiente utilizado na UFRJ (sala, laboratório, etc.). Deve possuir: localização, unidade, nível de risco, estado de utilização, dispositivo BIoT associado e tipo. |
| **Localização** | Estrutura hierárquica: **Centro -> Bloco -> Andar**. Propósito: identificar onde a instalação se encontra. |
| **MQTT** | *Message Queuing Telemetry Transport*. Protocolo leve de mensagens para sensores, otimizado para TCP/IP. |
| **MQTT_TOPICO** | Tópico principal para organizar dados. Valor default: “SAFE”. |
| **PPM** | "Partes por milhão". Unidade de medida para CO2. |
| **PPB** | "Partes por bilhão". Unidade de medida para VOCs (gases). |
| **Protocolo Speck** | Algoritmo de criptografia leve utilizado pelo CryptoComponent. |
| **Tipos de Instalação** | Sala de aula, laboratório, sala de reunião, auditório, sala administrativa, refeitório, sala de convivência, biblioteca. |
| **VOCs** | Compostos orgânicos voláteis. Substâncias com carbono, ponto de ebulição 50-260°C. Tóxicos. |

#### Definições de Máquina de Estados (Crítico para Teste)

**1. Estados de solicitações de limpeza ou manutenção:**
*   “Em andamento”
*   “Aguardando atendimento”
*   “Atendida”

**2. Estados de utilização das instalações (SAFE Dashboard/Manager):**
*   **“Não Liberado”** - Pode ocorrer pelos motivos:
    *   “Aguardando limpeza”
    *   “Aguardando manutenção”
    *   “Aguardando limpeza e manutenção”
    *   “Em uso”
*   **“Liberado”** - Instalação disponível para uso.
*   **“Bloqueado”** - Termo usado visualmente no Dashboard (vermelho).

---

### Dica para o Plano de Testes (Cobertura e Esforço Mínimo)

Dado que você tem 1 semana e precisa de "esforço mínimo", sugiro a seguinte estratégia ao usar essas informações:

1.  **Agrupe os Testes por "Fluxo de Dados" e não apenas por ID de requisito.**
    *   *Exemplo:* Em vez de testar RF01, RF02 e RNF01 separadamente, crie um **Caso de Teste de Cenário**: "Ciclo de vida de dados do Sensor".
    *   *Roteiro:* Simular envio de JSON pelo BIoT (RNF2) -> Verificar recebimento no Manager (RF13) -> Verificar exibição no Dashboard (RF01, RF02).
    *   Isso cobre uns 5 requisitos de uma vez.

2.  **Teste de Limites (Boundary Testing):**
    *   Use a tabela de constantes acima. O teste deve verificar o comportamento em: `TEMPO_MAXIMO_INATIVIDADE - 1 minuto` (ainda logado) e `TEMPO_MAXIMO_INATIVIDADE + 1 minuto` (deslogado).

3.  **Teste de API como base:**
    *   Como o SAFE é IoT, muito do teste funcional é verificar se o JSON enviado pelo MQTT chega corretamente na API/Banco. Priorize testes automatizados de API/Integração antes de testes manuais de tela.

Com os Requisitos extraídos anteriormente + Essas tabelas de domínio, você tem tudo para preencher os templates.