********Food Order Microservices App - POC********

******1. Goals: Requirements Grooming******
  Build a Food Order Microservices system using Reactive, Cloud-Native & Event-driven Microservices Architecture with the following Business & Technical requirements:

 ****a. Business Requirements Specification (BRS):****

  **1. Title: Customer Management:**
  **Description:** Customer Service should manage customer details, including Personal data, Contact details, Address, and Wallet balance, enabling seamless integration with the Order and Payment Services.
  
  **Business Rules:**
     i) Customer data (e.g., Name, Address, Contact, Wallet balance) should be imported from a JSON file containing 1 million records.
     ii) Delta imports must be enabled to allow incremental updates of customer records.
     iii) Wallet balances can be added or updated using bulk or delta imports.
  
  **Acceptance Criteria:**
    **Given:** A JSON file with customer data is provided.
    **Pre-Condition:** Customer Service DB should be able to process large volumes of Data (~ 1 million records).
    **When:** A bulk or Delta import is enabled/triggered for every new Customer added.
    **Then:** All Customer's Details are imported/updated successfully, including wallet balances.
  
  
  **2. Title: Order Placement and Validation of Customer Details:**
  **Description:** Customers should be able to browse products, add items to a cart, and place an order through the Order Service.
  
  **Business Rules:**
    i) Validate that the cart is not empty before order placement.
    ii) Ensure that the required Customer Details (address, contact, wallet balance) are available before proceeding.
    iii) Places the Order after all the Customer Details are validated via Order Service.
  
  **Acceptance Criteria:**
    **Given:** A customer browses products and adds them to the cart.
    **Pre-Condition:** Customer details and Food Cart are available.
    **When:** The customer places an order.
    **Then:** The order is placed successfully, and an initiation event is published to Kafka.
  
  
  **3. Title: Payment Processing via Wallet or Payment Interface:**
  **Description:** Customers should be able to pay using their wallet balance or a Payment Interface (internal) if the wallet balance is insufficient.
  
  **Business Rules:**
    i) Payment Service validates the Wallet Balance via Order Service i.e., through the Customer Service.
    ii) If sufficient, the Wallet is debited without invoking the Payment Interface.
    iii) If insufficient, the Payment Interface processes the payment.
    iv) In case of payment failure, the order is marked as Failed.
  
  **Acceptance Criteria:**
    **Given:** A customer places an order requiring Payment.
    **Pre-Condition:** Wallet and Payment Interface are operational.
    **When:** Payment Service processes the payment.
    **Then:**
      a) If the Wallet Balance is sufficient, payment succeeds without invoking the Payment Interface.
      b) If insufficient, the Payment Interface is invoked, to process the Payment and, Payment status is updated accordingly as, either Success or Failure.
      c) If Payment is Successful, then the Order Service is notified to proceed further.
      d) Order Service then, initiates Inventory Availability check with Restaurant Service.
      e) If Payment Fails, an event is published, and the Order is marked as Failed.
  
  
  **4. Title: Inventory Availability Check, Vendor Selection and, Order Confirmation by Restaurant Service:**
  **Description:** Restaurant Service initiates Inventory Availability check in parallel with multiple vendors. The best vendor is selected by Order Service based on price and delivery time.
  
  **Business Rules:**
    i) Order Service initiates Inventory Availability check by communicating with Restaurant Service.
    ii) Restaurant Service initiates Inventory Availability check in parallel with at least 2 vendors, and further notifies Order Service based on the responses.
    iii) If Inventory with both Vendors is available then,
    iv) Vendor with the lowest price and quickest delivery options is selected by Order Service
    v) Restaurant Service confirms/accepts the Order.
    vi) If no vendor has inventory availability, the Order is marked as Failed, and Order Status is updated/notified accordingly via Email/SMS notification to Customer.
    vii) Also, the Order Service initiates Payment Refund process for that Customer's Order Failure, by interacting with Payment Service.
    viii) Payment Service then, processes the Refund for Customer's Order amount depending upon Payment source i.e., either via Customer's Wallet update or Payment Interface.
  
  **Acceptance Criteria:**
    **Given:** An Order is placed, and Payment is confirmed/completed.
    **Pre-Condition:** Vendor inventory availability check operation is possible by Restaurant service.
    **When:** Order Service checks Inventory Availability with both Vendors via Restaurant Service communication.
    **Then:**
      a) If Inventory is available with both Vendors, the most suitable Vendor option is selected by Order Service, and the Order is confirmed/accepted by Restaurant Service.
      b) If no inventory is available, the order is rejected or Failed, Order Failure status is notified via Email/SMS notification to Customer and, 
      c) Payment Refund is initiated & processed via Payment Service for that Order Failure.
  
  
  **5. Title: Notification of Order Status:**
  
  **Description:** Customers should be notified of their order status via Email or SMS.
  
  **Business Rules:**
    i) Notification Service listens for order status events on Kafka.
    ii) Sends notifications based on the order status to Notification System(external), which sends Order Status as Email/SMS notification to Customer.
    iii) If Notification Service fails, fallback notifications are issued.
  
  **Acceptance Criteria:**
    **Given:** Order Status updates are received from Order Service.
    **Pre-Condition:** Customer's contact details(i.e., Email Address and Contact no.) are available from Order Service.
    **When:** Notification Service processes an event to trigger Notification System(external).
    **Then:** Notifications are sent successfully over the Email/SMS, or fallback responses are issued to Customer.
  
  **6. Error Scenarios:**
   **i) Payment Failure:**
    **Description:** If the wallet balance is insufficient and the Payment Interface fails, the Order should be marked as Failed.
    **Acceptance Criteria:** Order Service receives a Payment Failure event and the Order is rejected or Failed.
  
   **ii) Inventory Unavailability:**
    **Description:** If all available vendors have Zero/Nil availability, the Order should be rejected or failed and, Payment Refund should be initiated for Customer.
    **Acceptance Criteria:** Order Service receives an Inventory Unavailable event and marks the order as Failed thereby, initiating full Payment refund for the Customer.
  
   **iii) Notification Failure:**
    **Description:** If the Notification Service fails, fallback notifications should be sent.
    **Acceptance Criteria:** A fallback response with a default message is issued.
  
   **iv) Technical Errors (Service Down/Slow Responses):**
    **Description:** Services should implement resiliency patterns such as retries, circuit breakers, and fallbacks.
    **Acceptance Criteria:**
      a) If a service is unavailable, retries are attempted.
      b) If retries fail, fallback mechanisms are triggered.


  ****b. Technical Requirements and Notes:****
  **General Overview:**
  The Food Order Microservices Application will utilize a Reactive, Event-driven, and Cloud-Native architecture to meet business requirements while adhering to API-First, TDD, and Clean Code principles. It will leverage Google Cloud Services for deployment and observability, Confluent Kafka for event-driven communication, and MongoDB and KSQL DB for data persistence.
  
  **1. Architecture Overview:**
  **i) Microservices Design:**
        **a) Customer Service:** Manages customer details, wallet balances, and bulk/delta imports.
        **b) Order Service:** Acts as the SAGA orchestrator, handling order lifecycle and integration with other services.
        **c) Payment Service:** Processes payments via wallet or Payment Interface.
        **d) Restaurant Service:** Handles inventory checks, vendor selection, and order confirmations.
        **e) Notification Service:** Sends order status notifications via Email/SMS.
        
  **ii) Reactive Programming:** Use Spring Web Flux and Project Reactor to build non-blocking, asynchronous APIs.
    
  **iii) Event-Driven Communication:**
        a) Implemented via Confluent Kafka, ensuring decoupled communication between microservices.
        b) Kafka topics for publishing and subscribing to events, e.g., order.create, payment.processed, inventory.available.
        
  **iv) Database Design:**
        a) MongoDB (Customer Service): Stores customer details and wallet balances.
        b) KSQL DB (Transactional and Time-Series DB): Used for Order, Payment, Restaurant, and Notification Services.
  
  **v) Resiliency and Fault Tolerance:**
        a) Integrate Resilience4j for retries, circuit breakers, and fallbacks.
        b) Implement Dead Letter Queues (DLQ) for failed Kafka messages.
  
  **vi) DevOps and Deployment:**
        a) GitLab CI/CD for automated DevOps pipelines.
        b) Google Kubernetes Engine (GKE) for container orchestration & deployment.
        c) Google Cloud Monitoring for observability and distributed tracing.

