### Food Order Microservices App - POC

Business Requirements Specification (BRS):

Goals:

Build a Food Order Microservices system using Reactive, Cloud-Native & Event-driven Microservices Architecture with the following Business requirements:


Title: Customer Management

Description: Customer Service should manage customer details, including Personal data, Contact details, Address, and Wallet balance, enabling seamless integration with the Order and Payment Services.
Business Rules:

Customer data (e.g., Name, Address, Contact, Wallet balance) should be imported from a JSON file containing 1 million records.
Delta imports must be enabled to allow incremental updates of customer records.
Wallet balances can be added or updated using bulk or delta imports.

Acceptance Criteria:

Given: A JSON file with customer data is provided.
Pre-Condition: Customer Service DB should be able to process large volumes of Data (~ 1 million records).
When: A bulk or Delta import is enabled/triggered for every new Customer added.
Then: All Customer's Details are imported/updated successfully, including wallet balances.
Title: Order Placement and Validation of Customer Details

Description: Customers should be able to browse products, add items to a cart, and place an order through the Order Service.
Business Rules:

Validate that the cart is not empty before order placement.
Ensure that the required Customer Details (address, contact, wallet balance) are available before proceeding.
Places the Order after all the Customer Details are validated via Order Service.

Acceptance Criteria:

Given: A customer browses products and adds them to the cart.
Pre-Condition: Customer details and Food Cart are available.
When: The customer places an order.
Then: The order is placed successfully, and an initiation event is published to Kafka.
Title: Payment Processing via Wallet or Payment Interface

Description: Customers should be able to pay using their wallet balance or a Payment Interface (internal) if the wallet balance is insufficient.

Business Rules:

Payment Service validates the Wallet Balance via Order Service i.e., through the Customer Service.
If sufficient, the Wallet is debited without invoking the Payment Interface.
If insufficient, the Payment Interface processes the payment.
In case of payment failure, the order is marked as Failed.

Acceptance Criteria:

Given: A customer places an order requiring Payment.
Pre-Condition: Wallet and Payment Interface are operational.
When: Payment Service processes the payment.
Then:
If the Wallet Balance is sufficient, payment succeeds without invoking the Payment Interface.
If insufficient, the Payment Interface is invoked, to process the Payment and, Payment status is updated accordingly as, either Success or Failure.
If Payment is Successful, then the Order Service is notified to proceed further.
Order Service then, initiates Inventory Availability check with Restaurant Service.
If Payment Fails, an event is published, and the Order is marked as Failed.
Title: Inventory Availability Check, Vendor Selection and, Order Confirmation by Restaurant Service

Description: Restaurant Service initiates Inventory Availability check in parallel with multiple vendors. The best vendor is selected by Order Service based on price and delivery time.
Business Rules:

Order Service initiates Inventory Availability check by communicating with Restaurant Service.
Restaurant Service initiates Inventory Availability check in parallel with at least 2 vendors, and further notifies Order Service based on the responses.
If Inventory with both Vendors is available then,
Vendor with the lowest price and quickest delivery options is selected by Order Service
Restaurant Service confirms/accepts the Order.
If no vendor has inventory availability, the Order is marked as Failed, and Order Status is updated/notified accordingly via Email/SMS notification to Customer.
Also, the Order Service initiates Payment Refund process for that Customer's Order Failure, by interacting with Payment Service.
Payment Service then, processes the Refund for Customer's Order amount depending upon Payment source i.e., either via Customer's Wallet update or Payment Interface.

Acceptance Criteria:

Given: An Order is placed, and Payment is confirmed/completed.
Pre-Condition: Vendor inventory availability check operation is possible by Restaurant service.
When: Order Service checks Inventory Availability with both Vendors via Restaurant Service communication.
Then:
If Inventory is available with both Vendors, the most suitable Vendor option is selected by Order Service, and the Order is confirmed/accepted by Restaurant Service.
If no inventory is available, the order is rejected or Failed, Order Failure status is notified via Email/SMS notification to Customer and, 
Payment Refund is initiated & processed via Payment Service for that Order Failure.
Title: Notification of Order Status

