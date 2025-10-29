# Personalised Product Recommendations Architecture

**MACH-based Cloud Native Solution for Specsavers**



## Clarification Questions

- What are the primary business goals for this feature - conversion uplift, AOV increase, or customer retention?

- How will this system integrate with existing Specsavers systems like e-commerce, CRM, and inventory management?

- What are the data privacy requirements and GDPR compliance needs for handling customer data?

- What is the recommended placement strategy for recommendations across homepage, product pages, cart, and email?

- What are the budget and timeline constraints for implementing this personalized recommendation system?

## Key PILLARS 

- **Scalability**: Independent scaling of microservices ensures that services can handle their unique workloads without impacting others.
- **Speed**: Optimized data flows and caching strategies enable real-time recommendations and fast response times.
- **Resilience**: Asynchronous communication and circuit breakers mitigate the risk of cascading failures.
- **Flexibility**: A modular approach allows for iterative development, making introducing new features or updating existing ones easier.
- **Observability**: Centralized logging and monitoring enable debugging and performance analysis for distributed services.
- **Security**: API Gateway and token-based authentication (OAuth2) ensure a secure entry point.
   
## High Level Architecture

```
┌─────────────────┐
│   CLIENT        │       
└─────────────────┘
         ├───────────────────────┐
         │                       │
    (Access Token)         (Sends JWT)
         │                       │
         ▼                       ▼
┌──────────────────┐ JWT ┌──────────────────┐
│ AUTHORIZATION    │◄────│   API GATEWAY    │
│ SERVER           │────►│                  │
└──────────────────┘     └──────────────────┘
         │                      │
         │ (Access Token)       │ (JWT)
         ▼                      ▼
┌──────────────────┐    ┌──────────────────┐
│   USER STORE     │    │  REST API/KAFKA  │
│                  │    │                  │
└──────────────────┘    └──────────────────┘
                                │
                                │ 
                                ▼
                    ┌─────────────────────────┐
                    │ USER PROFILE SERVICE    │
                    ├─────────────────────────┤
                    │ PRODUCT CATALOG SERVICE │
                    ├─────────────────────────┤
                    │ RECOMMENDATION ENGINE   │
                    │ SERVICE                 │
                    ├─────────────────────────┤
                    │ BEHAVIOR TRACKING       │
                    │ SERVICE                 │
                    └─────────────────────────┘



```

## Low Level Architecture

### Core Components

- Microservices Layer

  - User Profile Service: Customer preferences and history

  - Product Catalog Service: Glasses, lenses, and accessories data

  - Recommendation Engine: Generates personalized suggestions

  - Behavior Tracking Service: Captures browsing and purchase events

- Content Service: Manages promotional content and assets

### Service Communication 
Services communicate via **REST APIs for synchronous requests** and **Apache Kafka for asynchronous event streaming**. The **API Gateway (GraphQL/REST) routes client requests**, while **services** exchange data through defined **contracts**.