****2. Technical Requirements for Business Requirements Specifications:****
  **i) Customer Management:**
      a) MongoDB Operations: 
        i. Enable sharding to handle ~1 million customer records.
        ii. Support for Bulk Import: Process customer details from a JSON file and write to MongoDB.
        iii. Support for Delta Import: Incrementally update customer details and wallet balances.
      b) REST API: Provide endpoints for fetching and updating customer details.
      c) Performance Requirements:
        i. Optimize queries for high-volume reads and writes.
        ii. Ensure sub-second latency for wallet balance validation.
      
  **ii) Order Placement and Validation:**
      a) Order Service as SAGA Orchestrator:
        i. Use synchronous REST communication with Customer Service to validate customer details.
        ii. Publish order.create, payment.initiate events to initiate the event-driven communication.
      b) Validation Rules:
        i. Cart must not be empty.
        ii. Customer details (address, contact, wallet balance) must be validated before order placement.
      c) Scalability: Support concurrent order placement with eventual consistency guarantees.
      d) KSQL DB Operations: Store order status updates (e.g., Created, Confirmed, Failed).
      
  **iii) Payment Processing:**
      a) Payment Service:
        i. Validate Wallet Balance via Order Service as SAGA Orchestrator, through REST API communication with Customer Service.
        ii. Handle payment via:
        iii. Wallet: Update(Debit) Wallet Balance for Payment, if sufficient.
        iv. Payment Interface: Process the payment through Payment Interface, if Wallet Balance is insufficient.
      b) Kafka Topics:
        i. payment.initiate: Subscribe/consume to initiate Payment process after Order is placed.
        ii. wallet.balance.validate: Publish to Validate Wallet Balance of Customer before Processing Payment.
        iii. wallet.balance.update: Publish to update the Wallet Balance, as per following scenarios:
          a.. Payment Debit if it's sufficient or,
          b. Refund credit processed for Inventory Unavailability related Order Failure.
        iv. payment.processed: Publish based on the Payment Process outcome i.e. Success or Failure.
        v. payment.refund: Published to initiate Refund of full Payment amount to Customer in case of Inventory Unavailable based Order Failure.
        vi. refund.processed: Subscribed when Refund is processed and completed to Customer for Order Failure as a result of Inventory Unavailable.
      c) Resiliency:
        i. Retries for wallet balance validation.
        ii. Circuit breaker for Payment Interface failures.
      d) KSQL DB Operations: Record transaction details and payment status.
      
  **iv) Inventory Availability and Vendor Selection:**
      a) Restaurant Service:
        i. Subscribes/consumes order.create, payment.processed and inventory.check events, once Order creation and Payment is successful.
        ii. Performs inventory availability checks in-parallel with at least two Vendors.
        iii. Notify Order Service of inventory availability status.
      b) Order Service:
        i. Select the most suitable vendor based on price and delivery time.
        ii. Publish order.vendor.select event for Restaurant Service to accept/confirm the Order and notify back by publishing order.confirm event for Order Service.
      c) Error Handling:
        i. If no inventory is available, mark the order as Failed.
        ii. Publish payment.refund event to initiate & complete the Payment Refund to Customer via Payment Service.
        iii. Publish order.confirm.notify (with Order Failure status) for Notification Service to notify Customer via Email/SMS and,
        iv. Notification Service then, triggers Notification System (external) to send notifications to Customer as aggregated Response via API Gateway.
      d) KSQL DB Operations: Record inventory check responses and vendor selection details.
      
  **v) Order Status Notifications:**
      a) Notification Service:
        i. Consume order.confirm.notify event from Order Service .
        ii. Send Email/SMS notifications via an external Notification System.
        iii. Publish notification.processed event to Order Service.
      b) Fallback Mechanism: If Notification Service fails, issue fallback notifications.
      c) KSQL DB Operations: Log notification delivery details for traceability.
      
  **vi) Resiliency and Observability:**
      a) Resilience4j:
        i) Retries for transient failures (e.g., Kafka publishing).
        ii) Circuit breakers for dependent service failures.
      b) Google Cloud Observability:
        i) Enable tracing and logging for all services.
        ii) Monitor Kafka topic metrics and DLQ utilization.
  
  **vii) Performance and Scalability:**
      a) Scalability Requirements:
        i. Autoscaling services in GKE to handle traffic spikes.
        ii. Use MongoDB sharding and Kafka partitioning to optimize throughput.
      b) Non-Functional Metrics:
        i. Throughput: Process up to 1000 orders per second.
        ii. Latency: Ensure sub-second response times for APIs.
        
