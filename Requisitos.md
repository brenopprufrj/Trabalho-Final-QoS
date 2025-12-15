# Requisitos do Sistema SAFE

## 1. Subsistema Manager

O Manager é o componente sistêmico central, responsável por orquestrar ações, gerenciar instalações e interagir com o broker MQTT.

### Requisitos Funcionais (RF)

| ID | Descrição |
| :--- | :--- |
| **RF01** | O Manager deve permitir que os administradores gerenciem os gestores das instalações. |
| **RF02** | O Manager deve permitir que administradores gerenciem dispositivos BIoT, caracterizados pelas seguintes informações: id, versão do software, porta, data de instalação. |
| **RF03** | O Manager deve permitir que administradores e gestores das instalações gerenciem as instalações, caracterizadas pelas seguintes informações: localização do espaço e de suas sublocalizações (salas, laboratórios, auditório). <br>**Regra de negócio:** Um gestor de instalação só pode gerenciar instalações incluídas na sua hierarquia de localização. |
| **RF04** | O Manager deve permitir que gestores, staff, equipe de manutenção, equipe de limpeza informem que uma instalação cadastrada necessita de manutenção. |
| **RF05** | O Manager deve notificar por e-mail a equipe de manutenção quando alguma manutenção para uma instalação for cadastrada. |
| **RF06** | O Manager deve permitir que a equipe de manutenção informe o aceite de uma solicitação de serviço de manutenção para uma instalação cadastrada. |
| **RF07** | O Manager deve permitir que o gerente da instalação informe que uma solicitação de serviço de manutenção foi concluída. |
| **RF08** | O Manager deve permitir que gestores das instalações, staff, equipe de manutenção, equipe de limpeza informem que uma instalação cadastrada necessita de serviço de limpeza. |
| **RF09** | O Manager deve notificar a equipe de limpeza quando algum serviço de limpeza for solicitado para uma instalação cadastrada. |
| **RF10** | O Manager deve permitir que a equipe de limpeza informe o aceite de uma solicitação de serviço de limpeza para uma instalação cadastrada. |
| **RF11** | O Manager deve permitir que o gerente de instalação informe que uma solicitação de serviço de limpeza foi concluída. |
| **RF12** | O Manager deve permitir que o Staff solicite o uso de uma instalação cadastrada em determinada data e horário. |
| **RF13** | O Manager deve armazenar em banco de dados todos os dados recebidos dos dispositivos BIoT, incluindo número de pessoas na instalação, temperatura, CO₂, umidade, VOCs, data, hora, identificador da instalação e do dispositivo. |
| **RF14** | O Manager deve guardar o histórico de eventos (nome do solicitante, data/hora da solicitação, nome do funcionário que atendeu à solicitação, data/hora do atendimento, data/hora da conclusão e a instalação para a qual o pedido foi realizado). |
| **RF15** | O Manager deve exibir o estado de utilização de cada instalação monitorada, atualizando-o conforme os eventos percebidos pelo sistema. Sendo estes:<br>• **“Não Liberada”** (Motivos: Aguardando limpeza, Aguardando manutenção, Aguardando limpeza e manutenção, Em uso)<br>• **“Liberado”**: disponível para uso. |
| **RF16** | O Manager deve notificar os gestores das instalações em caso de uso não autorizado de instalações cadastradas (por exemplo, alguém acessa uma sala com alto risco de contaminação sem autorização). |
| **RF17** | O Manager deve permitir o login no SAFE com os dados de e-mail e de senha do usuário. |
| **RF18** | O Manager deve permitir que o usuário modifique a senha após o primeiro acesso. Critérios: Maior que 6 caracteres, Possuir números, Possuir letras maiúsculas e minúsculas, Possuir caracteres especiais. |

### Requisitos Não Funcionais (RNF)

