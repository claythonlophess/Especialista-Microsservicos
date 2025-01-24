# Arquitetura de Microsserviços para Carteira Móvel 

## **Descrição**

Esta arquitetura implementa uma **carteira móvel** que permite aos usuários realizar transações como enviar, receber, pagar contas e retirar dinheiro em ATMs ou agentes. O sistema utiliza uma arquitetura de microsserviços para garantir escalabilidade, independência entre funcionalidades e alta disponibilidade.

---

## **Componentes Principais**

### 1. **Serviço de Autenticação e Autorização**

- **Responsabilidade:** Gerenciar login, registro de usuários, autenticação de dois fatores (2FA) e permissões.
- **Tecnologias:** JWT (JSON Web Tokens) para autenticação e OAuth 2.0.
- **Comunicação:** Usado por todos os outros serviços para validação de tokens.

---

### 2. **Serviço de Carteira (Wallet Service)**

- **Responsabilidade:** Gerenciar saldo da carteira, recarga de fundos e transferências entre carteiras.
- **Banco de Dados:** Base de dados transacional (e.g., PostgreSQL ou MongoDB).
- **Endpoints:**
  - `GET /wallet/{userId}/balance` (Consulta saldo)
  - `POST /wallet/{userId}/transfer` (Transferir fundos)

---

### 3. **Serviço de Pagamentos**

- **Responsabilidade:** Processar pagamentos a fornecedores, como contas de água, luz, internet, ou recarga de celular.
- **Integrações:** Conexão com APIs de provedores de pagamento.
- **Endpoints:**
  - `POST /payments/bill` (Pagamento de conta)
  - `POST /payments/topup` (Recarga de saldo para celular)

---

### 4. **Serviço de Retirada (ATM Service)**

- **Responsabilidade:**
  - Gerenciar solicitações de retirada de dinheiro em ATMs ou agentes.
  - Enviar tokens de autorização ou QR codes para retirada.
- **Fluxo:**
  1. O usuário solicita retirada.
  2. O serviço gera um token único ou QR code.
  3. O token é validado no ATM ou no agente para liberar o valor.
- **Endpoints:**
  - `POST /atm/withdrawal-request` (Solicitar retirada)
  - `POST /atm/validate-token` (Validar token no ATM)

---

### 5. **Serviço de Notificações**

- **Responsabilidade:** Enviar notificações por SMS, e-mail ou push (ex.: "Seu pagamento foi concluído com sucesso").
- **Tecnologias:** RabbitMQ ou Kafka para processamento assíncrono de mensagens.

---

### 6. **Serviço de Relatórios**

- **Responsabilidade:** Gerar relatórios de transações do usuário, estatísticas de uso e tendências de comportamento.
- **Endpoint:**
  - `GET /reports/transactions/{userId}` (Histórico de transações)

---

## **Fluxo de Exemplo: Retirada no ATM**

1. O usuário acessa o app e solicita uma retirada de \$100.
2. O **Serviço de Carteira** verifica o saldo disponível.
3. O **Serviço de Retirada** gera um token único ou QR code.
4. O usuário vai ao ATM e insere o token ou escaneia o QR code.
5. O ATM se comunica com o **Serviço de Retirada** para validar o token.
6. O **Serviço de Carteira** debita o valor da conta do usuário.
7. O ATM dispensa o dinheiro.

---

## **Tecnologias Recomendas**

- **Backend:** Node.js, Spring Boot, Laravel ou Django para os serviços.
- **Banco de Dados:**
  - Relacional (PostgreSQL/MySQL) para informações transacionais.
  - Não-relacional (MongoDB/Redis) para cache ou dados de leitura.
- **Mensageria:** RabbitMQ, Kafka ou Redis para comunicação entre microsserviços.
- **APIs:** REST ou gRPC para comunicação entre microsserviços.
- **Segurança:** Autenticação com OAuth 2.0 e encriptação de dados sensíveis (ex.: PCI DSS).

---

## **Benefícios da Arquitetura de Microsserviços**

1. **Escalabilidade:** Cada serviço pode ser escalado de forma independente.
2. **Manutenção:** Alterar um serviço não impacta os outros, reduzindo o risco de falhas gerais.
3. **Desempenho:** Uso de mensageria e processamento assíncrono para maior eficiência.
4. **Flexibilidade:** Permite a utilização de tecnologias diferentes para cada serviço.
5. **Segurança:** Cada serviço tem acesso restrito apenas às partes necessárias do sistema.