Description: Customers should be notified of their order status via Email or SMS.
Business Rules:

Notification Service listens for order status events on Kafka.
Sends notifications based on the order status to Notification System(external), which sends Order Status as Email/SMS notification to Customer.
If Notification Service fails, fallback notifications are issued.

Acceptance Criteria:

Given: Order Status updates are received from Order Service.
Pre-Condition: Customer's contact details(i.e., Email Address and Contact no.) are available from Order Service.
When: Notification Service processes an event to trigger Notification System(external).
Then: Notifications are sent successfully over the Email/SMS, or fallback responses are issued to Customer.
Error Scenarios:
Payment Failure:
Description: If the wallet balance is insufficient and the Payment Interface fails, the Order should be marked as Failed.
Acceptance Criteria:
Order Service receives a Payment Failure event and the Order is rejected or Failed.
Inventory Unavailability:
Description: If all available vendors have Zero/Nil availability, the Order should be rejected or failed and, Payment Refund should be initiated for Customer.
Acceptance Criteria:
Order Service receives an Inventory Unavailable event and marks the order as Failed thereby, initiating full Payment refund for the Customer.
Notification Failure:
Description: If the Notification Service fails, fallback notifications should be sent.
Acceptance Criteria:
A fallback response with a default message is issued.
Technical Errors (Service Down/Slow Responses):
Description: Services should implement resiliency patterns such as retries, circuit breakers, and fallbacks.
Acceptance Criteria:
If a service is unavailable, retries are attempted.
If retries fail, fallback mechanisms are triggered.
Technical Requirements and Notes:
General Overview:

The Food Order Microservices Application will utilize a Reactive, Event-driven, and Cloud-Native architecture to meet business requirements while adhering to API-First, TDD, and Clean Code principles. It will leverage Google Cloud Services for deployment and observability, Confluent Kafka for event-driven communication, and MongoDB and KSQL DB for data persistence.

1. Architecture Overview:
Microservices Design:
Customer Service: Manages customer details, wallet balances, and bulk/delta imports.
Order Service: Acts as the SAGA orchestrator, handling order lifecycle and integration with other services.
Payment Service: Processes payments via wallet or Payment Interface.
Restaurant Service: Handles inventory checks, vendor selection, and order confirmations.
Notification Service: Sends order status notifications via Email/SMS.
Reactive Programming:
Use Spring Web Flux and Project Reactor to build non-blocking, asynchronous APIs.
Event-Driven Communication:
Implemented via Confluent Kafka, ensuring decoupled communication between microservices.
Kafka topics for publishing and subscribing to events, e.g., order.create, payment.processed, inventory.available.
Database Design:
MongoDB (Customer Service): Stores customer details and wallet balances.
KSQL DB (Transactional and Time-Series DB): Used for Order, Payment, Restaurant, and Notification Services.


Resiliency and Fault Tolerance:
Integrate Resilience4j for retries, circuit breakers, and fallbacks.
Implement Dead Letter Queues (DLQ) for failed Kafka messages.


DevOps and Deployment:
GitLab CI/CD for automated DevOps pipelines.
Google Kubernetes Engine (GKE) for container orchestration & deployment.
Google Cloud Monitoring for observability and distributed tracing.