```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SPECSAVERS RECOMMENDATION PLATFORM                │
│                        Microservices Architecture - MACH Based              │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   WEB APP   │  │ MOBILE APP  │  │ IN-STORE    │  │   EMAIL     │
│             │  │             │  │ TABLETS     │  │  SERVICE    │
└──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
       │                │                │                │
       └────────────────┼────────────────┼────────────────┘
                        │                │
                 ┌──────▼────────────────▼──────┐
                 │         API GATEWAY          │                                                     
                 │  ┌────────────────────────┐  │
                 │  │ GraphQL • REST • Auth  │  │
                 │  │ Rate Limiting • Caching│  │
                 │  └────────────────────────┘  │
                 └───────────────┬──────────────┘
                                 │
    ┌────────────────────────────┼─────────────────────────────────────────┐
    │                      MICROSERVICES LAYER                             │
    │                                                                      │
    │  ┌─────────────┐  ┌─────────────┐  ┌─────────────-┐  ┌─────────────┐ │
    │  │   USER      │  │  PRODUCT    │  │RECOMMENDATION│  │ BEHAVIOR    │ │
    │  │  PROFILE    │  │  CATALOG    │  │   ENGINE     │  │ TRACKING    │ │
    │  │ SERVICE     │  │ SERVICE     │  │              │  │ SERVICE     │ │
    │  └──────┬──────┘  └──────┬──────┘  └──────┬──────-┘  └─────┬──────-┘ │
    │         │                │                │                │         │
    └─────────┼────────────────┼────────────────┼────────────────┼────────-┘
              │                │                │                │
    ┌─────────▼────────────────▼────────────────▼────────────────▼─────────┐
    │                        DATA & AI/ML LAYER                            │
    │                                                                      │
    │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
    │  │  USER       │  │  PRODUCT    │  │   VECTOR    │  │  EVENT      │  │
    │  │ PROFILES    │  │ CATALOG DB  │  │  DATABASE   │  │ STREAMING   │  │
    │  │  DATABASE   │  │             │  │             │  │   (Kafka)   │  │
    │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  │
    │         │                │                │                │         │
    │         │                │                │                │         │
    │         │                │                │                │         │
    │         ▼                ▼                ▼                ▼         │
    │  ┌───────────────────────────────────────────────┐  ┌─────────────┐  │         
    │  │           AI/ML PLATFORM                      │◄►│  CONTENT    │  │         
    │  │                                               │  │  SERVICE    │  │
    │  └───────────────────────────────────────────────┘  └─────────────┘  │
    │                                                                      │
    └──────────────────────────────────────────────────────────────────────┘
              │                                                  │
    ┌─────────▼──────┐                                  ┌────────▼────────┐
    │  HEADLESS CMS  │                                  │   FEATURE       │
    │                │                                  │     STORE       │
    │ • Contentful   │                                  │                 │
    │ • Storyblok    │                                  │ • Real-time     │
    │ • Content      │                                  │   Features      │
    │   Fragments    │                                  │ • Batch         │
    │ • Promotions   │                                  │   Features      │
    └────────────────┘                                  └─────────────────┘                        

```


### Data Layer

- User Profiles: Customer preferences and history
- Product Database: Complete catalog with attributes
- Vector Database: Similarity search for recommendations
- Event Streaming: Real-time user behavior tracking

## Data Flow
                     

                          ┌─────────────────┐
                          │   USER          │
                          │  INTERACTION    │
                          └────────┬────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
         ┌────▼────┐         ┌─────▼─────┐        ┌─────▼─────┐
         │BEHAVIOR │         │   API     │        │ CONTENT   │
         │TRACKING │         │ GATEWAY   │        │ SERVICE   │
         └────┬────┘         └─────┬─────┘        └─────┬─────┘
              │                    │                    │
         ┌────▼────┐         ┌─────▼─────┐        ┌─────▼─────┐
         │ EVENT   │         │   USER    │        │ HEADLESS  │
         │STREAMING│         │  PROFILE  │        │   CMS     │
         └────┬────┘         └─────┬─────┘        └───────────┘
              │                    │                    │
         ┌────▼────┐         ┌─────▼─────┐              │
         │ AI/ML   │         │  PRODUCT  │              │
         │PLATFORM │         │  CATALOG  │              │
         └────┬────┘         └─────┬─────┘              │
              │                    │                    │
              └────────┐     ┌─────┘                    │
                       │     │                          │
                  ┌────▼─────▼─────┐                    │
                  │ RECOMMENDATION │◄───────────────────┘
                  │   ENGINE       │
                  └───────┬────────┘
                          │
                  ┌───────▼────────┐
                  │     API        │
                  │   GATEWAY      │
                  └───────┬────────┘
                          │
                  ┌───────▼────────┐
                  │    RESPONSE    │
                  │    TO USER     │
                  └────────────────┘


### AI/ML Layer

- Real-time Inference: Immediate recommendation generation

- Model Training: Batch processing for algorithm improvement

- Feature Store: Consolidated data for ML models


## Real Time tracking

- The AI/ML Platform is the intelligent core that processes data to generate insights and predictions. Its primary role is to move the system from being **reactive** (" say show product X") to being **proactive** ("we recommend product X for you") enabling hyper-personalization.

- Real-time tracking events flow through Kafka to the AI platform, which updates recommendation models. Content Service syncs with Headless CMS via webhooks, and all services leverage Redis for caching to reduce latency.


- Content is Created in Headless CMS: A marketer creates a new promotional campaign  in say  Contentful, targeting students.

- Content is Served via Content Service: The Content Service fetches the promotion and makes it available to the API Gateway.

- AI/ML Informs the Personalization Logic: Separately, the AI/ML platform has identified that a specific user  is a high-potential candidate for this promotion. This intelligence is embedded in the Recommendation Engine.


## What happens next when a user logs in:

https://github.com/kukuu/AI-recommendation-real-time-tracking/blob/main/README.md

