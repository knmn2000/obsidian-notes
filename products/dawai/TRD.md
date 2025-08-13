# Technical Requirements Document: WhatsApp Medicine Reminder System

**Version:** 1.0 **Date:** July 8, 2025 **Author:** Gemini

## 1. Introduction

This document outlines the technical requirements and system design for a WhatsApp-based medicine reminder service. The primary goal is to provide a user-friendly, reliable, and cost-effective solution for elderly individuals to manage their medication schedules.

### 1.1. Purpose

The purpose of this system is to send automated, personalized medicine reminders to users via WhatsApp at specified times and dates, helping them adhere to their medication regimens.

### 1.2. Scope

**In Scope:**

- User interface for setting up, viewing, editing, and deleting medicine reminders.
    
- Secure storage of user and reminder data.
    
- Automated scheduling and triggering of WhatsApp messages.
    
- Integration with a WhatsApp Business API provider for message delivery.
    
- Focus on cost-effectiveness using serverless AWS services.
    
- Designed with considerations for elderly users (simple UI, minimal clicks).
    

**Out of Scope (for this phase):**

- Full-fledged telemedicine platform.
    
- Integration with Electronic Health Records (EHR) systems.
    
- Complex analytics dashboards for user adherence.
    
- Direct payment processing within the reminder system (assumes subscription managed externally).
    
- Mobile native applications (focus is on web-based setup and WhatsApp delivery).
    

### 1.3. Target Audience

- Elderly individuals who are comfortable with basic WhatsApp usage but may struggle with complex mobile interfaces or navigating to external websites.
    
- Business administrators managing user reminders.
    

## 2. Functional Requirements

### 2.1. User Management

- The system shall allow users to register and manage their profiles (implicitly via authentication).
    
- The system shall associate reminders with specific users.
    

### 2.2. Reminder Setup & Management

- Users shall be able to set up new medicine reminders via a web-based interface.
    
- For each reminder, users shall specify:
    
    - Medicine Name (e.g., "Paracetamol", "Vitamin D")
        
    - Dosage (e.g., "500mg", "1 tablet")
        
    - Instructions (e.g., "After breakfast", "Before bed", "Do not take with X")
        
    - Time of day (e.g., 8:00 AM, 1:30 PM)
        
    - Recurrence:
        
        - Daily
            
        - Specific days of the week (e.g., Mon, Wed, Fri)
            
        - One-time (specific date and time)
            
    - Active/Inactive status (to pause/resume reminders).
        
- Users shall be able to view all their active and inactive reminders.
    
- Users shall be able to edit existing reminder details.
    
- Users shall be able to delete reminders.
    

### 2.3. Message Delivery

- The system shall send WhatsApp messages to users at the exact scheduled time.
    
- Messages shall be personalized with user and medicine details.
    
- Messages shall utilize pre-approved WhatsApp Message Templates.
    
- The system shall handle different reminder scenarios (e.g., single medicine, multiple medicines, food instructions, "do not take together" warnings) via appropriate template usage or a hybrid approach.
    

### 2.4. User Interface (UI) for Elderly Users

- The setup interface shall feature large, legible text and clear visual cues.
    
- Buttons and interactive elements shall be large and easy to tap.
    
- Navigation shall be simple, minimizing clicks and avoiding complex gestures.
    
- The system shall aim to keep users within the WhatsApp environment for medicine details (via follow-up messages or in-app web views).
    

## 3. Non-Functional Requirements

### 3.1. Performance

- **Message Delivery:** Reminders shall be delivered within a few seconds of their scheduled time.
    
- **UI Responsiveness:** The web-based setup interface shall load quickly and be responsive across devices.
    
- **Backend Latency:** API responses for reminder management shall be low-latency (e.g., < 500ms).
    

### 3.2. Scalability

- The system shall be capable of handling 10,000+ messages per day initially, with the ability to scale to millions of messages per month without significant architectural changes.
    
- The database shall support a growing number of users and reminders (millions of records).
    

### 3.3. Cost-Effectiveness