****3. Dev Tools and Tech Stack:****
  1. Language: Java 17
  2. Frameworks: Spring Boot 3.x, Spring WebFlux
  3. Reactive Programming:	Project Reactor
  4. Event-driven Messaging:	Confluent Kafka, Spring Kafka
  5. Databases: MongoDB, KSQL DB
  6. Resiliency:	Resilience4j
  7. API Development: OpenAPI/Swagger, SpringDoc
  8. Observability: Google Cloud Monitoring, Micrometer, OpenTelemetry
  9. Containerization: Docker
  10. Orchestration: Kubernetes (Google Kubernetes Engine)
  11. CI/CD: GitLab CI/CD or Jenkins
  12. Security	Spring Security, OAuth2, Google Identity

****4. Key Non-Functional Requirements (NFRs):****
  1. Performance: Sub-second API latency; 1000 orders/second.
  2. Scalability: Autoscaling services in GKE.
  3. Fault Tolerance:	Resilience4j Circuit Breakers, Retry Mechanism
  4. Security: OAuth2, HTTPS for secure communication.
  5. Observability: Distributed tracing, centralized logging.


********System Design Diagrams:********
******1. High-Level System Design (System Context Diagram):******

  ![image](https://github.com/user-attachments/assets/14f1a8f1-6a18-4387-b396-ff51803e851d)
    
  The High-Level System Design (System Context Diagram) for this Food Order Microservices system includes the following major components:
    
 ******System Components:******
  ****1. Customer (Browser/App):****
      a) Represents the Customer or end-user who interacts with the Food Order Microservices system.
      b) Capabilities:
        i) Select and Add the products/items to the cart.
        ii) Place Food Order request.
        iii) Receives Order Status as response.
    
  ****2. Microservices:****
    **1. Customer Service:**
        a) Purpose:
          i) Manages customer data and wallet balances.
          ii) Exposes APIs for wallet validation during payment processing.
        b) Database: MongoDB (NoSQL) for high performance and scalability.
        c) Database Schema:
          customer_id (String): Unique identifier for the customer.
          name (String): Customer name.
          email (String): Email address of the customer.
          wallet_balance (Decimal): Current wallet balance of the customer.
          address (String): Customer delivery address.
      
  **2. Order Service:**
        a) Purpose:
          i) Acts as SAGA Orchestrator for the Food Order Microservices System lifecycle.
          ii) Handles Synchronous or REST based communication with Customer Services
          iii) Handles asynchronous & event-driven communication with Payment, Restaurant and Notification Microservices resp., using SAGA Orchestrator Microservices pattern.
          iv) Publishes and subscribes to Kafka events for event-driven processing.
        b) Database: KSQL DB (Transactional Database) for real-time query and event processing.
        c) Database Schema:
          i) order_id (String): Unique identifier for the order.
          ii) customer_id (String): Foreign key linking to the customer.
          iii) order_status (Enum): Status of the order (e.g., Created, Confirmed, Delivered).
          iv) total_amount (Decimal): Total amount for the order.
        Kafka Topics:
          i) order.create: Published when a new Oder is placed.
          ii) payment.initiate: Published for Payment Service to initiate Payment process.
          iii) wallet.balance.validate: Subscribe/Consume to Validate Wallet Balance of Customer before Processing Payment.
          iv) wallet.balance.update: Subscribe/Consume to update the Wallet Balance, as per following scenarios:
            a. Payment Debit if it's sufficient or,
            b. Refund credit processed for Inventory Unavailability related Order Failure.
          v) payment.processed: Subscribed/Consumed after payment processed via Wallet Balance (debit) or Payment Interface i.e., either Success or Failure status. 
          vi) refund.initiate: Published for Payment Service to initiate Refund for each Customer in case of Order Failure as a result of Inventory Unavailable.
          vii) refund.processed: Subscribed when Refund is processed and completed to Customer for Order Failure as a result of Inventory Unavailable.
          viii) inventory.check: Published to initiate Restaurant Service for Inventory availability check with both Vendors.
          ix) inventory.available: Subscribed/Consumed to Order Service for inventory availability status check with a vendor.
          x) order.vendor.select: Published when a Vendor is selected by Order service for Restaurant Service.
          xi) order.confirm: Subscribed/Consumed when Order is accepted by Restaurant Service after successful payment and vendor selection.
          xii) order.confirm.notify: Published for Order Confirmation status to Notification Service.
          iv) notification.processed: Subscribed/Consumed for notification processing(send/delivery) event i.e., executed via REST communication with Notification(Email/SMS) System.
    
  **3. Payment Service:**
        a) Purpose:
          i) Processes payments for orders.
          ii) Supports wallet-based payments or external payment gateways (internal component).
          iii) Publishes and Subscribes for payment processing status & payment initiation to Kafka events.
        b) Database: KSQL DB (Transactional Database).
        c) Database Schema:
          i) payment_id (String): Unique identifier for the payment.
          ii) order_id (String): Foreign key linking to the order.
          iii) amount (Decimal): Amount processed for the payment.
          iv) status (Enum): Payment status (Success, Failed).
        d) Kafka Topics:
          i) payment.initiate: Subscribed/Consumed for payment process initiation event after Order is placed from Order Service.
          ii) wallet.balance.validate: Publish to Validate Wallet Balance of Customer before Processing Payment.
          iii) wallet.balance.update: Publish to update the Wallet Balance, as per following scenarios:
            a. Payment Debit if it's sufficient or,
            b. Refund credit processed for Inventory Unavailability related Order Failure.
          iv) payment.processed: Published when payment processed via Wallet Balance (debit) or Payment Interface i.e., either Success or Failure status.
          v) refund.initiate: Subscribed/consumed for initiation Refund to each Customer in case of Order Failure as a result of Inventory Unavailable.
          vi) refund.processed: Published when Refund is processed and completed to Customer for Order Failure as a result of Inventory Unavailable.
         
  **4. Restaurant Service:**
        a) Purpose:
          i) Checks inventory availability with vendors.
          ii) Confirms and accepts orders based on vendor selection.
          iii) Publishes and Subscribes to Kafka events for inventory availability check and vendor selection.
        b) Database: KSQL DB (Transactional Database).
        c) Database Schema:
          i) restaurant_id (String): Unique identifier for the restaurant.
          ii) inventory_id (String): Inventory identifier.
          iii) order_id (String): Foreign key linking to the order.
          iv) availability_status (Boolean): Indicates whether the inventory is available.
          v) price (Decimal): Price offered by the vendor.
        d) Kafka Topics:
          i) inventory.check: Subscribed/Consumed for Inventory availability check with both Vendors. 
          ii) inventory.available: Published to Order Service for inventory availability status check with a vendor.
          iii) order.vendor.select: Subscribed/Consumed for Vendor Selection confirmed from Order service.
          iv) order.confirm: Published to Order Service when Order is accepted/confirmed.
            
  **5. Notification Service:**
        a) Purpose:
          i) Sends Email and SMS notifications to Customers based on Order Status events received via Notification System (External).
          ii) Publishes and subscribes to Kafka events for notification delivery status.
        b) Database: Time-series DB (KSQL from Confluent Ecosystem).
        c) Database Schema:
          notification_id (String): Unique identifier for the notification.
          order_id (String): Foreign key linking to the order.
          message (String): Notification message content.
          timestamp (DateTime): Time of notification creation.
        d) Kafka Topics:
          order.confirm.notify: Subscribed/Consumed for Notification Service after Order confirmation.  
          notification.processed: Published for notification processing(send/delivery) event i.e., executed via REST communication with external Notification(Email/SMS) System.

  ****3. Event Backbone:****
  **Confluent Kafka:** Kafka acts as the event backbone, enabling asynchronous communication between microservices.
    **Kafka Topics Overview:**
        a. order.create: Publish-Subscribe/Consume as an initiation event when a new Oder is placed.
        b. payment.initiate: Publish-Subscribe/Consume for payment process initiation event after Order is placed from Order Service. 
        c. wallet.balance.validate: Publish-Subscribe/Consume to Validate Wallet Balance of Customer before Processing Payment.
        d. wallet.balance.update: Publish-Subscribe/Consume to update Wallet Balance, as per following scenarios:
          i) Payment Debit if it's sufficient or,
          ii) Refund credit processed for Inventory Unavailability related Order Failure.
        e. payment.processed: Publish-Subscribe/Consume when payment processed via Wallet Balance (debit) or Payment Interface i.e., either Success or Failure status.
        f. refund.initiate: Publish-Subscribe/Consume to initiate and process complete Refund for each Customer in case of Order Failure as a result of Inventory Unavailable.
        g. refund.processed: Subscribed when Refund is processed and completed to the Customer for Order Failure as a result of Inventory Unavailable.
        h. inventory.check: Publish-Subscribe/Consume to Check Vendor Availability.
        i. inventory.available: Publish-Subscribe/Consume for checking the Inventory availability with both the Vendors of the Restaurant.
        j. order.vendor.select: Publish-Subscribe/Consume when a Vendor option serving with lowest price and quickest delivery is selected.
        k. order.confirm: Publish-Subscribe/Consume when Order is confirmed/accepted by the Restaurant after successful payment and vendor selection.
        l. order.confirm.notify: Publish-Subscribe/Consume for Order Confirmation Status (i.e., Success or Failure) notification.
        m. notification.processed: Publish-Subscribe/Consume for notification processing(send/delivery) event i.e., executed via REST communication with external Notification(Email/SMS) System.
        
  ****4. External Services:****
    **Notification (Email/SMS) System:**
      a) Purpose: Sends Email and SMS notifications for Customer's Order Status to API Gateway.
      b) Recommended OSS Tools:
        i) Email: SendGrid
        ii) SMS: Twilio
        
  ****5. GCP Components:****
  **Google Cloud API Gateway:**
    a) Purpose: 
      i) Single entry point for all Customer(Browser/App) requests.
      ii) Route the Customer's Food Order requests to Order Microservice.
      iii) Perform Load Balancing, Security posture(authentication, authorization), Response Aggregator.
      iv) Responds the Order Status received from Notification(Email/SMS) System to Customer.
  **GKE (Google Kubernetes Engine)**
    a) Purpose:
      i) Hosts all microservices in containerized environments.
      ii) Ensures scalability, resilience, and fault tolerance.
  **Google Cloud Logging & Monitoring**
    a) Purpose:
      i) Provides observability into system performance.
      ii) Monitors logs, metrics, and alerts.

  ****6. Legends:****
    a) Blue: Microservices
    b) Green:	Databases, REST API calls & DB operations
    c) Yellow: Confluent Kafka & Event Topics
    d) Orange: External Services
    e) Light Purple: GCP Components