2. Technical Requirements for Business Requirements Specifications:
Customer Management:
MongoDB Operations:
Enable sharding to handle ~1 million customer records.
Support for Bulk Import:
Process customer details from a JSON file and write to MongoDB.
Support for Delta Import:
Incrementally update customer details and wallet balances.
REST API:
Provide endpoints for fetching and updating customer details.
Performance Requirements:
Optimize queries for high-volume reads and writes.
Ensure sub-second latency for wallet balance validation.
Order Placement and Validation:
Order Service as SAGA Orchestrator:
Use synchronous REST communication with Customer Service to validate customer details.
Publish order.create, payment.initiate events to initiate the event-driven communication.
Validation Rules:
Cart must not be empty.
Customer details (address, contact, wallet balance) must be validated before order placement.
Scalability:
Support concurrent order placement with eventual consistency guarantees.
KSQL DB Operations:
Store order status updates (e.g., Created, Confirmed, Failed).
Payment Processing:
Payment Service:
Validate Wallet Balance via Order Service as SAGA Orchestrator, through REST API communication with Customer Service.
Handle payment via:
Wallet: Update(Debit) Wallet Balance for Payment, if sufficient.
Payment Interface: Process the payment through Payment Interface, if Wallet Balance is insufficient.
Kafka Topics:
payment.initiate: Subscribe/consume to initiate Payment process after Order is placed.
wallet.balance.validate: Publish to Validate Wallet Balance of Customer before Processing Payment.
wallet.balance.update: Publish to update the Wallet Balance, as per following scenarios:
Payment Debit if it's sufficient or,
Refund credit processed for Inventory Unavailability related Order Failure.
payment.processed: Publish based on the Payment Process outcome i.e. Success or Failure.
payment.refund: Published to initiate Refund of full Payment amount to Customer in case of Inventory Unavailable based Order Failure.
Resiliency:
Retries for wallet balance validation.
Circuit breaker for Payment Interface failures.
KSQL DB Operations:
Record transaction details and payment status.
Inventory Availability and Vendor Selection:
Restaurant Service:
Subscribes/consumes order.create, payment.processed and inventory.check events, once Order creation and Payment is successful.
Performs inventory availability checks in-parallel with at least two Vendors.
Notify Order Service of inventory availability status.
Order Service:
Select the most suitable vendor based on price and delivery time.
Publish order.vendor.select event for Restaurant Service to accept/confirm the Order and notify back by publishing order.confirm event for Order Service.
Error Handling:
If no inventory is available, mark the order as Failed.
Publish payment.refund event to initiate & complete the Payment Refund to Customer via Payment Service.
Publish order.confirm.notify (with Order Failure status) for Notification Service to notify Customer via Email/SMS and,
KSQL DB Operations:
Record inventory check responses and vendor selection details.
Order Status Notifications:
Notification Service:
Consume order.confirm.notify event from Order Service .
Send Email/SMS notifications via an external Notification System.
Publish notification.processed event to Order Service.
Fallback Mechanism:
If Notification Service fails, issue fallback notifications.
KSQL DB Operations:
Log notification delivery details for traceability.
Resiliency and Observability:
Resilience4j:
Retries for transient failures (e.g., Kafka publishing).
Circuit breakers for dependent service failures.
Google Cloud Observability:
Enable tracing and logging for all services.
Monitor Kafka topic metrics and DLQ utilization.
Performance and Scalability:
Scalability Requirements:
Autoscaling services in GKE to handle traffic spikes.
Use MongoDB sharding and Kafka partitioning to optimize throughput.
Non-Functional Metrics:
Throughput: Process up to 1000 orders per second.
Latency: Ensure sub-second response times for APIs.
3. Dev Tools and Tech Stack:
Category	Tools/Frameworks
Language	Java 17
Framework	Spring Boot 3.x, Spring WebFlux
Reactive Programming	Project Reactor
Event-driven Messaging	Confluent Kafka, Spring Kafka
Databases	MongoDB, KSQL DB
Resiliency	Resilience4j
API Development	OpenAPI/Swagger, SpringDoc
Observability	Google Cloud Monitoring, Micrometer, OpenTelemetry
Containerization	Docker
Orchestration	Kubernetes (Google Kubernetes Engine)
CI/CD	GitLab CI/CD
Security	Spring Security, OAuth2, Google Identity
4. Key Non-Functional Requirements (NFRs):
NFR	Details
Performance	Sub-second API latency; 1000 orders/second.
Scalability	Autoscaling services in GKE.
Fault Tolerance	Resilience4j Circuit Breakers, Retry Mechanism
Security	OAuth2, HTTPS for secure communication.
Observability	Distributed tracing, centralized logging.




