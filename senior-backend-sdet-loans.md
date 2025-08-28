# Senior Backend Developer (SDET) - Loan Syndication Platform

## Business Context & Core Domain Logic

You are tasked with designing and implementing the backend systems for a **loan syndication platform** that orchestrates complex multi-billion dollar transactions between borrowers and 100-300 lenders per deal. The platform manages the complete transaction lifecycle from origination through closing, handling critical business operations including:

### Primary Business Functions
- **Deal Origination & Structuring**: Process borrower applications, perform credit analysis, structure loan terms, and prepare transaction documentation
- **Market Testing & Pre-Launch**: Execute informal market soundings, develop pricing guidance, and finalize launch strategies
- **Syndication Execution**: Manage formal launch announcements, coordinate management presentations, execute book building processes, handle market flex pricing adjustments, and determine final allocations
- **Transaction Closing**: Process legal documentation, satisfy condition precedents, coordinate funding across hundreds of lenders, and establish administrative agent operations

### Critical Business Rules & Constraints
- **Transaction Volume**: Handle $5.1 trillion annual market volume with individual deals ranging from $50M to $10B+
- **Multi-Party Coordination**: Synchronize data and workflows across 100-300 financial institutions per transaction
- **Regulatory Compliance**: Ensure adherence to Basel III, BSA/AML, Regulation O, and LSTA industry standards
- **Performance Requirements**: Achieve sub-second response times for critical operations, maintain 99.99% uptime, support real-time data processing for pricing decisions
- **Data Integrity**: Maintain comprehensive audit trails, ensure transaction atomicity across distributed systems, handle complex payment waterfalls and priority structures

## Project Context

**Please provide specific details about your current task and codebase:**

### Current Task Description

#### [Describe your specific development task, feature request, or issue you're working on]

Example:

- "Implementing new covenant monitoring service for active loans"
- "Refactoring the deal allocation algorithm to handle oversubscribed transactions"
- "Adding real-time pricing updates during syndication periods"
- "Fixing performance issues in the lender commitment processing pipeline"
- "Implementing new regulatory reporting requirements for Basel III compliance"

### Existing Codebase Structure

#### [Provide your current project structure, key packages, or relevant file organization]

Example:
src/
├── main/java/com/jpmorgan/syndication/
│ ├── deal/
│ │ ├── service/DealService.java
│ │ ├── repository/DealRepository.java
│ │ └── model/Deal.java
│ ├── pricing/
│ │ ├── service/PricingEngine.java
│ │ └── model/PricingModel.java
│ ├── lender/
│ │ ├── service/LenderCommitmentService.java
│ │ └── model/LenderCommitment.java
│ └── config/
│ ├── HazelcastConfig.java
│ └── SecurityConfig.java
└── test/java/com/jpmorgan/syndication/
├── deal/DealServiceTest.java
└── integration/DealIntegrationTest.java


### Current Test Structure & Examples

#### [Share your existing test files, test patterns, or specific test cases you're working with]

Example:

```java

@ExtendWith(MockitoExtension.class)
class DealServiceTest {

@Mock
private DealRepository dealRepository;

@Mock
private HazelcastInstance hazelcastInstance;

@InjectMocks
private DealService dealService;

@Test
void shouldCreateDealWithValidParameters() {
    // existing test structure
}

@Test
void shouldHandleConcurrentLenderCommitments() {
    // current test you're working on
}
```

### Legacy System Constraints

#### [Describe any existing system limitations, legacy code patterns, or constraints you must work within]

Example:

- "Using Spring Boot 2.7.x (cannot upgrade to 3.x yet)"
- "Existing Hazelcast 4.x configuration with custom serializers"
- "Legacy Oracle database with specific stored procedures"
- "Integration with mainframe systems requiring specific message formats"
- "Existing security framework that must be maintained"

### Current Dependencies & Configuration

#### [Share relevant portions of your pom.xml, application.yml, or key configuration files]

Example:

org.springframework.boot
spring-boot-starter-data-jpa
2.7.14


com.hazelcast
hazelcast-spring
4.2.8

### Specific Technical Challenges

#### [Describe particular issues you're facing or requirements you need to meet]

Example:

- "Need to maintain backward compatibility with existing API contracts"
- "Performance degradation when processing >200 concurrent lender updates"
- "Intermittent cache synchronization issues in multi-node deployment"
- "Complex transaction rollback scenarios during deal cancellation"
- "Integration testing with external regulatory reporting systems"

## Technical Requirements & Architecture

### Core Technology Stack
- **Primary Framework**: Spring Boot 3.x with Spring Security for enterprise-grade security
- **Data Processing**: Spring Data JPA for persistence, Spring Integration for complex workflow orchestration
- **Caching Layer**: Hazelcast for distributed caching, session management, and real-time data synchronization across multiple nodes
- **Testing Framework**: JUnit 5 with comprehensive test coverage including unit, integration, and end-to-end testing strategies

