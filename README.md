# 🎬 CineMania - Sistema de Microsserviços para Cinema

O **CineMania** é uma plataforma distribuída e escalável para gerenciamento de cinemas, sessões e venda de ingressos em cenários de alta concorrência (como a pré-venda de grandes estreias do cinema, ex: *Homem-Aranha*). 

O projeto foi desenvolvido utilizando o ecossistema **Spring Cloud**, **Docker**, **Redis** e mensageria distribuída, aplicando conceitos rigorosos de segurança (**Tríade CIA**), arquitetura limpa e conformidade com a **LGPD**.

---

## 🏗️ Arquitetura do Sistema

O sistema é dividido em microsserviços totalmente autônomos, cada um possuindo sua própria base de dados (*Database per Service*):

* **`cinema-gateway` (API Gateway):** Porta de entrada única da aplicação. Responsável pelo roteamento dinâmico das requisições e centralização da segurança.
* **`cinema-catalog-service` (Catálogo de Filmes):** Gerencia os filmes em cartaz, salas (ex: IMAX, 3D) e horários de sessões. Utiliza cache estratégico para leitura rápida de programações muito acessadas.
* **`ticket-booking-service` (Reserva e Ingressos):** O coração transacional do sistema. Controla o mapa de poltronas das salas e gerencia o estado de reserva temporária de assentos durante o checkout.
* **`payment-service` (Pagamentos):** Processa as transações financeiras de forma assíncrona e gerencia o estorno ou liberação de poltronas em caso de falha.
* **`notification-service` (Notificações):** Microsserviço focado no envio de e-mails com os ingressos gerados em formato QR Code após a confirmação do pagamento.

---

## 🛠️ Tecnologias e Ferramentas Utilizadas

* **Linguagem Principal:** Java 17 / 21
* **Framework Base:** Spring Boot 3.x
* **Acesso a Dados:** Spring Data JPA / Hibernate
* **Bancos de Dados:** MySQL (Bancos isolados por serviço)
* **Performance e Concorrência:** Redis (Cache de sessões e Distributed Lock para evitar vendas duplicadas da mesma poltrona)
* **Comunicação Síncrona:** Spring Cloud OpenFeign (com Resilience4j para Circuit Breaker)
* **Comunicação Assíncrona:** Apache Kafka ou RabbitMQ (Event-Driven Architecture)
* **Conteinerização:** Docker e Docker Compose
* **Segurança:** Spring Security, OAuth2 e JWT (Criptografia AES para dados sensíveis de usuários em conformidade com a LGPD)

---

## 🛡️ Engenharia de Segurança & LGPD

* **Confidencialidade:** Dados sensíveis de usuários e cartões são criptografados a nível de banco de dados através de *Attribute Converters* do JPA utilizando o algoritmo AES. As credenciais de infraestrutura nunca ficam expostas no código, sendo injetadas via variáveis de ambiente (`${DB_PASSWORD}`).
* **Integridade:** Utilização de mecanismos de concorrência e *Distributed Locking* com Redis para assegurar a consistência do mapa de assentos, impedindo condições de corrida (*race conditions*).
* **Disponibilidade & Resiliência:** Caso o serviço de notificações ou pagamentos fique temporariamente indisponível, o sistema não cai por inteiro. As mensagens ficam retidas nas filas do Message Broker até o reestabelecimento do serviço.
* **LGPD (Privacy by Design):** Implementação de rotinas de exclusão lógica (*soft delete*) e anonimização de dados cadastrais mediante solicitação do usuário, mantendo o histórico de vendas íntegro para fins fiscais sem violar a privacidade do indivíduo.

---

## 🚀 Como Executar o Projeto Localmente

### Pré-requisitos
* JDK 17 ou superior instalado
* Maven instalado
* Docker e Docker Compose rodando na máquina

### Passo 1: Subir a Infraestrutura (Bancos e Cache)
Navegue até a pasta raiz onde está o arquivo `docker-compose.yml` e execute:
```bash
docker compose up -d