******2. Sequence Diagrams:******

Sequence Diagrams for all the possible scenarios with respect to Food Order Microservices System operational results, as follows:

****1. Positive/Success Flow (Happy Path):****

![image](https://github.com/user-attachments/assets/879283a4-0984-4d63-adc8-9ade0f4c5871)

**Happy Path Scenario: Flow Description**

  a). Customer places a Food Order request via the browser/app.
  b). API Gateway routes the Order request to Order Service.
  c). The Order Service (SAGA Orchestrator):
    i) Reads and Validates Customer Details (i.e., Profile, Address, Contact and Wallet Balance details) via REST API/Synchronous communication with Customer Service.
    ii) Publishes order.create, payment.initiate event notifications to initiate Event-driven communications as a SAGA Orchestrator further.
  d) Payment Service:
    i) Subscribes & consumes the payment.initiate event to initiate Payment process.
    ii) Publishes wallet.balance.validate event to validate Wallet Balance before Payment processing. 
  e) Order Service:
    i) Subscribes & consumes wallet.balance.validate event. 
    ii) Validates Wallet Balance through REST API communication with Customer Service.
    iii) If Wallet Balance is sufficient, publishes wallet.balance.update event.
  d) Payment Service:
  e) Subscribes & consumes wallet.balance.update event and, processes the Payment via updating(debit) Wallet Balance, if sufficient.
  f) Otherwise, processes the payment through the Payment Interface(Internal).
  g) Publishes payment.processed(Success)event notification.
  h) Order service:
    i) Subscribes and consumes payment.processed(Success) event and, publishes inventory.check event for further communication with Restaurant Service. 
  i) Restaurant Service:
    i) Checks inventory availability with both Vendors after subscribing and consuming order.create, payment.processed(Success),inventory.check events.
    ii) Publishes inventory.available(Success) event for Order Service, once it receives API response for Inventory Availability with both Vendors.
  j) Order Service:
    i) Subscribes and consumes inventory.available(Success) event from Restaurant Service and then,
    ii) Selects the Vendor option with lowest price and fastest delivery, and further publishes order.vendor.select event for Restaurant Service.
  k) Restaurant Service:
    i) Subscribes and consumes order.vendor.select event notification with selected Vendor details from Order Service and,
    ii) Confirms/accepts the Food Order for fulfilment. It further publishes order.confirm(Success/Accepted) event.
  l) Order Service:
    i) Subscribes and consumes order.confirm (Success/Accepted) event and, hence performs the Order Confirmation.
    ii) Publishes order.confirm.notify (Success)event.
  m) Notification Service:
    i) Subscribes and consumes the order.confirm.notify (Success)event having the Order Confirmation (Success) and Customer details.
    ii) Sends an Order Confirmation notification to Notification System(External) via REST API communication.
  n) Notification Service:
    i) Sends the Email/SMS Notifications for Order Confirmation(Success) status to API Gateway. 
    ii) Publishes notification.processed(Success) event.
  o) Order Service: Subscribes and consumes notification.processed(Success) event.
  p) Hence, the event-driven communication for Success/Positive or Happy Path scenario is completed there by, implementation of SAGA orchestration.
  q) API Gateway: Sends Order Confirmation(Success) Status as the Food Order response back to the Customer.
  r) Customer Service: Receives Food Order Confirmation or completion status as a response to the initial Food Order request and hence, completes the Successful execution flow.

