# OnlineCourseSystem.Notifications

![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=flat-square&logo=dotnet&logoColor=white)
![C#](https://img.shields.io/badge/C%23-12.0-239120?style=flat-square&logo=c-sharp&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL%20Server-2022+-CC2927?style=flat-square&logo=microsoft-sql-server&logoColor=white)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-FF6600?style=flat-square&logo=rabbitmq&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat-square&logo=firebase&logoColor=black)
![Status](https://img.shields.io/badge/Status-Production%20Ready-success?style=flat-square)

A comprehensive, enterprise-grade notifications microservice built with ASP.NET Core 8, designed to handle multi-channel notification delivery for the Codeway Online Courses platform. This service provides robust, scalable, and reliable notification infrastructure supporting in-app notifications, email delivery, and push notifications across multiple platforms.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [API Documentation](#api-documentation)
- [Background Workers](#background-workers)
- [Event-Driven Integration](#event-driven-integration)
- [Database Schema](#database-schema)
- [Validation & Error Handling](#validation--error-handling)
- [Testing](#testing)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

---

## 🎯 Overview

The **OnlineCourseSystem.Notifications** microservice is a centralized notification hub that orchestrates multi-channel communication across the Codeway platform. It handles notification creation, user preference management, delivery tracking, and provides both RESTful APIs and event-driven integration capabilities.

### Core Capabilities

- **Multi-Channel Delivery**: In-app notifications, email, and push notifications (iOS/Android)
- **User Preferences**: Granular control over notification channels per notification type
- **Event-Driven Architecture**: Seamless integration via RabbitMQ message bus
- **Reliable Delivery**: Outbox pattern ensures message delivery guarantees
- **Scheduled Notifications**: Support for time-based notification scheduling
- **Direct Messaging**: User-to-user messaging capabilities
- **Device Management**: Register and manage user devices for push notifications
- **Delivery Tracking**: Comprehensive tracking of notification delivery status

---

## ✨ Key Features

### 🔔 Multi-Channel Notification System
- **In-App Notifications**: Real-time notifications stored in database with read/unread status
- **Email Notifications**: SMTP-based email delivery with retry mechanisms
- **Push Notifications**: Firebase Cloud Messaging (FCM) integration for iOS and Android

### 🎛️ User Preference Management
- Per-user, per-notification-type channel preferences
- Lazy initialization of default preferences
- Granular control (In-App, Email, Push) for each notification type

### 📨 Event-Driven Integration
- **MassTransit + RabbitMQ**: Reliable message bus integration
- **Event Consumers**: Automatic notification creation from domain events
- **Integration Events**: `NotificationRequestedEvent`, `UserEnrolledEvent`, `PaymentSucceededEvent`

### 🔄 Outbox Pattern Implementation
- **EmailOutbox**: Reliable email delivery with retry logic
- **PushOutbox**: Reliable push notification delivery
- **Background Workers**: Dedicated workers process outbox items asynchronously

### ⏰ Scheduled Notifications
- Schedule notifications for future delivery
- Support for reminders, course deadlines, and time-based alerts
- Background worker processes scheduled items automatically

### 💬 Direct Messaging
- User-to-user messaging system
- Conversation-based message retrieval
- Automatic read status tracking

### 📱 Device Management
- Register user devices for push notifications
- Support for multiple platforms (iOS, Android)
- Device token management and validation

### 🛡️ Reliability & Resilience
- **Retry Mechanisms**: Exponential backoff for transient failures
- **Transaction Safety**: Stored procedures ensure atomic operations
- **Error Handling**: Comprehensive exception handling with structured logging
- **Delivery Tracking**: Track delivery status across all channels

---

## 🏗️ Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    External Services                         │
│  (Auth Service, Course Service, Payment Service, etc.)      │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       │ REST API / RabbitMQ Events
                       │
┌──────────────────────▼──────────────────────────────────────┐
│              Notifications Microservice                      │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Controllers Layer                        │  │
│  │  (Notifications, Preferences, Messages, Devices)     │  │
│  └──────────────┬───────────────────────────────────────┘  │
│                 │                                           │
│  ┌──────────────▼───────────────────────────────────────┐  │
│  │         CQRS Layer (MediatR)                         │  │
│  │  (Commands, Queries, Handlers, Validators)          │  │
│  └──────────────┬───────────────────────────────────────┘  │
│                 │                                           │
│  ┌──────────────▼───────────────────────────────┐  │
│  │      Service Layer                                   │  │
│  │  • NotificationService                               │  │
│  │  • EmailService                                     │  │
│  │  • PushNotificationService                          │  │
│  └──────────────┬──────────────────────────────────────┘  │
│                 │                                           │
│  ┌──────────────▼───────────────────────────────────────┐  │
│  │      Repository Layer (Unit of Work)                 │  │
│  └──────────────┬───────────────────────────────────────┘  │
│                 │                                           │
│  ┌──────────────▼───────────────────────────────────────┐  │
│  │      Database Layer (EF Core + SQL Server)          │  │
│  │  • Stored Procedures (sp_CreateNotification)        │  │
│  │  • Table-Valued Parameters                          │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │      Background Workers                              │  │
│  │  • EmailOutboxWorker                                 │  │
│  │  • PushOutboxWorker                                  │  │
│  │  • ScheduledNotificationWorker                       │  │
│  └──────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### Data Flow: Notification Creation

1. **Request Received**: REST API or RabbitMQ event triggers notification creation
2. **Validation**: FluentValidation validates request payload
3. **User Verification**: Verify all target users exist in UserReferences
4. **Stored Procedure Execution**: `sp_CreateNotification` performs atomic operation:
   - Creates notification record
   - Creates user notifications based on preferences
   - Inserts into EmailOutbox (if email enabled)
   - Inserts into PushOutbox (if push enabled)
5. **Background Processing**: Workers process outbox items asynchronously
6. **Delivery Tracking**: Delivery status tracked in UserNotificationDeliveries

---

## 🛠️ Technology Stack

### Core Framework
- **.NET 8.0** - Latest LTS version with performance improvements
- **ASP.NET Core Web API** - RESTful API framework
- **C# 12** - Modern C# features and nullable reference types

### Data Access
- **Entity Framework Core 8.0** - ORM with SQL Server provider
- **SQL Server 2019+** - Relational database
- **Stored Procedures** - Optimized batch operations with TVPs

### Messaging & Events
- **MassTransit 8.2** - Message bus abstraction
- **RabbitMQ** - Message broker for event-driven architecture
- **Integration Events** - Domain event contracts

### Validation & CQRS
- **MediatR 14.0** - CQRS pattern implementation
- **FluentValidation 12.1** - Fluent validation library
- **Validation Behaviors** - Pipeline behaviors for automatic validation

### Push Notifications
- **Firebase Admin SDK 3.4** - Firebase Cloud Messaging integration
- **Google.Apis.Auth 1.73** - Google authentication

### Documentation & Testing
- **Swashbuckle (Swagger)** - API documentation
- **XML Documentation** - Code documentation generation

### Shared Libraries
- **GlobalResponse.Shared** - Standardized API response contracts
- **OnlineCourseSystem.Contracts.Messaging** - Event contracts

---

## 📁 Project Structure

```
OnlineCourseSystem.Notifications/
├── Behaviors/                          # MediatR pipeline behaviors
│   └── ValidationBehavior.cs           # Automatic validation
│
├── Consumers/                          # MassTransit event consumers
│   ├── NotificationRequestedConsumer.cs
│   ├── UserEnrolledConsumer.cs
│   └── PaymentSucceededConsumer.cs
│
├── Controllers/                        # REST API endpoints
│   ├── NotificationsController.cs      # Notification CRUD operations
│   ├── NotificationPreferencesController.cs
│   ├── MessagesController.cs           # Direct messaging
│   ├── ScheduledNotificationsController.cs
│   └── UserDevicesController.cs       # Device registration
│
├── Exceptions/                         # Custom exception types
│   └── NotificationExceptions.cs
│
├── Features/                           # CQRS feature modules
│   ├── Messages/                      # Direct messaging feature
│   │   ├── Commands/
│   │   ├── Queries/
│   │   └── DTOs/
│   ├── NotificationPreference/        # Preference management
│   ├── Notifications/                 # Core notification feature
│   ├── ScheduledNotifications/         # Scheduled notifications
│   └── UserDevices/                   # Device management
│
├── Infrastructure/
│   ├── Database/                      # Database artifacts
│   │   ├── AllTables.sql              # Table creation scripts
│   │   ├── SP_CreateNotification.sql # Stored procedure
│   │   └── SP_GetUserNotifications.sql
│   ├── Repositories/                  # Data access layer
│   │   ├── Interfaces/
│   │   ├── UnitOfWork/
│   │   └── [Repository implementations]
│   └── Services/                      # Domain services
│       ├── Interfaces/
│       ├── NotificationService.cs
│       ├── EmailService.cs
│       └── PushNotificationService.cs
│
├── Middlewares/                        # HTTP pipeline middleware
│   └── ExceptionMiddleware.cs         # Global exception handling
│
├── Models/                            # Domain models
│   ├── Data/
│   │   ├── NotificationsDbContext.cs
│   │   └── NotificationsDbContextFactory.cs
│   ├── Enums/
│   │   ├── NotificationType.cs
│   │   ├── OutboxStatus.cs
│   │   └── Platform.cs
│   └── [Entity models]
│
├── Validators/                         # FluentValidation validators
│   └── Notification/
│
├── Workers/                           # Background services
│   ├── EmailOutboxWorker.cs           # Email delivery worker
│   ├── PushOutboxWorker.cs            # Push notification worker
│   └── ScheduledNotificationWorker.cs # Scheduled notification worker
│
├── appsettings.json                   # Configuration
├── Program.cs                         # Application entry point
└── OnlineCourseSystem.Notifications.csproj
```

---

## 🚀 Getting Started

### Prerequisites

- **.NET 8.0 SDK** (8.0.100 or later)
- **SQL Server 2022+** (local instance or remote server)
- **RabbitMQ Server** (for event-driven features)
- **Firebase Project** (for push notifications)
- **EF Core Tools** (for migrations):
  ```powershell
  dotnet tool install --global dotnet-ef
  ```

### Installation Steps

1. **Clone the Repository**
   ```powershell
   git clone <repository-url>
   cd codeway-online-courses-platform
   ```

2. **Restore Dependencies**
   ```powershell
   dotnet restore OnlineCourseSystem.sln
   ```

3. **Configure Connection String**
   
   Update `appsettings.json` or set environment variables:
   ```json
   {
     "ConnectionStrings": {
       "DefaultConnection": "Server=.;Database=OnlineCourseDB;Integrated Security=True;TrustServerCertificate=True"
     }
   }
   ```

4. **Configure RabbitMQ** (optional, for event-driven features)
   ```json
   {
     "RabbitMq": {
       "Host": "localhost",
       "VirtualHost": "/",
       "Username": "guest",
       "Password": "guest"
     }
   }
   ```

5. **Configure Firebase** (for push notifications)
   - Download Firebase service account JSON
   - Set path in `appsettings.json`:
     ```json
     {
       "Firebase": {
         "CredentialsPath": "path/to/firebase-credentials.json"
       }
     }
     ```

6. **Apply Database Migrations**
   ```powershell
   dotnet ef database update `
     --project OnlineCourseSystem.Notifications `
     --startup-project OnlineCourseSystem.Notifications
   ```

7. **Create Database Artifacts**
   
   Execute the stored procedure script:
   ```sql
   -- Run Infrastructure/Database/SP_CreateNotification.sql
   ```

8. **Seed Initial Data** (if required)
   - Populate `UserReferences` table with user data
   - Notification preferences are created lazily on first access

9. **Run the Service**
   ```powershell
   dotnet run --project OnlineCourseSystem.Notifications
   ```

10. **Access Swagger UI**
    
    Navigate to: `https://localhost:5001/swagger` (or configured port)

---

## ⚙️ Configuration

### Application Settings

| Setting | Description | Default |
|---------|-------------|---------|
| `ConnectionStrings:DefaultConnection` | SQL Server connection string | Required |
| `RabbitMq:Host` | RabbitMQ server hostname | `localhost` |
| `RabbitMq:VirtualHost` | RabbitMQ virtual host | `/` |
| `RabbitMq:Username` | RabbitMQ username | `guest` |
| `RabbitMq:Password` | RabbitMQ password | `guest` |
| `EmailSettings:SmtpHost` | SMTP server hostname | Required |
| `EmailSettings:SmtpPort` | SMTP server port | `587` |
| `EmailSettings:SmtpUsername` | SMTP username | Required |
| `EmailSettings:SmtpPassword` | SMTP password | Required |
| `EmailSettings:FromEmail` | Sender email address | Required |
| `Firebase:CredentialsPath` | Path to Firebase credentials JSON | Required |
| `WorkerSettings:EmailBatchSize` | Email worker batch size | `10` |
| `WorkerSettings:EmailPollingIntervalSeconds` | Email polling interval | `30` |
| `WorkerSettings:PushBatchSize` | Push worker batch size | `10` |
| `WorkerSettings:PushPollingIntervalSeconds` | Push polling interval | `30` |
| `WorkerSettings:ScheduledBatchSize` | Scheduled worker batch size | `10` |
| `WorkerSettings:ScheduledPollingIntervalSeconds` | Scheduled polling interval | `60` |

### Environment Variables

All settings can be overridden using environment variables with double underscore notation:

```powershell
$env:ConnectionStrings__DefaultConnection = "Server=...;Database=..."
$env:EmailSettings__SmtpHost = "smtp.gmail.com"
```

---

## 📚 API Documentation

### Notifications API

#### Create Notification
```http
POST /api/notifications
Content-Type: application/json

{
  "notificationType": "Course",
  "title": "New lesson available!",
  "content": "Lesson 5: Advanced Topics is now available.",
  "courseId": "5f2c8c2c-18d8-46b0-b4e9-f0e4dc8e6a61",
  "userIds": [
    "c8c49232-4c8a-4f8c-8de1-4a70c7ad1140",
    "15a07664-ef29-4f55-b93d-4fbc0c8337aa"
  ]
}
```

**Response:**
```json
{
  "success": true,
  "message": "Notification created successfully",
  "data": [
    "user-notification-id-1",
    "user-notification-id-2"
  ]
}
```

#### Get User Notifications
```http
GET /api/notifications?userId={userId}&isRead={true|false}&pageNumber=1&pageSize=20
```

#### Mark Notification as Read
```http
POST /api/notifications/{userNotificationId}/read
```

#### Get Notification Types
```http
GET /api/notifications/Types
```

### Notification Preferences API

#### Get User Preferences
```http
GET /api/notificationpreferences/{userId}
```

#### Update Preference
```http
POST /api/notificationpreferences/{userId}
Content-Type: application/json

{
  "notificationType": "Reminder",
  "email": true,
  "push": false,
  "inApp": true
}
```

### Messages API

#### Send Message
```http
POST /api/messages
Content-Type: application/json

{
  "senderId": "user-id-1",
  "receiverId": "user-id-2",
  "content": "Hello, how are you?",
  "courseId": "optional-course-id"
}
```

#### Get Conversation
```http
GET /api/messages?conversationId={conversationId}&readerId={userId}&pageNumber=1&pageSize=50
```

### Scheduled Notifications API

#### Schedule Notification
```http
POST /api/schedulednotifications
Content-Type: application/json

{
  "notificationType": "Reminder",
  "title": "Assignment due soon",
  "content": "Your assignment is due in 2 days",
  "userIds": ["user-id-1", "user-id-2"],
  "scheduledFor": "2024-12-25T10:00:00Z",
  "courseId": "optional-course-id"
}
```

### User Devices API

#### Register Device
```http
POST /api/userdevices
Content-Type: application/json

{
  "userId": "user-id",
  "deviceToken": "fcm-device-token",
  "platform": "Android"
}
```

---

## 🔄 Background Workers

### EmailOutboxWorker

Processes pending email notifications from the `EmailOutbox` table.

- **Batch Processing**: Configurable batch size (default: 10)
- **Polling Interval**: Configurable interval (default: 30 seconds)
- **Retry Logic**: Automatic retry with exponential backoff
- **Failure Handling**: Marks items as failed after max retries

### PushOutboxWorker

Processes pending push notifications from the `PushOutbox` table.

- **Firebase Integration**: Sends notifications via FCM
- **Device Management**: Retrieves device tokens for users
- **Multi-Platform**: Supports iOS and Android
- **Retry Logic**: Automatic retry with exponential backoff

### ScheduledNotificationWorker

Processes scheduled notifications that are due for delivery.

- **Time-Based Processing**: Checks for notifications scheduled for current time
- **Batch Processing**: Processes multiple scheduled notifications
- **Integration**: Uses NotificationService for actual delivery

---

## 📡 Event-Driven Integration

### Integration Events

The service consumes the following events via RabbitMQ:

#### NotificationRequestedEvent
```csharp
{
  "eventId": "guid",
  "userIds": ["guid1", "guid2"],
  "notificationType": "Course",
  "title": "Notification title",
  "content": "Notification content",
  "courseId": "optional-guid"
}
```

#### UserEnrolledEvent
Automatically creates welcome notifications when users enroll in courses.

#### PaymentSucceededEvent
Creates payment confirmation notifications.

### Event Consumer Configuration

Events are automatically consumed via MassTransit consumers registered in `Program.cs`:

```csharp
x.AddConsumers(typeof(Program).Assembly);
```

---

## 🗄️ Database Schema

### Core Tables

- **Notifications**: Master notification records
- **UserNotifications**: User-specific notification instances
- **NotificationPreferences**: User channel preferences per notification type
- **UserReferences**: Reference table for valid users
- **UserNotificationDeliveries**: Delivery tracking across channels

### Outbox Tables

- **EmailOutbox**: Pending email notifications
- **PushOutbox**: Pending push notifications
- **ScheduledNotifications**: Scheduled notification records

### Messaging Tables

- **Messages**: Direct user-to-user messages
- **UserDevices**: Registered devices for push notifications

### Stored Procedures

- **sp_CreateNotification**: Atomic notification creation with TVP
- **sp_GetUserNotifications**: Paginated user notification retrieval

---

## ✅ Validation & Error Handling

### FluentValidation

All requests are validated using FluentValidation:

- **CreateNotificationValidator**: Validates notification creation requests
- **Automatic Validation**: MediatR pipeline behavior validates all commands/queries
- **Structured Errors**: Returns standardized error responses

### Exception Handling

- **ExceptionMiddleware**: Global exception handler
- **Custom Exceptions**: `BadRequestException`, `NotFoundException`
- **Structured Logging**: Comprehensive logging with correlation IDs
- **Error Responses**: Consistent error response format via `GlobalResponse.Shared`

---

## 🧪 Testing

### Running Tests

```powershell
dotnet test OnlineCourseSystem.sln
```

### Code Formatting

```powershell
dotnet format OnlineCourseSystem.Notifications
```

### Database Migrations

**Create Migration:**
```powershell
dotnet ef migrations add <MigrationName> `
  --project OnlineCourseSystem.Notifications `
  --startup-project OnlineCourseSystem.Notifications
```

**Apply Migration:**
```powershell
dotnet ef database update `
  --project OnlineCourseSystem.Notifications `
  --startup-project OnlineCourseSystem.Notifications
```

---

## 🚢 Deployment

### Build for Production

```powershell
dotnet publish OnlineCourseSystem.Notifications `
  -c Release `
  -o ./publish/notifications
```

### Deployment Checklist

- [ ] Configure production connection string
- [ ] Set up RabbitMQ cluster (if using events)
- [ ] Configure Firebase credentials
- [ ] Set up SMTP server credentials
- [ ] Apply database migrations
- [ ] Execute stored procedure scripts
- [ ] Configure logging (Application Insights, ELK, etc.)
- [ ] Set up monitoring and alerting
- [ ] Configure HTTPS/TLS certificates
- [ ] Set up health checks
- [ ] Configure worker settings (batch sizes, intervals)
- [ ] Test all notification channels
- [ ] Verify event consumers are working

### Docker Deployment (Future)

Docker support can be added by creating a `Dockerfile`:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["OnlineCourseSystem.Notifications/OnlineCourseSystem.Notifications.csproj", "OnlineCourseSystem.Notifications/"]
RUN dotnet restore "OnlineCourseSystem.Notifications/OnlineCourseSystem.Notifications.csproj"
COPY . .
WORKDIR "/src/OnlineCourseSystem.Notifications"
RUN dotnet build "OnlineCourseSystem.Notifications.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "OnlineCourseSystem.Notifications.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "OnlineCourseSystem.Notifications.dll"]
```

---

## 🤝 Contributing

We welcome contributions! Please follow these guidelines:

1. **Fork the Repository**: Create your own fork
2. **Create Feature Branch**: `git checkout -b feature/amazing-feature`
3. **Follow Coding Standards**:
   - Use nullable reference types
   - Follow async/await patterns
   - Add comprehensive logging
   - Write XML documentation
4. **Add Tests**: Include unit and integration tests
5. **Run Checks**: 
   ```powershell
   dotnet format
   dotnet test
   ```
6. **Commit Changes**: Use descriptive commit messages
7. **Push to Branch**: `git push origin feature/amazing-feature`
8. **Create Pull Request**: Provide detailed description

### Coding Standards

- **C# 12 Features**: Use modern C# syntax
- **Nullable Reference Types**: Enable nullable context
- **Async/Await**: Use async methods for I/O operations
- **Logging**: Use structured logging with `ILogger<T>`
- **Documentation**: Add XML comments for public APIs
- **Error Handling**: Use custom exceptions with meaningful messages

---

## 📄 License

This project is part of the Codeway Online Courses Platform. License details will be finalized before public release.

---

## 📞 Support

For issues, questions, or contributions, please open an issue on the repository or contact the development team.

---

**Built with ❤️ for the Codeway Online Courses Platform**
