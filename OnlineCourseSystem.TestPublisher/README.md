# OnlineCourseSystem.TestPublisher

![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=flat-square&logo=dotnet&logoColor=white)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-FF6600?style=flat-square&logo=rabbitmq&logoColor=white)
![MassTransit](https://img.shields.io/badge/MassTransit-9.0.0-512BD4?style=flat-square)
![Status](https://img.shields.io/badge/Status-Development%20Tool-blue?style=flat-square)

Console utility for testing event-driven architecture. Manually publish integration events to RabbitMQ to test event consumers without running the full application stack.

---

## 🎯 Purpose

- Test event consumers in the Notifications microservice
- Debug event flow and consumer behavior
- Validate event contracts and serialization
- Simulate scenarios during development

---

## 🚀 Quick Start

### Prerequisites
- .NET 8.0 SDK
- RabbitMQ Server (local or remote)

### Setup

1. **Install RabbitMQ** (Docker recommended)
   ```powershell
   docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
   ```
   Management UI: `http://localhost:15672` (guest/guest)

2. **Configure**
   
   Create `appsettings.json`:
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

3. **Run**
   ```powershell
   dotnet run
   ```

---

## 📖 Usage

### Interactive Menu

```
Publisher started.
1) Publish NotificationRequestedEvent
2) Publish PaymentSucceededEvent
3) Publish UserEnrolledEvent
0) Exit

Choose:
```

### Examples

**Publish NotificationRequestedEvent:**
```
Choose: 1
Enter UserId (GUID): c8c49232-4c8a-4f8c-8de1-4a70c7ad1140
NotificationRequestedEvent published!
```

**Publish UserEnrolledEvent:**
```
Choose: 3
Enter UserId (GUID): c8c49232-4c8a-4f8c-8de1-4a70c7ad1140
Enter CourseId (GUID): 5f2c8c2c-18d8-46b0-b4e9-f0e4dc8e6a61
UserEnrolledEvent published!
```

---

## 📨 Supported Events

### NotificationRequestedEvent
- **UserIds**: List of user GUIDs
- **NotificationType**: "System", "Course", "Reminder", "Announcement"
- **Title/Content**: Notification details
- **Test Values**: Type="System", Title="RabbitMQ Test Notification"

### PaymentSucceededEvent
- **UserId**: User who made payment
- **Amount**: 200 (default)
- **Currency**: "EGP" (default)
- **Description**: "Test payment from Publisher Service"

### UserEnrolledEvent
- **UserId**: User who enrolled
- **CourseId**: Course GUID
- **CourseName**: "Test Course" (default)

---

## ⚙️ Configuration

**Environment Variables:**
```powershell
$env:RabbitMq__Host = "rabbitmq.example.com"
$env:RabbitMq__Username = "myuser"
$env:RabbitMq__Password = "mypassword"
```

---

## 🔍 Troubleshooting

**Cannot connect to RabbitMQ:**
- Verify RabbitMQ is running: `docker ps | findstr rabbitmq`
- Check connection settings in `appsettings.json`
- Test connectivity: `Test-NetConnection -ComputerName localhost -Port 5672`
- Check Management UI: `http://localhost:15672`

**Invalid GUID Error:**
- Use format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
- Example: `c8c49232-4c8a-4f8c-8de1-4a70c7ad1140`

**Events not consumed:**
- Verify Notifications microservice is running
- Check RabbitMQ Management UI for queues/consumers
- Verify connection settings match in both services
- Review Notifications service logs

---

## 🛠️ Development

**Project Structure:**
```
OnlineCourseSystem.TestPublisher/
├── Program.cs              # Main entry point
└── appsettings.json        # Configuration
```

**Dependencies:**
- MassTransit 9.0.0
- MassTransit.RabbitMQ 9.0.0
- OnlineCourseSystem.Contracts.Messaging

**Extending:**
1. Add menu option in `Program.cs`
2. Add event publishing logic
3. Rebuild and test

---

## 📝 Example Workflow

1. **Start Services:**
   ```powershell
   # Terminal 1: RabbitMQ
   docker start rabbitmq
   
   # Terminal 2: Notifications Service
   cd OnlineCourseSystem.Notifications
   dotnet run
   
   # Terminal 3: Test Publisher
   cd OnlineCourseSystem.TestPublisher
   dotnet run
   ```

2. **Publish Event** via interactive menu

3. **Verify** in Notifications service logs and RabbitMQ Management UI

---

## 🔐 Security

⚠️ **Development tool only** - Do not use in production.

- Don't commit production credentials
- Use environment variables for sensitive config
- Ensure RabbitMQ is secured in production

---

**Happy Testing! 🚀**