****2. Negative/Failure Flows:****
  **i) Payment Failure:**
  
  ![image](https://github.com/user-attachments/assets/8f51d23c-2cd6-4b4c-93d9-b4aa8fb7a73f)
  
  **Failure Scenario Flow: Payment Failure**
    1. Customer places and Order via Order Service.
    2. Customer Service validates wallet balance, as requested by Payment Service.
    3. The payment fails due to wallet balance insufficiency as well as Payment Interface issues.
    4. The order is canceled, and the customer is notified with Payment Failure Notification via Email/SMS.
  
  **ii) Inventory Unavailable Failure:**
  
  ![image](https://github.com/user-attachments/assets/42005581-fb3d-42b1-966b-e70807ebfef7)
  
  **Failure Scenario: Inventory Unavailable**
    1. Customer places an order via Order Service.
    2. Customer Service validates wallet balance, as requested by Payment Service.
    3. Payment Service processes payment successfully by either Wallet Ballance deduction, or through Payment Interface, if Wallet Balance is insufficient.
    4. The Restaurant Service fails to confirm inventory availability i.e. Inventory is Unavailable with Vendors.
    5. Order Service rejects the Order as Failed. It also initiates and processes Payment Refund to the Customer.
    6. Customer is notified with Inventory Unavailable Failure Notification via Email/SMS.


****3. Exception Flow (Technical Errors):****

![image](https://github.com/user-attachments/assets/8241c93c-f03f-4bf3-be49-caea678737ca)

**Scenario 1: Customer Service is Down or Slow:**
  1. Customer places and Order via Order Service.
  2. Order Service requests Customer Service via REST API to Fetch/Read Customer Details.
  3. If Customer Service is down. Circuit Breaker Opens Circuit to avoid Cascading failures and triggers Fallback notifications after retry timeout.
  4. Order Service publishes Order Failure event via Kafka topic order.confirm.notify for Notification Service.
  5. If Customer Service is responding Slow, then Retry Mechanism is activated and triggers Retry Notifications at specified intervals.

**Scenario 2: Notification Service is Down or Slow:**
  **1. Notification Service Down:**
    a) Circuit Breaker in the Notification Service opens to prevent further failures.
    b) Order Service logs the failure and re-queues the notification event.
  **2. Notification Service Slow:** Retry mechanism attempts to resend the notification before logging the failure.