- The system shall primarily leverage AWS serverless services (Lambda, DynamoDB, EventBridge) to minimize operational costs.
    
- Maximize utilization of AWS Free Tier limits.
    
- Pay-as-you-go model for services exceeding free tier.
    

### 3.4. Reliability & Availability

- Message delivery shall have high reliability (e.g., > 99.9% success rate).
    
- The system shall be highly available, ensuring reminders are sent even during peak loads or minor service disruptions.
    
- Error handling and retry mechanisms shall be in place for message sending failures.
    

### 3.5. Security

- User authentication for access to reminder setup.
    
- Data encryption at rest (DynamoDB) and in transit (HTTPS, WhatsApp API).
    
- Strict adherence to WhatsApp Business Messaging Policy and WhatsApp Messaging Guidelines.
    
- Compliance with Indian data privacy laws (e.g., Digital Personal Data Protection Act, 2023).
    
- Least privilege principle for all AWS IAM roles.
    

### 3.6. Maintainability

- The system shall be built with modular, loosely coupled components (Lambda functions).
    
- Code shall be well-commented and follow best practices.
    
- Infrastructure as Code (IaC) is preferred for deployment.
    

### 3.7. Compliance

- **WhatsApp Policy:** Strict adherence to WhatsApp's policies regarding health-related information, medical products, and template content. This is a critical constraint for direct medicine name transmission.
    
- **GST:** The business shall obtain and maintain GST registration in India.
    
- **DLT (if SMS used):** If SMS is adopted as an alternative channel, full compliance with TRAI's DLT regulations (Entity, Header, and Template registration) is mandatory.
    

## 4. System Architecture

The system follows a serverless, event-driven architecture on AWS, complemented by a WhatsApp Business API provider.

```
+-------------------+       +-------------------+       +-------------------+
|                   |       |                   |       |                   |
|  User Interface   |       |  AWS API Gateway  |       | AWS Lambda        |
|  (Web Frontend)   +------>| (REST API)        +------>| (Reminder Mgmt)   |
|  (S3 + Cloudflare)|       |                   |       |                   |
+-------------------+       +-------------------+       +-------------------+
        ^                                                          |
        |                                                          v
        |                                                 +-----------------+
        |                                                 | Amazon DynamoDB |
        |                                                 | (Medicine       |
        |                                                 | Reminders Table)|
        |                                                 +-----------------+
        |                                                          ^
        |                                                          |
        |                                                          |
+-------------------+       +-------------------+       +-------------------+
|                   |       |                   |       | AWS Lambda        |
| WhatsApp User     |<------| WhatsApp Business |<------| (WhatsApp Sender) |
| (Receives Message)|       | API Provider      |       |                   |
+-------------------+       +-------------------+       +-------------------+
        ^                                                          ^
        |                                                          |
        | (Triggered by Precise `at()` Rules)                      | (Updates nextSendTimestamp)
        |                                                          |
+-------------------+       +-------------------+       +-------------------+
|                   |       |                   |       | AWS Lambda        |
| Amazon EventBridge|<------| EventBridge Rule  |<------| (Daily Scheduler) |
| (Scheduled Events)|       | (Daily Cron)      |       |                   |
+-------------------+       +-------------------+       +-------------------+
```

## 5. Component Details

### 5.1. Frontend (User Setup Interface)

- **Technology:** HTML, CSS, JavaScript.
    
- **Hosting:** Static website hosted on **AWS S3**.
    
- **CDN/Security (Optional but Recommended):** **Cloudflare Free Tier** for global caching, fast DNS, SSL, and basic DDoS protection.
    
- **Design:** Adheres to elderly-friendly UI principles: large text, high contrast, prominent buttons, minimal clicks, simple navigation.
    

### 5.2. AWS API Gateway

- **Role:** Public-facing HTTP endpoint for the user setup interface.
    
- **Endpoints:** Defines RESTful API endpoints (e.g., `POST /reminders`, `GET /reminders`, `PUT /reminders/{id}`, `DELETE /reminders/{id}`).
    
- **Integration:** Integrates directly with AWS Lambda functions as backend.
    