System Design Diagrams:
High-Level System Design (System Context Diagram):



The High-Level System Design (System Context Diagram) for this Food Order Microservices system includes the following major components:

System Components:
1. Customer (Browser/App):
Represents the Customer or end-user who interacts with the Food Order Microservices system.
Capabilities:
Select and Add the products/items to the cart.
Place Food Order request.
Receives Order Status as response.




2. Microservices:
1. Customer Service:
Purpose:
Manages customer data and wallet balances.
Exposes APIs for wallet validation during payment processing.
Database: MongoDB (NoSQL) for high performance and scalability.
Database Schema:
customer_id (String): Unique identifier for the customer.
name (String): Customer name.
email (String): Email address of the customer.
wallet_balance (Decimal): Current wallet balance of the customer.
address (String): Customer delivery address.
2. Order Service:
Purpose:
Acts as SAGA Orchestrator for the Food Order Microservices System lifecycle.
Handles Synchronous or REST based communication with Customer Services
Handles asynchronous & event-driven communication with Payment, Restaurant and Notification Microservices resp., using SAGA Orchestrator Microservices pattern.
Publishes and subscribes to Kafka events for event-driven processing.
Database: KSQL DB (Transactional Database) for real-time query and event processing.
Database Schema:
order_id (String): Unique identifier for the order.
customer_id (String): Foreign key linking to the customer.
order_status (Enum): Status of the order (e.g., Created, Confirmed, Delivered).
total_amount (Decimal): Total amount for the order.
Kafka Topics:
order.create: Published when a new Oder is placed.
payment.initiate: Published for Payment Service to initiate Payment process.
wallet.balance.validate: Subscribe/Consume to Validate Wallet Balance of Customer before Processing Payment.
wallet.balance.update: Subscribe/Consume to update the Wallet Balance, as per following scenarios:
Payment Debit if it's sufficient or,
Refund credit processed for Inventory Unavailability related Order Failure.
payment.processed: Subscribed/Consumed after payment processed via Wallet Balance (debit) or Payment Interface i.e., either Success or Failure status. 
refund.initiate: Published for Payment Service to initiate Refund for each Customer in case of Order Failure as a result of Inventory Unavailable.
refund.processed: Subscribed when Refund is processed and completed to Customer for Order Failure as a result of Inventory Unavailable.
inventory.check: Published to initiate Restaurant Service for Inventory availability check with both Vendors.
inventory.available: Subscribed/Consumed to Order Service for inventory availability status check with a vendor.
order.vendor.select: Published when a Vendor is selected by Order service for Restaurant Service.
order.confirm: Subscribed/Consumed when Order is accepted by Restaurant Service after successful payment and vendor selection.
order.confirm.notify: Published for Order Confirmation status to Notification Service.
notification.processed: Subscribed/Consumed for notification processing(send/delivery) event i.e., executed via REST communication with external Notification(Email/SMS) System.
3. Payment Service:
Purpose:
Processes payments for orders.
Supports wallet-based payments or external payment gateways (internal component).
Publishes and Subscribes for payment processing status & payment initiation to Kafka events.
Database: KSQL DB (Transactional Database).
Database Schema:
payment_id (String): Unique identifier for the payment.
order_id (String): Foreign key linking to the order.
amount (Decimal): Amount processed for the payment.
status (Enum): Payment status (Success, Failed).
Kafka Topics:
payment.initiate: Subscribed/Consumed for payment process initiation event after Order is placed from Order Service.
wallet.balance.validate: Publish to Validate Wallet Balance of Customer before Processing Payment.
wallet.balance.update: Publish to update the Wallet Balance, as per following scenarios:
Payment Debit if it's sufficient or,
Refund credit processed for Inventory Unavailability related Order Failure.
payment.processed: Published when payment processed via Wallet Balance (debit) or Payment Interface i.e., either Success or Failure status.
refund.initiate: Subscribed/consumed for initiation Refund to each Customer in case of Order Failure as a result of Inventory Unavailable.
refund.processed: Published when Refund is processed and completed to Customer for Order Failure as a result of Inventory Unavailable.
4. Restaurant Service:
Purpose:
Checks inventory availability with vendors.
Confirms and accepts orders based on vendor selection.
Publishes and Subscribes to Kafka events for inventory availability check and vendor selection.
Database: KSQL DB (Transactional Database).
Database Schema:
restaurant_id (String): Unique identifier for the restaurant.
inventory_id (String): Inventory identifier.
order_id (String): Foreign key linking to the order.
availability_status (Boolean): Indicates whether the inventory is available.
price (Decimal): Price offered by the vendor.
Kafka Topics:
inventory.check: Subscribed/Consumed for Inventory availability check with both Vendors. 
inventory.available: Published to Order Service for inventory availability status check with a vendor.
order.vendor.select: Subscribed/Consumed for Vendor Selection confirmed from Order service.
order.confirm: Published to Order Service when Order is accepted/confirmed.
5. Notification Service:
Purpose:
Sends Email and SMS notifications to Customers based on Order Status events received via Notification System (External).
Publishes and subscribes to Kafka events for notification delivery status.
Database: Time-series DB (KSQL from Confluent Ecosystem).
Database Schema:
notification_id (String): Unique identifier for the notification.
order_id (String): Foreign key linking to the order.
message (String): Notification message content.
timestamp (DateTime): Time of notification creation.
Kafka Topics:
order.confirm.notify: Subscribed/Consumed for Notification Service after Order confirmation.  
notification.processed: Published for notification processing(send/delivery) event i.e., executed via REST communication with external Notification(Email/SMS) System.
3. Event Backbone:
Confluent Kafka
Kafka acts as the event backbone, enabling asynchronous communication between microservices.
Kafka Topics Overview:
order.create: Publish-Subscribe/Consume as an initiation event when a new Oder is placed.
payment.initiate: Publish-Subscribe/Consume for payment process initiation event after Order is placed from Order Service. 
wallet.balance.validate: Publish-Subscribe/Consume to Validate Wallet Balance of Customer before Processing Payment.
wallet.balance.update: Publish-Subscribe/Consume to update Wallet Balance, as per following scenarios:
Payment Debit if it's sufficient or,
Refund credit processed for Inventory Unavailability related Order Failure.
payment.processed: Publish-Subscribe/Consume when payment processed via Wallet Balance (debit) or Payment Interface i.e., either Success or Failure status.
refund.initiate: Publish-Subscribe/Consume to initiate and process complete Refund for each Customer in case of Order Failure as a result of Inventory Unavailable.
refund.processed: Subscribed when Refund is processed and completed to the Customer for Order Failure as a result of Inventory Unavailable.
inventory.check: Publish-Subscribe/Consume to Check Vendor Availability.
inventory.available: Publish-Subscribe/Consume for checking the Inventory availability with both the Vendors of the Restaurant.
order.vendor.select: Publish-Subscribe/Consume when a Vendor option serving with lowest price and quickest delivery is selected.
order.confirm: Publish-Subscribe/Consume when Order is confirmed/accepted by the Restaurant after successful payment and vendor selection.
order.confirm.notify: Publish-Subscribe/Consume for Order Confirmation Status (i.e., Success or Failure) notification.
notification.processed: Publish-Subscribe/Consume for notification processing(send/delivery) event i.e., executed via REST communication with external Notification(Email/SMS) System.
4. External Services:
Notification (Email/SMS) System:
Purpose:
Sends Email and SMS notifications for Customer's Order Status to API Gateway.
Recommended OSS Tools:
Email: SendGrid
SMS: Twilio
5. GCP Components:
Google Cloud API Gateway:
Purpose: 
Single entry point for all Customer(Browser/App) requests.
Route the Customer's Food Order requests to Order Microservice.
Perform Load Balancing, Security posture(authentication, authorization), Response Aggregator.
Responds the Order Status received from Notification(Email/SMS) System to Customer.
GKE (Google Kubernetes Engine)
Purpose:
Hosts all microservices in containerized environments.
Ensures scalability, resilience, and fault tolerance.
Google Cloud Logging & Monitoring
Purpose:
Provides observability into system performance.
Monitors logs, metrics, and alerts.
6. Legends:
Color	Component
Blue	