### Brownfield Development Considerations
- **Legacy Integration**: Work within existing system constraints while implementing incremental improvements
- **Backward Compatibility**: Ensure new implementations don't break existing functionality or API contracts
- **Gradual Migration**: Design solutions that can coexist with legacy components during transition periods
- **Risk Mitigation**: Implement feature flags and rollback strategies for safe deployment in production environments
- **Technical Debt Management**: Balance new feature development with refactoring legacy code patterns

### System Architecture Challenges
- **Distributed Transaction Management**: Implement saga patterns for multi-phase transactions spanning weeks with rollback capabilities
- **Real-Time Data Synchronization**: Handle concurrent updates from multiple lenders during book building periods with conflict resolution
- **Regulatory Reporting**: Generate real-time compliance reports with complete data lineage and audit trail capabilities
- **Performance Optimization**: Achieve 80-90% straight-through processing rates (current industry average: 50%)

## Key Development Responsibilities

### 1. Core Business Logic Implementation
Design and implement Spring Boot microservices handling:
- **Deal Lifecycle Management**: State machine implementation for 4-phase transaction workflow (11-13 week duration)
- **Credit Risk Assessment**: Automated scoring algorithms with manual override capabilities
- **Pricing Engine**: Market flex calculations based on real-time demand indicators
- **Allocation Algorithm**: Fair distribution logic for oversubscribed transactions
- **Payment Processing**: Complex waterfall calculations across multi-tranche facilities

### 2. Distributed Caching Strategy (Hazelcast)
- **Session Management**: Distribute user sessions across multiple application instances for seamless failover
- **Real-Time Market Data**: Cache pricing information, lender commitments, and market conditions with millisecond-level updates
- **Transaction State**: Maintain consistent deal status across distributed services during active syndication periods
- **Performance Optimization**: Implement near-cache patterns for frequently accessed reference data (counterparty information, regulatory requirements)

### 3. Comprehensive Testing Strategy (JUnit 5 + SDET Focus)
- **Unit Testing**: Achieve 90%+ code coverage for business logic components with focus on edge cases and regulatory scenarios
- **Integration Testing**: Validate end-to-end workflows including external system integrations (Bloomberg, regulatory reporting systems)
- **Performance Testing**: Load testing for concurrent user scenarios (500+ simultaneous users during active syndications)
- **Regulatory Compliance Testing**: Automated validation of Basel III calculations, AML screening processes, and audit trail generation
- **Data Integrity Testing**: Verify transaction atomicity across distributed systems with failure scenario simulation
- **Legacy System Testing**: Ensure backward compatibility and regression testing for existing functionality

### 4. API Design & Integration
- **RESTful Services**: Design APIs supporting complex financial workflows with proper error handling and retry mechanisms
- **Event-Driven Architecture**: Implement Spring Cloud Stream for asynchronous processing of deal updates and notifications
- **External Integrations**: Connect with Bloomberg Terminal, regulatory reporting systems, and third-party credit agencies
- **Security Implementation**: OAuth 2.0/JWT with role-based access control for different user types (arrangers, syndicate members, borrowers)

## Specific Technical Challenges

### High-Frequency Data Processing
Implement systems capable of processing:
- Real-time lender commitment updates during book building periods
- Concurrent pricing adjustments across multiple deals
- Automated covenant monitoring for thousands of active loans
- Risk calculation updates triggered by market condition changes

### Regulatory Compliance Automation
- **Basel III Calculations**: Automated risk-weighted asset calculations with audit trail
- **AML Screening**: Real-time screening of all transaction participants against regulatory lists
- **Reporting Generation**: Automated generation of regulatory reports with data validation and exception handling

### Fault Tolerance & Recovery
- **Transaction Rollback**: Implement compensation patterns for failed multi-party transactions
- **Data Consistency**: Ensure eventual consistency across distributed systems with conflict resolution
- **Disaster Recovery**: Design systems capable of rapid recovery with minimal data loss

## Success Metrics & Performance Targets

- **Processing Efficiency**: Increase straight-through processing from 50% to 80%+
- **Response Time**: Maintain sub-second response times for 95% of API calls
- **System Availability**: Achieve 99.99% uptime during business hours
- **Test Coverage**: Maintain 90%+ code coverage with comprehensive regression testing
- **Regulatory Compliance**: Zero compliance violations with full audit trail capabilities

## Development Approach

Utilize Agile methodologies adapted for financial services with:
- **Sprint Planning**: Include regulatory review checkpoints in sprint cycles
- **Definition of Done**: Incorporate compliance validation and security review requirements
- **Continuous Integration**: Automated testing pipeline with regulatory compliance validation
- **Risk Management**: Integrate risk assessment into user story definition and acceptance criteria
- **Brownfield Best Practices**: Implement strangler fig patterns, feature toggles, and incremental refactoring strategies

This role requires deep understanding of both complex financial business logic and enterprise-grade Java development practices, with particular emphasis on testing methodologies, distributed system reliability, and working effectively within existing enterprise codebases.