- **Security:** Handles user authentication (e.g., via AWS Cognito Authorizer, if user accounts are managed), authorization, and provides HTTPS.
    
- **Features:** Can configure throttling, caching, and integrates with CloudWatch for monitoring and logging.
    

### 5.3. AWS Lambda Functions (Python Runtime)

- **`ReminderManagementFunction`:**
    
    - **Trigger:** AWS API Gateway.
        
    - **Purpose:** Handles CRUD (Create, Read, Update, Delete) operations for medicine reminders in DynamoDB.
        
    - **Logic:** Validates input, interacts with DynamoDB, calculates and sets `nextSendTimestamp` for new/updated recurring reminders.
        
- **`DailyReminderSchedulerFunction`:**
    
    - **Trigger:** Daily AWS EventBridge cron rule (e.g., `cron(0 0 * * ? *)` UTC).
        
    - **Purpose:** Orchestrates the scheduling of individual reminder messages for the day.
        
    - **Logic:**
        
        1. Queries DynamoDB's `ActiveRemindersByNextSendTime` GSI for all active reminders with `nextSendTimestamp` falling within the current day's window.
            
        2. For each due reminder, it programmatically creates a **one-time, precise EventBridge `at()` rule** with the exact send time.
            
        3. The target of this `at()` rule is the `WhatsAppSenderFunction`, passing the reminder details as input.
            
- **`WhatsAppSenderFunction`:**
    
    - **Trigger:** Precise, one-time EventBridge `at()` rules.
        
    - **Purpose:** Sends the actual WhatsApp message.
        
    - **Logic:**
        
        1. Receives reminder details (user ID, medicine name, dosage, instructions) as input.
            
        2. Constructs the WhatsApp message payload using an **approved WhatsApp Message Template**.
            
        3. Makes an API call to the configured **WhatsApp Business API Provider**.
            
        4. **Crucially:** After successful sending, updates the `MedicineReminders` table in DynamoDB:
            
            - Sets `lastSentAt` to the current timestamp.
                
            - **Calculates and updates the `nextSendTimestamp` for the** _**next**_ **occurrence of this recurring reminder.** This ensures it's picked up by `DailyReminderSchedulerFunction` on the subsequent day.
                
            - If it's a one-time reminder, sets `isActive = false`.
                

### 5.4. Amazon DynamoDB

- **Role:** Serverless NoSQL database for storing all reminder and user-related data.
    
- **Table:** `MedicineReminders`
    
    - **Partition Key (PK):** `userId` (String)
        
    - **Sort Key (SK):** `reminderId` (String - UUID)
        
    - **Attributes:** `medicineName`, `dosage`, `instructions`, `scheduleType`, `dailyTime`, `daysOfWeek`, `specificDate`, `nextSendTimestamp` (Unix Epoch), `isActive` (Boolean), `createdAt` (Unix Epoch), `lastSentAt` (Unix Epoch, optional).
        
- **Global Secondary Index (GSI):** `ActiveRemindersByNextSendTime`
    
    - **GSI PK:** `isActive` (Boolean)
        
    - **GSI SK:** `nextSendTimestamp` (Number - Unix Epoch)
        
    - **Purpose:** Enables efficient querying for all active reminders due within a specific time range, without scanning the entire table.
        

### 5.5. Amazon EventBridge

- **Role:** Serverless event bus and scheduler.
    
- **Daily Cron Rule:** A single EventBridge rule (e.g., `cron(0 0 * * ? *)`) triggers the `DailyReminderSchedulerFunction` once per day.
    
- **One-Time `at()` Rules:** Programmatically created by `DailyReminderSchedulerFunction` to trigger `WhatsAppSenderFunction` at precise future timestamps for each individual message. These rules are self-deleting after triggering.
    

### 5.6. WhatsApp Business API Provider

- **External Service:** A third-party provider (e.g., Twilio, AiSensy, Exotel, MSG91) that facilitates sending messages via the official WhatsApp Business API.
    
- **Functionality:** Handles the actual delivery of messages to WhatsApp users.
    