Microservices


Green	Databases, REST API calls & DB operations
Yellow	Confluent Kafka & Event Topics
Orange	External Services
Light Purple	

GCP Components

Sequence Diagrams:

Sequence Diagrams for all the possible scenarios with respect to Food Order Microservices System operational results, as follows:

1. Positive/Success Flow (Happy Path):

Happy Path Scenario:

Flow Description
Customer places a Food Order request via the browser/app.
API Gateway routes the Order request to Order Service.
The Order Service (SAGA Orchestrator):
Reads and Validates Customer Details (i.e., Profile, Address, Contact and Wallet Balance details) via REST API/Synchronous communication with Customer Service.
Publishes order.create, payment.initiate event notifications to initiate Event-driven communications as a SAGA Orchestrator further.
Payment Service:
Subscribes & consumes the payment.initiate event to initiate Payment process.
Publishes wallet.balance.validate event to validate Wallet Balance before Payment processing. 
Order Service:
Subscribes & consumes wallet.balance.validate event. 
Validates Wallet Balance through REST API communication with Customer Service.
If Wallet Balance is sufficient, publishes wallet.balance.update event.
Payment Service:
Subscribes & consumes  wallet.balance.update event and, processes the Payment via updating(debit) Wallet Balance, if sufficient.
Otherwise, processes the payment through the Payment Interface(Internal).
Publishes payment.processed(Success)event notification.
Order service:
Subscribes and consumes payment.processed(Success) event and, publishes inventory.check event for further communication with Restaurant Service. 
Restaurant Service:
Checks inventory availability with both Vendors after subscribing and consuming order.create, payment.processed(Success),inventory.check events.
Publishes inventory.available(Success) event for Order Service, once it receives API response for Inventory Availability with both Vendors.
Order Service:
Subscribes and consumes inventory.available(Success) event from Restaurant Service and then,
Selects the Vendor option with lowest price and fastest delivery, and further publishes order.vendor.select event for Restaurant Service.
Restaurant Service:
Subscribes and consumes order.vendor.select event notification with selected Vendor details from Order Service and,
Confirms/accepts the Food Order for fulfilment. It further publishes order.confirm(Success/Accepted) event.
Order Service:
Subscribes and consumes order.confirm (Success/Accepted) event and, hence performs the Order Confirmation.
Publishes order.confirm.notify (Success)event.
Notification Service:
Subscribes and consumes the order.confirm.notify (Success)event having the Order Confirmation (Success) and Customer details.
Sends an Order Confirmation notification to Notification System(External) via REST API communication.
Notification Service:
Sends the Email/SMS Notifications for Order Confirmation(Success) status to API Gateway. 
Publishes notification.processed(Success) event.
Order Service:

Subscribes and consumes notification.processed(Success) event.
Hence, the event-driven communication for Success/Positive or Happy Path scenario is completed there by, implementation of SAGA orchestration.
API Gateway:
 Sends Order Confirmation(Success) Status as the Food Order response back to the Customer.
Customer Service:

Receives Food Order Confirmation or completion status as a response to the initial Food Order request and hence, completes the Successful execution flow.
2. Negative/Failure Flows:




Payment Failure:


Failure Scenario Flow: Payment Failure:

Customer places and Order via Order Service.
Customer Service validates wallet balance, as requested by Payment Service.
The payment fails due to wallet balance insufficiency as well as Payment Interface issues.
The order is canceled, and the customer is notified with Payment Failure Notification via Email/SMS.

2. Inventory Unavailable Failure:

Failure Scenario: Inventory Unavailable:

Customer places an order via Order Service.
Customer Service validates wallet balance, as requested by Payment Service.
Payment Service processes payment successfully by either Wallet Ballance deduction, or through Payment Interface, if Wallet Balance is insufficient.
The Restaurant Service fails to confirm inventory availability i.e. Inventory is Unavailable with Vendors.
Order Service rejects the Order as Failed. It also initiates and processes Payment Refund to the Customer.
Customer is notified with Inventory Unavailable Failure Notification via Email/SMS.
3. Exception Flow (Technical Errors):

