# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [√] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [√] Commit: `Create Subscriber model struct.`
    -   [√] Commit: `Create Notification model struct.`
    -   [√] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [√] Commit: `Implement add function in Subscriber repository.`
    -   [√] Commit: `Implement list_all function in Subscriber repository.`
    -   [√] Commit: `Implement delete function in Subscriber repository.`
    -   [√] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [√] Commit: `Create Notification service struct skeleton.`
    -   [√] Commit: `Implement subscribe function in Notification service.`
    -   [√] Commit: `Implement subscribe function in Notification controller.`
    -   [√] Commit: `Implement unsubscribe function in Notification service.`
    -   [√] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [√] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [√] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [√] Commit: `Implement publish function in Program service and Program controller.`
    -   [√] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [√] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
Based on my analysis of the BambangShop codebase, here are concise answers to your reflection questions:
1. Observer Pattern Interface vs Single Model Struct:
In this BambangShop case, a single Model struct is sufficient. The Subscriber struct already fulfills the observer role without needing an interface/trait. In Rust, the implementation is pragmatic - the observer functionality is achieved through the Subscriber model directly, with notification handling embedded in its methods rather than through an interface abstraction.
2. Vec vs DashMap for Unique IDs:
DashMap is necessary here. While Vec could store unique elements, it would require O(n) lookups and additional logic to ensure uniqueness. DashMap provides O(1) lookups, built-in uniqueness by key, and thread-safety. The code uses nested DashMaps for efficient product_type → url → Subscriber mapping, which would be cumbersome with Vec.
3. DashMap vs Singleton Pattern:
Both are actually being used together. The lazy_static! { static ref SUBSCRIBERS } is implementing the Singleton pattern, while DashMap provides thread-safety. You need both - the Singleton ensures a single global instance, while DashMap handles concurrent access safely. Replacing DashMap would require implementing your own thread-safe collection, which would be reinventing what DashMap already provides efficiently.

#### Reflection Publisher-2
Based on my experience implementing BambangShop's services and controllers, here are my reflections:
1. Separating Service and Repository from Model:
The separation follows the Single Responsibility Principle. Models should only represent data structures, while Services handle business logic and Repositories manage data persistence. This separation makes the code more maintainable - we can modify business rules in Services without touching data storage logic in Repositories or data structures in Models. For example, in BambangShop, the NotificationService handles subscription logic while SubscriberRepository handles the actual storage, keeping concerns cleanly separated.

2. Using Only Models:
Without separation, each Model would need to handle data structure, business logic, and storage. For example, the Subscriber model would need subscription logic, notification sending, and storage management. This creates tight coupling - changes to notification logic could affect storage code. The Program model would need to know about Subscriber internals to notify them, violating encapsulation. The complexity grows exponentially as models interact more, making the code harder to maintain and test.

3. Postman Experience:
Postman has been invaluable for testing BambangShop's endpoints. Key helpful features:
- Collections: Organizing related requests (subscribe/unsubscribe) together
- Environment variables: Storing the base URL to easily switch between local/deployed
- Request history: Tracking failed attempts while debugging
- Response validation: Verifying correct status codes and JSON responses
- Request chaining: Testing subscriber notification flow end-to-end

#### Reflection Publisher-3
Based on my implementation experience with BambangShop's notification system, here are my reflections:

1. Push vs Pull Model:
BambangShop uses the Push model variation of the Observer Pattern. When a product is created/deleted/published, the NotificationService actively pushes the notification data to all subscribed URLs via HTTP POST requests. Subscribers don't need to periodically check for updates - they just need to have an endpoint ready to receive notifications.

2. Using Pull Model Instead:
Advantages of Pull:
- Less network overhead if changes are infrequent since subscribers only pull when needed
- More resilient to subscriber downtime since they control when to get updates
- Simpler error handling since failed pulls can be retried by subscribers

Disadvantages of Pull:
- Higher latency since subscribers need to poll periodically
- More complex subscriber implementation to handle polling logic
- Increased load on BambangShop server from constant polling
- Wasted requests when there are no updates to pull

For BambangShop's use case, Push is better since product updates need to be delivered promptly and the subscriber count is likely manageable.

3. Impact Without Multi-threading:
Without multi-threading in the notification process:
- Notifications would be sent sequentially to each subscriber
- Each HTTP request would block until completed
- A slow/unresponsive subscriber would delay notifications to all other subscribers
- The product API endpoint would be blocked until all notifications complete
- Overall system responsiveness would degrade significantly with more subscribers

This is why BambangShop uses async HTTP requests to notify subscribers concurrently, preventing any single slow subscriber from becoming a bottleneck.