- **Requirement:** Requires pre-approved WhatsApp Message Templates. Content will be subject to WhatsApp's strict policies, especially for health-related information. A hybrid approach (initial generic message + in-app web view) or special Meta partnership might be necessary for direct medicine details.
    

## 6. Data Flow

1. **User Creates/Updates Reminder:** User interacts with the web frontend. Frontend sends an API request (e.g., `POST /reminders`) to **AWS API Gateway**.
    
2. **Reminder Management:** API Gateway invokes `ReminderManagementFunction` (Lambda). This Lambda validates the request, stores/updates the reminder in **DynamoDB**, and calculates the initial `nextSendTimestamp`.
    
3. **Daily Scheduling:** At a predefined time (e.g., midnight UTC), an **EventBridge cron rule** triggers `DailyReminderSchedulerFunction` (Lambda).
    
4. **Due Reminders Identification:** `DailyReminderSchedulerFunction` queries the `ActiveRemindersByNextSendTime` GSI in **DynamoDB** to find all reminders scheduled for the current day.
    
5. **Precise Trigger Creation:** For each identified reminder, `DailyReminderSchedulerFunction` programmatically creates a **one-time EventBridge `at()` rule** with the exact send time. This rule's target is the `WhatsAppSenderFunction`, passing the reminder details as input.
    
6. **Message Sending:** When an `at()` rule's time arrives, it triggers `WhatsAppSenderFunction` (Lambda). This Lambda constructs the message using an **approved WhatsApp Message Template** and calls the **WhatsApp Business API Provider**.
    
7. **Status Update:** After sending, `WhatsAppSenderFunction` updates the `lastSentAt` and calculates/updates the `nextSendTimestamp` for the _next_ occurrence of the recurring reminder in **DynamoDB**.
    
8. **User Receives Message:** The WhatsApp Business API Provider delivers the message to the user's WhatsApp client.
    

## 7. Security Considerations

- **Authentication & Authorization:** API Gateway and Lambda will enforce user authentication (e.g., via Cognito) and authorization (IAM roles) to ensure users can only manage their own reminders.
    
- **Least Privilege:** All Lambda functions will have IAM roles with only the minimum necessary permissions to perform their tasks (e.g., `WhatsAppSenderFunction` can only write to `MedicineReminders` table, not delete it).
    
- **Data Encryption:** DynamoDB encrypts data at rest by default. HTTPS ensures data encryption in transit between client, API Gateway, Lambda, and the WhatsApp API Provider.
    
- **Sensitive Data Handling:** Avoid storing highly sensitive personal health information directly in plain text. If necessary, explore encryption at the application layer.
    
- **WhatsApp Policy Compliance:** Continuously monitor and adapt to WhatsApp's evolving Business Messaging Policy, especially regarding health content.
    

## 8. Compliance Considerations

- **WhatsApp Business Messaging Policy:** Critical for template approval and avoiding account suspension. The direct inclusion of medicine names and dosages in templates is highly restricted; a hybrid approach or special Meta partnership may be necessary.
    
- **GST Registration:** Mandatory for the business in India due to projected turnover.
    
- **Indian Data Privacy Laws:** Ensure compliance with the Digital Personal Data Protection Act, 2023, and other relevant regulations for handling personal data.
    
- **DLT (Distributed Ledger Technology):** If SMS is used as an alternative or supplementary channel, full DLT compliance (Entity, Header, Template registration) is required in India.
    

## 9. Deployment Strategy

- **Infrastructure as Code (IaC):** Use AWS Serverless Application Model (SAM) or AWS CloudFormation to define and deploy all AWS resources (Lambda functions, DynamoDB tables, EventBridge rules, API Gateway endpoints). This ensures consistent, repeatable deployments.
    
- **Version Control:** All code and IaC templates will be managed in a version control system (e.g., Git).
    
- **Monitoring & Logging:** Leverage AWS CloudWatch for centralized logging of Lambda execution, API Gateway access, and system metrics. Set up alarms for critical events (e.g., high error rates, low free tier remaining).