Scenario 1: Customer Service is Down or Slow:


Customer places and Order via Order Service.
Order Service requests Customer Service via REST API to Fetch/Read Customer Details.
If Customer Service is down. Circuit Breaker Opens Circuit to avoid Cascading failures and triggers Fallback notifications after retry timeout.
Order Service publishes Order Failure event via Kafka topic order.confirm.notify for Notification Service.
If Customer Service is responding Slow, then Retry Mechanism is activated and triggers Retry Notifications at specified intervals.

Scenario 2: Notification Service is Down or Slow:

Description:
Notification Service Down:
Circuit Breaker in the Notification Service opens to prevent further failures.
Order Service logs the failure and re-queues the notification event.
Notification Service Slow:
Retry mechanism attempts to resend the notification before logging the failure.
Tech Stack & Tools:
1. Core Development:

Programming Language:

Java 17: Enables modern language features like records, sealed classes, and pattern matching.

Frameworks:

Spring Boot 3.x: Microservice framework supporting reactive programming with Spring WebFlux.
Spring Cloud: For service discovery, config server, and distributed tracing.
Spring Data Reactive: Reactive access to MongoDB.
Spring Security: OAuth2/OpenID Connect for securing APIs.
Spring Kafka: Integration with Confluent Kafka for event-driven architecture.
Schema Registry: Ensure compatibility between producers and consumers.
Spring Validation: Input validation and constraints.

Reactive Framework:

Project Reactor: For building non-blocking, asynchronous systems.
Web-Client: For reactive HTTP communication.

Database:

Reactive MongoDB: NoSQL database used by Customer Service for scalability and reactive support.
Confluent KsqlDB: Transactional & Time-Series DB for Order Service, Payment Service, Restaurant Service & Notification Service.

Messaging/Event Streaming:

Confluent Kafka: Distributed event streaming platform.
2. Development Tools:

API Development:

Open API/Swagger Code Generator: API-First development thru Code Generator.
Postman/Insomnia: API testing.

IDE:

IntelliJ IDEA

Build Tools:

Maven

Version Control:

GitLab for version control.
GitLab CI for CI/CD pipelines.

Code Quality and Coverage:

SonarQube: Static code analysis for clean code.
JaCoCo: Code coverage reporting.

Testing:

JUnit 5: Unit testing.
Mockito: Mocking dependencies.
Testcontainers: Integration testing with ephemeral containers.
Postman: API testing.
3. Google Cloud Services:
API Gateway: Google Cloud Endpoints for API management.
Kubernetes: Google Kubernetes Engine (GKE) for container orchestration.
Cloud Pub/Sub: As a complementary messaging service (if Confluent Kafka not available).
Cloud Monitoring: Observability and logging with Google Operations Suite.
Cloud Secrets Manager: Secure management of credentials and keys.
4. DevOps Tools:

Containerization:

Docker: For containerized microservices.

CI/CD:

GitLab CI/CD: Pipeline for build, test, and deploy to GKE. (TBD)

Observability:

Prometheus + Grafana: Metrics collection and visualization.
Google Cloud Logging: Centralized logging.
Distributed Tracing: Open-Telemetry with GCP integrations.

### CI/CD with Auto DevOps

This template is compatible with [Auto DevOps](https://docs.gitlab.com/ee/topics/autodevops/).

If Auto DevOps is not already enabled for this project, you can [turn it on](https://docs.gitlab.com/ee/topics/autodevops/#enabling-auto-devops) in the project settings.