******Tech Stack & Tools:******

****1. Core Development:****
  **a) Programming Language:**
    **Java 17:** Enables modern language features like records, sealed classes, and pattern matching.
  
  **b) Frameworks:**
    i) **Spring Boot 3.x:** Microservice framework supporting reactive programming with Spring WebFlux.
    ii) **Spring Cloud:** For service discovery, config server, and distributed tracing.
    iii) **Spring Data Reactive:** Reactive access to MongoDB.
    iv) **Spring Security:** OAuth2/OpenID Connect for securing APIs.
    v) **Spring Kafka:** Integration with Confluent Kafka for event-driven architecture.
    vi) **Confluent Schema Registry:** Ensure compatibility between producers and consumers.
    vii) **Spring Validation:** Input validation and constraints.
  
  **c) Reactive Framework:**
    i) **Project Reactor:** For building non-blocking, asynchronous systems.
    ii) **Web-Client:** For reactive HTTP communication.
  
  **d) Database:**
    i) **Reactive MongoDB:** NoSQL database used by Customer Service for scalability and reactive support.
    ii) **Confluent KsqlDB:** Transactional & Time-Series DB for Order Service, Payment Service, Restaurant Service & Notification Service.
  
  **e) Messaging/Event Streaming:**
    **Confluent Kafka:** Distributed event-driven messsaging & event-streaming platform.
  
 ****2. Development Tools:****
  **a) API Development:**
    i) Open API/Swagger Code Generator: API-First development thru Code Generator.
    ii) Postman/Insomnia: API testing.
  **b) IDE:** IntelliJ IDEA
  **c) Build Tools:**  Maven
  **d) Version Control:**
    i) GitLab for version control.
    ii) GitLab CI for CI/CD pipelines.  
  **e) Code Quality and Coverage:**
    i) SonarQube: Static code analysis for clean code.
    ii) JaCoCo: Code coverage reporting.
  **f) Testing:**
    i) JUnit 5: Unit testing.
    ii) Mockito: Mocking dependencies.
    iii) Testcontainers: Integration testing with ephemeral containers.
    iv) Postman: API testing.
    
****3. Google Cloud Services:****
  **i) API Gateway:** Google Cloud Endpoints for API management.
  **ii) Kubernetes:** Google Kubernetes Engine (GKE) for container orchestration.
  **iii) Cloud Pub/Sub:** As a complementary messaging service (if Confluent Kafka not available).
  **iv) Cloud Monitoring:** Observability and logging with Google Operations Suite.
  **v) Cloud Secrets Manager:** Secure management of credentials and keys.

****4. DevOps Tools:****
  **i) Containerization:**
    a) Docker: For containerized microservices.
    b) CI/CD: GitLab CI/CD Pipeline for build, test, and deploy to GKE. (TBD)
  **ii) Observability:**
    a) Prometheus + Grafana: Metrics collection and visualization.
    b) Google Cloud Logging: Centralized logging.
    c) Distributed Tracing: Open-Telemetry with GCP integrations.

### CI/CD with Auto DevOps

This template is compatible with [Auto DevOps](https://docs.gitlab.com/ee/topics/autodevops/).

If Auto DevOps is not already enabled for this project, you can [turn it on](https://docs.gitlab.com/ee/topics/autodevops/#enabling-auto-devops) in the project settings.
