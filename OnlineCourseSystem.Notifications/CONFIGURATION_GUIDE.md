# Notifications Microservice — Production Configuration Guide

- The Notifications service runs background workers (Email, Push, Scheduled) inside the same API host.

The Notifications service requires the following external services to run in production:

1- SQL Server database

2- RabbitMQ server

3- SMTP provider for email delivery

4- Firebase project (for push notifications)

---

## Table of Contents

1. [Configuration Categories](#1-configuration-categories)
2. [Required Environment Variables](#2-required-environment-variables)
3. [Example appsettings.Production.json](#3-example-appsettingsproductionjson)
4. [Firebase Configuration](#4-firebase-configuration)
5. [API Base URLs & Ports](#5-api-base-urls--ports)
---

## 1. Configuration Categories

### A) Backend

| Configuration Key | Description |
|-------------------|-------------|
| `ConnectionStrings:DefaultConnection` | SQL Server connection string (credentials) |
| `EmailSettings:SmtpUsername` | SMTP account username |
| `EmailSettings:SmtpPassword` | SMTP account password (app password for Gmail) |
| `EmailSettings:FromEmail` | Sender email address |
| `Firebase:CredentialsPath` | Path to Firebase Admin JSON service account file |
| `RabbitMq:Username` | RabbitMQ username |
| `RabbitMq:Password` | RabbitMQ password |

### B) Frontend

See [Section 4](#4-firebase-configuration-frontend) for the exact structure.

---

## 2. Required Environment Variables

For production, prefer environment variables over `appsettings.Production.json` for secrets.

| Environment Variable | Maps To | Required | Example |
|----------------------|---------|----------|---------|
| `ConnectionStrings__DefaultConnection` | Connection string | Yes | `Server=...;Database=...;User Id=...;Password=...` |
| `EmailSettings__SmtpHost` | SMTP server | Yes | `smtp.gmail.com` |
| `EmailSettings__SmtpPort` | SMTP port | Yes | `587` |
| `EmailSettings__SmtpUsername` | SMTP user | Yes | `your-email@gmail.com` |
| `EmailSettings__SmtpPassword` | SMTP password | Yes | `YOUR_APP_PASSWORD` |
| `EmailSettings__FromEmail` | Sender email | Yes | `noreply@yourdomain.com` |
| `EmailSettings__FromName` | Sender display name | No | `Online Course System` |
| `Firebase__CredentialsPath` | Firebase Admin JSON path | Yes (push) | `/secrets/firebase-adminsdk.json` |
| `RabbitMq__Host` | RabbitMQ host | Yes | `rabbitmq.yourdomain.com` |
| `RabbitMq__VirtualHost` | RabbitMQ vhost | No | `/` |
| `RabbitMq__Username` | RabbitMQ user | Yes | `YOUR_RABBITMQ_USER` |
| `RabbitMq__Password` | RabbitMQ password | Yes | `YOUR_RABBITMQ_PASSWORD` |
| `WorkerSettings__EmailBatchSize` | Email worker batch size | No | `10` |
| `WorkerSettings__EmailPollingIntervalSeconds` | Email worker interval | No | `30` |
| `WorkerSettings__PushBatchSize` | Push worker batch size | No | `10` |
| `WorkerSettings__PushPollingIntervalSeconds` | Push worker interval | No | `30` |
| `WorkerSettings__ScheduledBatchSize` | Scheduled worker batch size | No | `10` |
| `WorkerSettings__ScheduledPollingIntervalSeconds` | Scheduled worker interval | No | `60` |
| `ASPNETCORE_ENVIRONMENT` | Environment name | Yes | `Production` |
| `ASPNETCORE_URLS` | Listening URLs | No | `https://+:443;http://+:80` |

---

## 3. Example appsettings.Production.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SQL_SERVER;Database=OnlineCourseDB;User Id=YOUR_DB_USER;Password=YOUR_DB_PASSWORD;TrustServerCertificate=True;Encrypt=True"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "EmailSettings": {
    "SmtpHost": "smtp.gmail.com",
    "SmtpPort": "587",
    "SmtpUsername": "YOUR_SMTP_USERNAME",
    "SmtpPassword": "YOUR_SMTP_PASSWORD",
    "FromEmail": "YOUR_FROM_EMAIL",
    "FromName": "Online Course System"
  },
  "Firebase": {
    "CredentialsPath": "/secrets/firebase-adminsdk.json"
  },
  "RabbitMq": {
    "Host": "YOUR_RABBITMQ_HOST",
    "VirtualHost": "/",
    "Username": "YOUR_RABBITMQ_USERNAME",
    "Password": "YOUR_RABBITMQ_PASSWORD"
  },
  "WorkerSettings": {
    "EmailBatchSize": "10",
    "EmailPollingIntervalSeconds": "30",
    "PushBatchSize": "10",
    "PushPollingIntervalSeconds": "30",
    "ScheduledBatchSize": "10",
    "ScheduledPollingIntervalSeconds": "60"
  }
}
```

---

## 4. Firebase Configuration

The **backend** uses Firebase Admin SDK (service account JSON). The **frontend** uses Firebase Web SDK with a different config.

### Backend (Admin SDK)

- **Credentials**: Service account JSON file at `Firebase:CredentialsPath`
- **Purpose**: Send push notifications to devices via FCM
- **Keep secret**: Never expose the service account JSON to the frontend

### Frontend (Web SDK)

The web/mobile app needs the following to request push permission and obtain FCM tokens:

```javascript
// Firebase Web Config
const firebaseConfig = {
  apiKey: "YOUR_FIREBASE_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const vapidKey = "YOUR_PUBLIC_VAPID_KEY";

```

---

## 5. API Base URLs & Ports

### Default Ports (Development)

| Profile | HTTP | HTTPS |
|---------|------|-------|
| http | 5122 | — |
| https | 5122 | 7108 |

### Required Ports

- **Inbound**: 80 (HTTP), 443 (HTTPS)
- **Outbound**: 1433 (SQL), 5672 (RabbitMQ AMQP), 587 (SMTP), 443 (Firebase/HTTPS)

---
