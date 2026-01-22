# OnlineCourseSystem.Contracts.Messaging

![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=flat-square&logo=dotnet&logoColor=white)
![C#](https://img.shields.io/badge/C%23-12.0-239120?style=flat-square&logo=c-sharp&logoColor=white)
![Status](https://img.shields.io/badge/Status-Shared%20Library-blue?style=flat-square)

Shared event contracts library for the Codeway Online Courses Platform. Defines canonical integration events used across microservices for event-driven communication via RabbitMQ/MassTransit.

---

## 🎯 Overview

This library provides the **single source of truth** for integration event contracts used in the event-driven architecture. All microservices reference this shared library to ensure type-safe, version-controlled event schemas between publishers and consumers.

**Key Benefits:**
- **Type Safety**: Compile-time validation of event contracts
- **Version Control**: Centralized contract management
- **Compatibility**: Ensures publisher/consumer compatibility
- **Documentation**: Self-documenting event structure

---

## 📦 Contents

### IntegrationEvent (Base Class)

Abstract base record providing common event metadata:

```csharp
public abstract record IntegrationEvent
{
    public Guid EventId { get; init; }              // Unique event identifier
    public DateTime OccurredAt { get; init; }       // UTC timestamp
    public Guid? CorrelationId { get; init; }       // Cross-service correlation
    public string? TraceId { get; init; }           // Distributed tracing ID
}
```

### Event Types

#### NotificationRequestedEvent
Generic notification request that any service can publish to trigger notifications.

**Properties:**
- `UserIds` (required): Collection of user GUIDs to notify
- `NotificationType` (required): "System", "Course", "Reminder", "Announcement"
- `Title` (required): Notification title
- `Content` (required): Notification content
- `CourseId` (optional): Related course identifier

**Usage:** Published by any service to request notifications via the Notifications microservice.

#### UserEnrolledEvent
Raised when a user successfully enrolls in a course.

**Properties:**
- `UserId` (required): User who enrolled
- `CourseId` (required): Course identifier
- `CourseName` (optional): Course name for descriptive purposes

**Usage:** Published by Course/Auth services, consumed by Notifications service to create welcome notifications.

#### PaymentSucceededEvent
Raised when a payment transaction succeeds.

**Properties:**
- `UserId` (required): User who made payment
- `Amount` (required): Payment amount in minor units
- `Currency` (optional): ISO 4217 currency code (e.g., "EGP", "USD")
- `Description` (optional): Payment description
- `CourseId` (optional): Related course if applicable

**Usage:** Published by Payment service, consumed by Notifications service to create payment confirmations.

---

## 🚀 Usage

### Adding Reference

**Project Reference** (same solution):
```xml
<ProjectReference Include="..\OnlineCourseSystem.Contracts.Messaging\OnlineCourseSystem.Contracts.Messaging.csproj" />
```

**NuGet Package** (separate repos):
```xml
<PackageReference Include="OnlineCourseSystem.Contracts.Messaging" Version="1.0.0" />
```

### Publishing Events

```csharp
using OnlineCourseSystem.Contracts.Messaging.Events;
using MassTransit;

public class CourseService
{
    private readonly IPublishEndpoint _publishEndpoint;

    public async Task EnrollUserAsync(Guid userId, Guid courseId, string courseName)
    {
        // Business logic...
        
        // Publish event
        var ev = new UserEnrolledEvent
        {
            UserId = userId,
            CourseId = courseId,
            CourseName = courseName,
            CorrelationId = Guid.NewGuid(),
            TraceId = Activity.Current?.Id
        };

        await _publishEndpoint.Publish(ev);
    }
}
```

### Consuming Events

```csharp
using MassTransit;
using OnlineCourseSystem.Contracts.Messaging.Events;

public class NotificationRequestedConsumer : IConsumer<NotificationRequestedEvent>
{
    public async Task Consume(ConsumeContext<NotificationRequestedEvent> context)
    {
        var message = context.Message;
        
        // Access event properties
        var userIds = message.UserIds;
        var notificationType = message.NotificationType;
        
        // Process event...
        
        // Access correlation/trace IDs for logging
        var correlationId = context.CorrelationId ?? message.CorrelationId;
        var traceId = message.TraceId;
    }
}
```

### Notification Request Example

```csharp
var notificationEvent = new NotificationRequestedEvent
{
    UserIds = new[] { userId1, userId2 },
    NotificationType = "Course",
    Title = "New lesson available!",
    Content = "Lesson 5 is now available in your course.",
    CourseId = courseId,
    CorrelationId = Guid.NewGuid(),
    TraceId = Activity.Current?.Id
};

await _publishEndpoint.Publish(notificationEvent);
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────┐
│  Publisher Services                    │
│  (Auth, Course, Payment, etc.)          │
└──────────────┬──────────────────────────┘
               │
               │ Uses Contracts.Messaging
               │
┌──────────────▼──────────────────────────┐
│  OnlineCourseSystem.Contracts.Messaging │
│  • IntegrationEvent (base)              │
│  • NotificationRequestedEvent          │
│  • UserEnrolledEvent                   │
│  • PaymentSucceededEvent              │
└──────────────┬──────────────────────────┘
               │
               │ RabbitMQ / MassTransit
               │
┌──────────────▼──────────────────────────┐
│  Consumer Services                       │
│  (Notifications, Analytics, etc.)       │
└─────────────────────────────────────────┘
```

---

## 📋 Event Contract Guidelines

### Adding New Events

1. **Create Event Record**
   ```csharp
   public sealed record YourNewEvent : IntegrationEvent
   {
       public required Guid YourProperty { get; init; }
       public string? OptionalProperty { get; init; }
   }
   ```

2. **Follow Naming Conventions**
   - Event names end with `Event`
   - Use `required` for mandatory properties
   - Use nullable types for optional properties
   - Use `init` accessors (immutable records)

3. **Documentation**
   - Add XML comments explaining the event purpose
   - Document when/where the event is published
   - Document which services consume it

4. **Version Coordination**
   - Increment package version on breaking changes
   - Coordinate updates across all consuming services
   - Consider backward compatibility

### Best Practices

- **Immutable Records**: All events are `record` types with `init` properties
- **Required vs Optional**: Use `required` for essential data, nullable for optional
- **Correlation Tracking**: Always set `CorrelationId` and `TraceId` when available
- **Type Safety**: Use strong types (Guid, DateTime) instead of strings where possible
- **Backward Compatibility**: Add new properties as optional to avoid breaking changes

---

## 🔄 Version Management

### Breaking Changes

When making breaking changes:
1. Increment major version
2. Notify all consuming services
3. Coordinate deployment across services
4. Update documentation

### Non-Breaking Changes

- Adding new optional properties
- Adding new event types
- Increment minor/patch version

---

## 🧪 Testing

Events can be tested using the `OnlineCourseSystem.TestPublisher` utility:

```powershell
cd OnlineCourseSystem.TestPublisher
dotnet run
# Select event type to publish
```

---

## 📁 Project Structure

```
OnlineCourseSystem.Contracts.Messaging/
├── Event.cs                    # All event definitions
└── OnlineCourseSystem.Contracts.Messaging.csproj
```

**Current Events:**
- `IntegrationEvent` (base class)
- `NotificationRequestedEvent`
- `UserEnrolledEvent`
- `PaymentSucceededEvent`

---

## 🔗 Related Projects

- **OnlineCourseSystem.Notifications**: Consumes all events, creates notifications
- **OnlineCourseSystem.TestPublisher**: Testing utility for publishing events
- **OnlineCourseSystem.Auth**: Publishes UserEnrolledEvent (future)

---

## ⚠️ Important Notes

- **Single Source of Truth**: All event contracts must be defined here
- **No Business Logic**: This library contains only data contracts
- **Version Coordination**: Coordinate version updates across all services
- **Breaking Changes**: Treat contract changes like API changes—coordinate carefully

---

## 🤝 Contributing

When adding new events:

1. Follow the existing event structure pattern
2. Add comprehensive XML documentation
3. Update this README with event details
4. Coordinate with teams that will consume the event
5. Test event serialization/deserialization

---

**Shared Contracts for Event-Driven Architecture** 🚀