| ID | Descrição |
| :--- | :--- |
| **RNF01** | O Manager deve se comunicar com o broker pelo protocolo MQTT. |
| **RNF02** | O Manager deve enviar os limites de biossegurança das instalações cadastradas aos dispositivos BIoT por meio do broker e ao subsistema Dashboard, por meio da API. |
| **RNF03** | O Manager deve obter os dados enviados pelos dispositivos BIoT por meio do broker. |
| **RNF04** | O Manager deve enviar o estado de uma instalação ao subsistema Dashboard a cada mudança de estado, por meio do Broker. |
| **RNF05** | O Manager deve exibir, na tela, o alerta “FALTA DE DADOS DA INSTALAÇÃO `<ID>`” quando detectar a falta de dados de um dispositivo IoT associado. |
| **RNF06** | O Manager deve exibir o alerta “DADOS DO DISPOSITIVO `<ID>` EM FORMATO ERRADO” na tela em caso de recebimento de dados inválidos. |
| **RNF07** | O Manager deve buscar novos dados transmitidos pelos dispositivos BIoT no broker a cada `INTERVALO_ATUALIZAR_DADOS`. |
| **RNF08** | O Manager deve buscar novas notificações no broker a cada `INTERVALO_ATUALIZACAO_NOTIFICACOES`. |
| **RNF09** | O Manager deve garantir disponibilidade mínima de 99,9% ao mês, exceto janelas de manutenção. |
| **RNF10** | O Manager deve permitir atualizações de versão por meio de Docker. |
| **RNF11** | O Manager deve oferecer aos Administradores um meio direto de acesso ao servidor em caso de falha sistêmica. |
| **RNF12** | O Manager deve realizar atualizações mensais dos componentes de software (dependências, bibliotecas). |
| **RNF13** | O Manager deve garantir compatibilidade com, no mínimo, as versões: Chrome 118, Firefox 115, Edge 118 e Safari 16. |
| **RNF14** | O Manager deve realizar o controle de acesso baseado em papéis (administrador e gestor), garantindo acesso apenas a pessoal autorizado. |
| **RNF15** | O Manager deve encerrar automaticamente a sessão após o tempo máximo de inatividade configurado (`TEMPO_MAXIMO_INATIVIDADE`). |
| **RNF16** | O Manager deve utilizar o CryptoComponent, com o protocolo Speck, para o tratamento dos dados recebidos dos dispositivos BIoT. |
| **RNF17** | O Manager deve utilizar algoritmos de criptografia (CryptoComponent/Speck) para o tratamento dos dados enviados aos dispositivos BIoT. |
| **RNF18** | O Manager deve oferecer uma interface responsiva considerando:<br>• **Pequeno:** 640px ou menos (Telefones e TVs)<br>• **Médio:** 641px a 1007px (Phablets, tablets)<br>• **Grande:** 1008px ou mais (Computadores, laptops, Surface Hubs) |
| **RNF19** | O Manager deve consumir o que o BIoT publicou via broker nos tópicos (escritos em JSON) "SAFE_IAQ" e "SAFE_ENTRY_FLOW". |

---

## 2. Subsistema SAFE BIoT

O BIoT é o dispositivo IoT para coleta de dados ambientais.

### Requisitos Funcionais (RF)

| ID | Descrição |
| :--- | :--- |
| **RF01** | O dispositivo BIoT deve, na inicialização, obter do broker (via tópico MQTT definido) os limites de temperatura, CO₂ e ocupação publicados pelo SAFE Manager. |
| **RF02** | O dispositivo BIoT deve contabilizar cada movimento de entrada e saída de pessoas na instalação por meio dos sensores HC-SR04. |
| **RF03** | O dispositivo BIoT deve coletar a temperatura (em °C) utilizando o sensor DHT11 ou AHT21. |
| **RF04** | O dispositivo BIoT deve coletar a medida da concentração de eCO₂ (ppm) usando o sensor CJMCU-811 ou o ENS160. |
| **RF05** | O dispositivo BIoT deve apresentar o estado da instalação em relação aos parâmetros monitorados e emitir um sinal de risco em um painel sinalizador localizado na instalação. |
| **RF06** | O dispositivo BIoT deve coletar a medida de umidade da instalação utilizando o sensor DHT11 ou AHT21. |
| **RF07** | O dispositivo BIoT deve coletar a medida de VOCs (Compostos orgânicos voláteis) (ppm) da instalação. |

### Requisitos Não Funcionais (RNF)

