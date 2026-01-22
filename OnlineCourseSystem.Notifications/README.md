# OnlineCourseSystem.Notifications

![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=flat-square&logo=dotnet&logoColor=white)
![C#](https://img.shields.io/badge/C%23-12.0-239120?style=flat-square&logo=c-sharp&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL%20Server-2022+-CC2927?style=flat-square&logo=microsoft-sql-server&logoColor=white)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-FF6600?style=flat-square&logo=rabbitmq&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat-square&logo=firebase&logoColor=black)
![Status](https://img.shields.io/badge/Status-Production%20Ready-success?style=flat-square)

Enterprise-grade notifications microservice for the Codeway Online Courses Platform. Handles multi-channel delivery (in-app, email, push) with event-driven architecture, outbox pattern, and comprehensive user preference management.

---

## 🎯 Overview

Centralized notification hub that orchestrates multi-channel communication across the Codeway platform. Supports REST APIs and event-driven integration via RabbitMQ.

**Core Capabilities:**
- Multi-channel delivery (In-app, Email, Push)
- User preference management per notification type
- Event-driven architecture (MassTransit + RabbitMQ)
- Outbox pattern for reliable delivery
- Scheduled notifications
- Direct messaging between users
- Device management for push notifications

---

## 🏗️ Architecture

```
External Services → REST API / RabbitMQ Events
         ↓
Controllers → CQRS (MediatR) → Services → Repositories → SQL Server
         ↓
Background Workers (EmailOutbox, PushOutbox, Scheduled)
```

**Data Flow:**
1. Request/Event → Validation → User Verification
2. `sp_CreateNotification` (stored procedure) creates notification + outbox items atomically
3. Background workers process outbox items asynchronously
4. Delivery tracking across all channels

---

## 🛠️ Tech Stack

- **.NET 8.0** + ASP.NET Core Web API
- **EF Core 8.0** + SQL Server (stored procedures with TVPs)
- **MassTransit 8.2** + RabbitMQ
- **MediatR 14.0** (CQRS)
- **FluentValidation 12.1**
- **Firebase Admin SDK** (FCM push notifications)

---

## 🚀 Quick Start

### Prerequisites
- .NET 8.0 SDK
- SQL Server 2022+
- RabbitMQ (for events)
- Firebase project (for push)

### Setup

1. **Restore & Configure**
   ```powershell
   dotnet restore OnlineCourseSystem.sln
   ```
   Update `appsettings.json` with connection strings and credentials.

2. **Database Setup**
   ```powershell
   dotnet ef database update --project OnlineCourseSystem.Notifications
   ```
   Execute `Infrastructure/Database/SP_CreateNotification.sql`

3. **Run**
   ```powershell
   dotnet run --project OnlineCourseSystem.Notifications
   ```
   Swagger UI: `https://localhost:5001/swagger`

---

## ⚙️ Configuration

| Setting | Description |
|---------|-------------|
| `ConnectionStrings:DefaultConnection` | SQL Server connection string |
| `RabbitMq:Host/Username/Password` | RabbitMQ connection |
| `EmailSettings:*` | SMTP configuration |
| `Firebase:CredentialsPath` | Firebase service account JSON path |
| `WorkerSettings:*` | Batch sizes and polling intervals |

Override via environment variables: `ConnectionStrings__DefaultConnection=...`

---

## 📚 API Examples

### Create Notification
```http
POST /api/notifications
{
  "notificationType": "Course",
  "title": "New lesson available!",
  "content": "Lesson 5 is now available.",
  "courseId": "5f2c8c2c-18d8-46b0-b4e9-f0e4dc8e6a61",
  "userIds": ["c8c49232-4c8a-4f8c-8de1-4a70c7ad1140"]
}
```

### Get User Notifications
```http
GET /api/notifications?userId={userId}&isRead=false&pageNumber=1&pageSize=20
```

### Update Preferences
```http
POST /api/notificationpreferences/{userId}
{
  "notificationType": "Reminder",
  "email": true,
  "push": false,
  "inApp": true
}
```

### Schedule Notification
```http
POST /api/schedulednotifications
{
  "notificationType": "Reminder",
  "title": "Assignment due soon",
  "content": "Your assignment is due in 2 days",
  "userIds": ["user-id-1"],
  "scheduledFor": "2024-12-25T10:00:00Z"
}
```

---

## 🔄 Background Workers

- **EmailOutboxWorker**: Processes pending emails (batch size: 10, interval: 30s)
- **PushOutboxWorker**: Sends push notifications via FCM (batch size: 10, interval: 30s)
- **ScheduledNotificationWorker**: Processes scheduled notifications (batch size: 10, interval: 60s)

All workers include retry logic with exponential backoff.

---

## 📡 Event-Driven Integration

Consumes events via RabbitMQ:

- **NotificationRequestedEvent**: Creates notifications for specified users
- **UserEnrolledEvent**: Auto-creates welcome notifications
- **PaymentSucceededEvent**: Creates payment confirmation notifications

Events are automatically consumed via MassTransit consumers registered in `Program.cs`.

---

## 🗄️ Database

**Core Tables:** Notifications, UserNotifications, NotificationPreferences, UserReferences, UserNotificationDeliveries

**Outbox Tables:** EmailOutbox, PushOutbox, ScheduledNotifications

**Stored Procedures:**
- `sp_CreateNotification`: Atomic notification creation with TVP
- `sp_GetUserNotifications`: Paginated user notification retrieval

---

## ✅ Validation & Error Handling

- **FluentValidation**: Automatic validation via MediatR pipeline behaviors
- **ExceptionMiddleware**: Global exception handler with structured logging
- **Custom Exceptions**: `BadRequestException`, `NotFoundException`
- **Standardized Responses**: Via `GlobalResponse.Shared`

---

## 🧪 Development

```powershell
# Run tests
dotnet test OnlineCourseSystem.sln

# Format code
dotnet format OnlineCourseSystem.Notifications

# Create migration
dotnet ef migrations add <Name> --project OnlineCourseSystem.Notifications

# Apply migration
dotnet ef database update --project OnlineCourseSystem.Notifications
```

---

## 🚢 Deployment

```powershell
dotnet publish OnlineCourseSystem.Notifications -c Release -o ./publish/notifications
```

**Checklist:**
- [ ] Production connection strings configured
- [ ] RabbitMQ cluster setup (if using events)
- [ ] Firebase credentials configured
- [ ] SMTP credentials configured
- [ ] Database migrations applied
- [ ] Stored procedures executed
- [ ] Logging/monitoring configured
- [ ] Worker settings tuned

---

## 📁 Project Structure

```
OnlineCourseSystem.Notifications/
├── Controllers/          # REST endpoints
├── Consumers/            # MassTransit event consumers
├── Features/            # CQRS feature modules (Commands/Queries/Handlers)
├── Infrastructure/
│   ├── Database/        # Stored procedures
│   ├── Repositories/     # Data access (Unit of Work)
│   └── Services/         # Domain services
├── Workers/              # Background services
├── Models/               # Entities, DbContext, Enums
└── Validators/           # FluentValidation rules
```

---

## 🤝 Contributing

1. Fork & create feature branch
2. Follow coding standards (nullable ref types, async/await, logging)
3. Add tests for new functionality
4. Run `dotnet format` and `dotnet test`
5. Submit PR with detailed description

---

**Built with ❤️ for the Codeway Online Courses Platform**
