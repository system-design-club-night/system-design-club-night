```
flowchart TD
    Client[Client] -->|POST /subscription| APIGateway[API Gateway + Auth]
    APIGateway --> Subscription[Subscription Service]
    Subscription -->|Validate Payment Info| PaymentGateway[Payment Gateway]
    PaymentGateway -->|Return Token| Subscription
    Subscription -->|Save Subscription Info| SubscriptionDB[Subscription DB]
    Subscription -->|Save History| UserSubscriptionHistoryDB[User Subscription History DB]
    Subscription -->|Return Response| APIGateway
    APIGateway --> Client

    Scheduler[Scheduler Service] -->|Daily Check| SubscriptionDB_Scheduler[Subscription DB]
    SubscriptionDB_Scheduler -->|Return Payment List| Scheduler
    Scheduler -->|Process Payment| PaymentGateway_Scheduler[Payment Gateway]
    PaymentGateway_Scheduler -->|Return Payment Result| Scheduler
    Scheduler -->|Notify Result| Notification[Notification Service]
    Notification --> Client
    Scheduler -->|Update History| UserSubscriptionHistoryDB_Scheduler[User Subscription History DB]

```