| ID | Descrição |
| :--- | :--- |
| **RNF1** | O dispositivo BIoT deve se comunicar com o broker por meio do protocolo MQTT. |
| **RNF2** | O dispositivo BIoT deve enviar as informações dos sensores para o broker utilizando o formato JSON através dos respectivos tópicos:<br><br>**SAFE_IAQ:**<br>`{ "idBIoT": <MAC>, "timestamp": <data>, "temperatura": <valor>, "umidade": <valor>, "co2": <valor>, "vocs":<valor>, "erro": <valor> }`<br>*(Onde "erro" é uma lista de flags)*<br><br>**SAFE_ENTRY_FLOW:**<br>`{ "idBIoT": <MAC>, "timestamp": <data>, "entry_flow": <valor>, "erro": <valor> }` |
| **RNF3** | O dispositivo BIoT deve obter os valores atualizados dos parâmetros de biossegurança a cada `INTERVALO_ATUALIZACAO_LIMITES`. |
| **RNF4** | O dispositivo BIoT deve consultar as informações dos parâmetros no broker com o tópico `MQTT_TOPICO` no formato JSON:<br>`{ "idBIoT": <MAC>, "temperaturaMin": <v>, "temperaturaMax": <v>, "umidadeMin": <v>, "umidadeMax": <v>, "numeroPessoasMin": <v>, "numeroPessoasMax": <v> }` |
| **RNF5** | Os dados (Número de pessoas, Temp, CO₂, umidade) devem ser informados com precisão de pelo menos 90%. |
| **RNF6** | O dispositivo BIoT deve retomar automaticamente suas atividades após restabelecimento de energia. |
| **RNF7** | O dispositivo BIoT deve se reconectar à internet automaticamente quando ela voltar a estar disponível. |
| **RNF8** | O dispositivo BIoT deve sincronizar seu relógio com o horário da Internet sempre que publicar um tópico no broker. |
| **RNF9** | O dispositivo BIoT deve armazenar os dados mais recentes em memória local em caso de falha de conexão e enviá-los assim que restabelecida. |
| **RNF10** | O dispositivo BIoT deve publicar no Broker os dados de qualidade de ar a cada 15 segundos (`INTERVALO_ATUALIZACAO_DADOS_QUALIDADE_AR`). |
| **RNF11** | O dispositivo BIoT deve publicar no Broker os dados de contagem de pessoas quando detectar um evento de entrada/saída. |
| **RNF12** | O dispositivo BIoT deve atualizar os limites dos parâmetros de biossegurança assim que se ligar e a cada `INTERVALO_ATUALIZACAO_LIMITES`. |
| **RNF13** | O dispositivo BIoT deve coletar e enviar as informações de maneira ininterrupta na semana, seguindo os intervalos definidos. |
| **RNF14** | O dispositivo BIoT deve ser compatível com a plataforma NodeMCU. |
| **RNF15** | O dispositivo BIoT deve estabelecer comunicação autenticada com o Broker por meio de chaves. |
| **RNF16** | O dispositivo BIoT não deve permitir acesso ou login via Wi-Fi. |
| **RNF17** | O dispositivo BIoT deve criptografar os dados transmitidos com o CryptoComponent (protocolo Speck). |
| **RNF18** | Os dispositivos devem ser encapsulados em uma estrutura protetora. |
| **RNF19** | O dispositivo BIoT deve possuir um broker privado. |
| **RNF20** | O dispositivo BIoT deve se comunicar a partir de uma rede local e sem fio. |
| **RNF21** | O dispositivo BIoT deve ser alimentado por meio de uma fonte 5V, utilizando a porta micro USB. |
| **RNF22** | O dispositivo BIoT deve reportar falhas através de etiquetas de erro (bit a bit):<br>1. Falha na obtenção de data e hora (1)<br>2. Falha na coleta de temperatura (2)<br>3. Falha na coleta de umidade (4)<br>4. Falha na coleta de CO₂ (8) |

---

## 3. Subsistema SAFE Dashboard

Painel de Monitoramento e Interface de Gerenciamento.

**Definições de Constantes de Erro:**
*   `#define ERROR_TIME (1 << 0)`
*   `#define ERROR_TEMP (1 << 1)`
*   `#define ERROR_HUMID (1 << 2)`
*   `#define ERROR_CO2 (1 << 3)`
*   `#define ERROR_TVOC (1 << 4)`

### Requisitos Funcionais (RF)

