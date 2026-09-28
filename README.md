# Event-Driven Production Integration

Expansion of [legacy-integration-bridge](https://github.com/linho22w/legacy-integration-bridge), adding financial reporting, asynchronous decoupling, and real-time streaming on top of the same production data pipeline. Built for the Systems Integration course of my MSc in Computer Engineering at UTAD.

## 🎯 Context

The first part of this project bridged a legacy production-line system to a REST API. This second part expands that integration in three directions: exposing financial data through SOAP web services, decoupling the production source from the API with RabbitMQ, and adding a real-time streaming dashboard for management.

## 🧱 Architecture

```
Producer (WinForms, replaces the legacy desktop app)
      │
      ├──► RabbitMQ Topic Exchange ──► Consumer ──► ProducaoAPI (REST) ──► SQL Server
      │         (routing key: ok / falha)              │
      │                                          shows only failed pieces
      │
      └──► RabbitMQ Stream ──► Gestao_app (real-time dashboard: total, OK, failures, avg time)

API_SOAP (financial Web Service) ◄── Client_Soap (WinForms test client)
      │
      └──► Contabilidade DB (via stored procedures)
```

- **Producer**, replaces the legacy desktop app entirely. Generates production data and publishes it both to a RabbitMQ topic exchange (routing key based on pass/fail) and to a RabbitMQ Stream, in parallel.
- **Consumer**, subscribes to the topic exchange, forwards every record to the REST API, and prints failed pieces to the console as they arrive.
- **Gestao_app**, a management dashboard that reads directly from the RabbitMQ Stream and shows live totals: pieces produced, OK count, failures, and average production time.
- **API_SOAP**, a SOAP web service exposing financial analysis over the `Contabilidade` database (piece with the highest loss, total cost/profit/loss over a period, detailed financials per piece).
- **Client_Soap**, a WinForms client used to exercise and demo the SOAP service.
- **ProducaoAPI**, the REST API from Part 1, unchanged in role: the single point of truth for production data.

## 🐇 Why RabbitMQ Streams alongside a topic exchange

A topic exchange is great for routing (send failures one way, everything another), but messages are consumed once and gone. A management dashboard needs to replay history and support multiple independent readers without competing for the same messages. RabbitMQ Streams solves that: the same production event is published once and can be consumed both as a one-shot integration message (topic exchange, by the Consumer) and as a persisted, replayable log (stream, by the dashboard), each for a different purpose.

## 🗄️ Database

- `database/tables/`, schema for `Produto`, `Testes` and `Custos_Peca`.
- `database/stored-procedures/`, all data access from the REST API and the SOAP service goes through these.
- `database/triggers/`, automatic cost/profit/loss calculation on every new production record.

## 🛠️ Tech Stack

<p align="center">
  <img src="https://skillicons.dev/icons?i=cs,dotnet" />
</p>
<p align="center">
  <img src="https://img.shields.io/badge/SOAP-005571?style=for-the-badge" />
  <img src="https://img.shields.io/badge/RabbitMQ-FF6600?style=for-the-badge&logo=rabbitmq&logoColor=white" />
  <img src="https://img.shields.io/badge/RabbitMQ%20Streams-FF6600?style=for-the-badge" />
  <img src="https://img.shields.io/badge/REST%20API-2496ED?style=for-the-badge" />
  <img src="https://img.shields.io/badge/SQL%20SERVER-CC2927?style=for-the-badge" />
  <img src="https://img.shields.io/badge/WinForms-37474F?style=for-the-badge" />
</p>

## 📂 Repository structure

```
src/
  IS_TP2.sln
  ProducaoAPI/       REST API (from Part 1)
  API_SOAP/          SOAP financial web service
  Client_Soap/       WinForms client for the SOAP service
  Producer/          publishes to RabbitMQ (exchange + stream)
  Consumer/          consumes the exchange, forwards to the REST API
  Gestao_app/        real-time management dashboard (consumes the stream)
database/
  tables/  stored-procedures/  triggers/
```

## ▶️ Running it

This is a proof-of-concept built for a university assignment, so running it needs SQL Server, RabbitMQ (with the Stream plugin enabled) and Visual Studio locally.

1. Create the `Producao` and `Contabilidade` databases and run the scripts in `database/` (tables, then stored procedures, then triggers).
2. Update the connection strings in `src/ProducaoAPI/Controllers/*.cs` and `src/API_SOAP/WebService1.asmx.cs` to point at your local SQL Server instance.
3. Open `src/IS_TP2.sln` in Visual Studio.
4. Run `ProducaoAPI`, then `Producer`, `Consumer`, and `Gestao_app` to see the event-driven pipeline end to end.
5. Run `API_SOAP` and `Client_Soap` to see the financial SOAP service in action.

## 📄 Full Report

[Read the full project report (PDF)](IS_TP2_Relatório.pdf)

## 👤 About

Part of my portfolio. See my [GitHub profile](https://github.com/linho22w) for more projects in AI/ML and backend development.
