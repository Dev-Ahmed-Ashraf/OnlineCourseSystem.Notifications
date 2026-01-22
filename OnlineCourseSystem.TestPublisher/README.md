# OnlineCourseSystem.TestPublisher

![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=flat-square&logo=dotnet&logoColor=white)
![C#](https://img.shields.io/badge/C%23-12.0-239120?style=flat-square&logo=c-sharp&logoColor=white)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-FF6600?style=flat-square&logo=rabbitmq&logoColor=white)
![MassTransit](https://img.shields.io/badge/MassTransit-9.0.0-512BD4?style=flat-square)
![Status](https://img.shields.io/badge/Status-Development%20Tool-blue?style=flat-square)

A console application utility for testing and debugging the event-driven architecture of the Codeway Online Courses Platform. This tool allows developers to manually publish integration events to RabbitMQ, enabling easy testing of event consumers and the notifications microservice without requiring the full application stack.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Purpose](#purpose)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Usage](#usage)
- [Supported Events](#supported-events)
- [Troubleshooting](#troubleshooting)
- [Development](#development)

---

## 🎯 Overview

**OnlineCourseSystem.TestPublisher** is a lightweight console application designed to facilitate testing of the event-driven microservices architecture. It provides an interactive command-line interface for publishing integration events to RabbitMQ, making it easy to:

- Test event consumers without running the entire application
- Debug event flow and consumer behavior
- Validate event contracts and message serialization
- Simulate real-world scenarios during development

---

## 🎯 Purpose

This tool serves as a **development and testing utility** for the Codeway platform's event-driven architecture. It enables developers to:

1. **Test Event Consumers**: Verify that event consumers in the Notifications microservice are working correctly
2. **Debug Event Flow**: Troubleshoot issues with event publishing and consumption
3. **Validate Contracts**: Ensure event contracts are properly serialized and deserialized
4. **Simulate Scenarios**: Create test scenarios without requiring full system integration
5. **Development Workflow**: Speed up development by quickly testing event-driven features

---

## ✨ Features

- **Interactive CLI**: User-friendly command-line interface for event publishing
- **Multiple Event Types**: Support for all major integration events
- **GUID Validation**: Input validation for user IDs and course IDs
- **Correlation Tracking**: Automatic generation of correlation IDs and trace IDs
- **RabbitMQ Integration**: Direct integration with RabbitMQ via MassTransit
- **Easy Configuration**: Simple JSON-based configuration

---

## 🔧 Prerequisites

Before using this tool, ensure you have:

- **.NET 8.0 SDK** (8.0.100 or later)
- **RabbitMQ Server** running and accessible
  - Local installation, or
  - Remote server with network access
- **Access to Event Contracts**: The `OnlineCourseSystem.Contracts.Messaging` project must be available

### Installing RabbitMQ

#### Windows
```powershell
# Using Chocolatey
choco install rabbitmq

# Or download from: https://www.rabbitmq.com/download.html
```

#### Docker (Recommended)
```powershell
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

Access RabbitMQ Management UI at: `http://localhost:15672` (guest/guest)

---

## 🚀 Getting Started

### 1. Clone and Navigate

```powershell
cd codeway-online-courses-platform
cd OnlineCourseSystem.TestPublisher
```

### 2. Restore Dependencies

```powershell
dotnet restore
```

### 3. Configure RabbitMQ Connection

Create or update `appsettings.json` in the project root:

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

**Note**: If `appsettings.json` doesn't exist in the project root, you can create it. The application will also look for it in the output directory (`bin/Debug/net8.0/`).

### 4. Build the Project

```powershell
dotnet build
```

### 5. Run the Publisher

```powershell
dotnet run
```

---

## ⚙️ Configuration

### Configuration File

The application reads configuration from `appsettings.json`:

```json
{
  "RabbitMq": {
    "Host": "localhost",           // RabbitMQ server hostname
    "VirtualHost": "/",            // RabbitMQ virtual host
    "Username": "guest",           // RabbitMQ username
    "Password": "guest"            // RabbitMQ password
  }
}
```

### Environment Variables

You can override configuration using environment variables:

```powershell
$env:RabbitMq__Host = "rabbitmq.example.com"
$env:RabbitMq__Username = "myuser"
$env:RabbitMq__Password = "mypassword"
```

### Remote RabbitMQ Configuration

For connecting to a remote RabbitMQ server:

```json
{
  "RabbitMq": {
    "Host": "rabbitmq.example.com",
    "VirtualHost": "production",
    "Username": "your-username",
    "Password": "your-password"
  }
}
```

---

## 📖 Usage

### Starting the Application

Run the application:

```powershell
dotnet run
```

You'll see the main menu:

```
Publisher started.
1) Publish NotificationRequestedEvent
2) Publish PaymentSucceededEvent
3) Publish UserEnrolledEvent
0) Exit

Choose:
```

### Publishing Events

#### 1. Publish NotificationRequestedEvent

Select option `1` and follow the prompts:

```
Choose: 1
Enter UserId (GUID): c8c49232-4c8a-4f8c-8de1-4a70c7ad1140
NotificationRequestedEvent published!
```

This publishes a notification request event that will trigger the `NotificationRequestedConsumer` in the Notifications microservice.

#### 2. Publish PaymentSucceededEvent

Select option `2`:

```
Choose: 2
Enter UserId (GUID): c8c49232-4c8a-4f8c-8de1-4a70c7ad1140
PaymentSucceededEvent published!
```

This simulates a successful payment event.

#### 3. Publish UserEnrolledEvent

Select option `3`:

```
Choose: 3
Enter UserId (GUID): c8c49232-4c8a-4f8c-8de1-4a70c7ad1140
Enter CourseId (GUID): 5f2c8c2c-18d8-46b0-b4e9-f0e4dc8e6a61
UserEnrolledEvent published!
```

This simulates a user enrollment event.

#### 4. Exit

Select option `0` to exit the application:

```
Choose: 0
Publisher stopped.
```

---

## 📨 Supported Events

### 1. NotificationRequestedEvent

Publishes a notification request that triggers notification creation in the Notifications microservice.

**Event Properties:**
- `UserIds`: List of user GUIDs to receive the notification
- `NotificationType`: Type of notification (e.g., "System", "Course", "Reminder")
- `Title`: Notification title
- `Content`: Notification content
- `CourseId`: Optional course ID (nullable)
- `CorrelationId`: Auto-generated correlation ID
- `TraceId`: Auto-generated trace ID

**Test Values:**
- NotificationType: `"System"`
- Title: `"RabbitMQ Test Notification"`
- Content: `"Hello from Publisher Service!"`

### 2. PaymentSucceededEvent

Simulates a successful payment transaction.

**Event Properties:**
- `UserId`: User who made the payment
- `Amount`: Payment amount (default: 200)
- `Currency`: Currency code (default: "EGP")
- `Description`: Payment description
- `CourseId`: Optional course ID (nullable)
- `CorrelationId`: Auto-generated correlation ID
- `TraceId`: Auto-generated trace ID

**Test Values:**
- Amount: `200`
- Currency: `"EGP"`
- Description: `"Test payment from Publisher Service"`

### 3. UserEnrolledEvent

Simulates a user enrollment in a course.

**Event Properties:**
- `UserId`: User who enrolled
- `CourseId`: Course ID
- `CourseName`: Course name (default: "Test Course")
- `CorrelationId`: Auto-generated correlation ID
- `TraceId`: Auto-generated trace ID

**Test Values:**
- CourseName: `"Test Course"`

---

## 🔍 Troubleshooting

### Connection Issues

**Problem**: Cannot connect to RabbitMQ

**Solutions:**
1. Verify RabbitMQ is running:
   ```powershell
   # Check if RabbitMQ is running (Docker)
   docker ps | findstr rabbitmq
   ```

2. Verify connection settings in `appsettings.json`

3. Check network connectivity:
   ```powershell
   Test-NetConnection -ComputerName localhost -Port 5672
   ```

4. Check RabbitMQ Management UI: `http://localhost:15672`

### Invalid GUID Error

**Problem**: "❌ Invalid GUID" or "Invalid GUID(s)"

**Solution**: Ensure you're entering valid GUID format:
- Format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
- Example: `c8c49232-4c8a-4f8c-8de1-4a70c7ad1140`

### Events Not Being Consumed

**Problem**: Events are published but not consumed by the Notifications service

**Solutions:**
1. Verify the Notifications microservice is running
2. Check that consumers are registered in the Notifications service
3. Verify RabbitMQ connection settings match in both services
4. Check RabbitMQ Management UI for message queues and consumers
5. Review logs in the Notifications service for errors

### Configuration File Not Found

**Problem**: Application cannot find `appsettings.json`

**Solution**: 
1. Create `appsettings.json` in the project root directory
2. Or copy from `bin/Debug/net8.0/appsettings.json` to project root
3. Ensure the file is included in the project (not excluded by `.gitignore`)

### MassTransit Connection Errors

**Problem**: MassTransit connection errors in console output

**Solutions:**
1. Verify RabbitMQ credentials are correct
2. Check virtual host exists (default: `/`)
3. Ensure user has permissions to publish to exchanges
4. Review RabbitMQ logs for detailed error messages

---

## 🛠️ Development

### Project Structure

```
OnlineCourseSystem.TestPublisher/
├── Program.cs                          # Main application entry point
├── OnlineCourseSystem.TestPublisher.csproj
└── appsettings.json                    # Configuration file
```

### Dependencies

- **MassTransit 9.0.0**: Message bus abstraction
- **MassTransit.RabbitMQ 9.0.0**: RabbitMQ transport
- **Microsoft.Extensions.Hosting 8.0.1**: Hosting abstractions
- **OnlineCourseSystem.Contracts.Messaging**: Event contracts

### Building from Source

```powershell
# Restore packages
dotnet restore

# Build
dotnet build

# Run
dotnet run

# Publish (standalone)
dotnet publish -c Release -o ./publish
```

### Extending the Tool

To add support for new event types:

1. **Add Menu Option** in `Program.cs`:
   ```csharp
   Console.WriteLine("4) Publish YourNewEvent");
   ```

2. **Add Event Publishing Logic**:
   ```csharp
   else if (input == "4")
   {
       // Collect input
       // Create event instance
       // Publish event
   }
   ```

3. **Rebuild and Test**:
   ```powershell
   dotnet build
   dotnet run
   ```

### Testing Workflow

1. **Start RabbitMQ** (if not already running)
2. **Start Notifications Microservice** (to consume events)
3. **Run TestPublisher**:
   ```powershell
   dotnet run
   ```
4. **Publish Test Events** using the interactive menu
5. **Verify Events** are consumed in the Notifications service logs
6. **Check Results** in the database or application UI

---

## 📝 Example Workflow

### Complete Testing Scenario

1. **Start Services**:
   ```powershell
   # Terminal 1: Start RabbitMQ (Docker)
   docker start rabbitmq

   # Terminal 2: Start Notifications Service
   cd OnlineCourseSystem.Notifications
   dotnet run

   # Terminal 3: Start Test Publisher
   cd OnlineCourseSystem.TestPublisher
   dotnet run
   ```

2. **Publish Test Event**:
   ```
   Choose: 1
   Enter UserId (GUID): c8c49232-4c8a-4f8c-8de1-4a70c7ad1140
   NotificationRequestedEvent published!
   ```

3. **Verify in Notifications Service**:
   - Check console logs for consumer activity
   - Verify notification created in database
   - Check email/push outbox if applicable

4. **Check RabbitMQ Management**:
   - Navigate to: `http://localhost:15672`
   - View queues and message flow
   - Monitor consumer activity

---

## 🔐 Security Notes

⚠️ **Important**: This tool is intended for **development and testing purposes only**.

- Do not use in production environments
- Do not commit production credentials to configuration files
- Use environment variables for sensitive configuration
- Ensure RabbitMQ is properly secured in production

---

## 📄 License

This project is part of the Codeway Online Courses Platform. License details will be finalized before public release.

---

## 🤝 Contributing

To improve this testing tool:

1. Add support for additional event types
2. Implement batch event publishing
3. Add event replay capabilities
4. Create event templates for common scenarios
5. Add logging and event history tracking

---

**Happy Testing! 🚀**

For issues or questions, please refer to the main project documentation or open an issue in the repository.