| ID | Descrição |
| :--- | :--- |
| **RF01** | O Dashboard deve exibir a temperatura (°C) de cada instalação cadastrada com dispositivo vinculado, sem necessidade de autenticação. |
| **RF02** | O Dashboard deve mostrar a concentração de CO₂ (ppm) de cada instalação cadastrada com dispositivo vinculado, sem necessidade de autenticação. |
| **RF03** | O Dashboard deve mostrar o número de pessoas presentes em cada instalação cadastrada com dispositivo vinculado, sem necessidade de autenticação. |
| **RF04** | O Dashboard deve mostrar a classificação de risco de cada instalação cadastrada, baseada nos limites do Guia de Biossegurança, sem necessidade de autenticação. |
| **RF05** | O Dashboard deve mostrar o estado de utilização ("bloqueado", "bloqueado (em limpeza)", "liberado") de cada instalação, sem necessidade de autenticação. |
| **RF06** | O Dashboard deve apresentar a localização da instalação cadastrada, juntamente com as informações da unidade. |
| **RF07** | O Dashboard deve apresentar a evolução dos marcadores (temp, CO₂, umidade, VOCs, pessoas) na última hora, em gráficos lineares temporais (intervalo de 2 min). |
| **RF08** | O Dashboard deve exibir os valores máximos de cada medida na última hora. |
| **RF09** | O Dashboard deve exibir o percentual de umidade de cada instalação cadastrada com dispositivo vinculado. |
| **RF10** | O Dashboard deve mostrar a unidade de VOCs de cada instalação cadastrada com dispositivo vinculado. |
| **RF11** | O Dashboard deve alertar o usuário quando o limite máximo de qualquer medição for atingido. |
| **RF12** | O Dashboard deve exibir uma legenda no rodapé indicando: <br>🔴 → Bloqueado<br>🟢 → Liberado<br>⚠ → Limite máximo atingido |

### Requisitos Não Funcionais (RNF)

| ID | Descrição |
| :--- | :--- |
| **RNF01** | O Dashboard deve se comunicar com o subsistema SAFE Manager (servidor) por meio das rotas de API. |
| **RNF02** | O Dashboard deve obter os limites dos parâmetros de segurança das instalações consultando a API do SAFE Manager com a estrutura JSON:<br>`{ "id": <v>, "name": <v>, "installation_requests": [<v>], "installation_threshold": [ { "temperature": <v>, "humidity": <v>, "ppmco2": <v>, "number_people": <v> } ] }` |
| **RNF03** | O Dashboard deve obter os dados enviados pelos dispositivos BIoT do subsistema SAFE Manager. |
| **RNF04** | O Dashboard deve manter um alerta “OS DADOS NÃO ESTÃO SENDO ATUALIZADOS”, caso não consiga atualizar as informações no `TEMPO_MAXIMO_ANTES_ALERTA_DASHBOARD`. |
| **RNF05** | O Dashboard deve obter novos dados enviados pelos dispositivos a cada `INTERVALO_ATUALIZAR_DADOS`. |
| **RNF06** | O Dashboard deve atualizar os limites dos parâmetros de biossegurança a cada `INTERVALO_ATUALIZACAO_LIMITES`. |
| **RNF07** | O Dashboard deve atualizar os estados de utilização das instalações a cada `INTERVALO_ATUALIZACAO_ESTADO`. |
| **RNF08** | O Dashboard deve informar caso haja indisponibilidade no subsistema SAFE Manager. |
| **RNF09** | O Dashboard deve permitir atualizações por meio de Docker. |
| **RNF10** | O Dashboard deve oferecer aos Administradores um meio direto de acesso ao servidor do Manager em caso de falha sistêmica. |
| **RNF11** | O Dashboard deve realizar atualizações mensais dos componentes de software. |
| **RNF12** | O Dashboard deve garantir compatibilidade com, no mínimo: Chrome 118, Firefox 115, Edge 118 e Safari 16. |
| **RNF13** | O Dashboard deve oferecer uma interface responsiva (Pequeno, Médio, Grande) seguindo os mesmos critérios definidos no Manager. |
| **RNF14** | O Dashboard deve apresentar as instalações cadastradas em espaços individuais na interface (cards). |
| **RNF15** | O Dashboard deve se comunicar com o subsistema SAFE Manager por meio de internet. |