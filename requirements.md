# Requirements Document: Evaluation-Driven Adaptive Learning System

## Introduction

The Evaluation-Driven Adaptive Learning System is an institutional infrastructure platform designed to address systemic challenges in programming education at Indian Tier-2 and Tier-3 engineering colleges. Unlike generic AI tutoring tools, this system implements a structured assessment-driven approach to personalize learning pathways based on measured conceptual understanding rather than behavioral patterns.

The system operates on the core principle: **Assess → Plan → Learn → Reassess → Adapt**. It leverages AWS-native services, particularly Amazon Bedrock's multi-agent architecture, to orchestrate curriculum planning, evaluation generation, adaptive tutoring, and progress coaching at institutional scale.

### Problem Statement

Programming education in Indian engineering colleges faces critical challenges:

- **Static Curriculum Structures**: Fixed syllabi fail to accommodate varying student preparedness levels
- **One-Size-Fits-All Pacing**: Uniform teaching pace leaves weak students behind and strong students unchallenged
- **Weak Formative Evaluation**: Limited continuous assessment results in late detection of conceptual gaps
- **Overloaded Faculty**: High student-to-teacher ratios prevent individualized attention
- **Poor Remediation**: Lack of structured intervention when students fall behind

These challenges result in high failure rates, weak foundational understanding, and graduates unprepared for industry demands.

### Objectives

1. Implement evaluation-driven adaptive learning that restructures curriculum based on diagnostic assessments
2. Provide institutional-scale personalized learning pathways for 500+ concurrent students per college
3. Enable early detection and remediation of conceptual gaps through continuous formative assessment
4. Reduce faculty workload through automated evaluation generation and progress monitoring
5. Deliver AWS-native, cost-efficient serverless architecture suitable for resource-constrained institutions

### Scope

**In-Scope:**
- Diagnostic assessment system for programming concepts
- Adaptive curriculum roadmap generation and restructuring
- Multi-agent orchestration using Amazon Bedrock
- Formative and summative evaluation generation
- Student progress tracking and analytics
- Faculty dashboard for cohort monitoring
- RAG-based syllabus-grounded content delivery
- AWS serverless infrastructure (Lambda, API Gateway, Bedrock, DynamoDB)

**Out-of-Scope:**
- Live video instruction or synchronous tutoring
- Code execution environments or IDEs
- Plagiarism detection systems
- Campus management system integration
- Mobile native applications (Phase 1)
- Multi-language support beyond English (Phase 1)
- Automated grading of subjective assignments

## Glossary

- **System**: The Evaluation-Driven Adaptive Learning System
- **Curriculum_Agent**: Bedrock agent responsible for generating structured learning roadmaps
- **Tutor_Agent**: Bedrock agent providing syllabus-grounded explanations and learning content
- **Evaluation_Agent**: Bedrock agent creating diagnostic tests and mastery assessments
- **Progress_Coach_Agent**: Bedrock agent analyzing learning trends and triggering roadmap adaptations
- **Student**: End-user learner enrolled in programming courses
- **Faculty**: Instructor monitoring cohort progress and intervening when needed
- **Admin**: System administrator managing institutional configuration
- **Diagnostic_Assessment**: Initial evaluation measuring baseline conceptual understanding
- **Formative_Assessment**: Continuous evaluation during learning to identify gaps
- **Summative_Assessment**: End-of-module mastery test
- **Adaptation_Event**: Triggered restructuring of learning roadmap based on assessment results
- **Knowledge_Base**: Amazon Bedrock Knowledge Base containing curriculum PDFs and institutional policies
- **RAG_Pipeline**: Retrieval-Augmented Generation system grounding agent responses in curriculum documents

## Requirements

### Requirement 1: Diagnostic Assessment System

**User Story:** As a student, I want to take an initial diagnostic assessment, so that the system can identify my current knowledge level and conceptual gaps.

#### Acceptance Criteria

1. WHEN a student first accesses the system, THE System SHALL present a diagnostic assessment covering prerequisite concepts
2. WHEN generating diagnostic questions, THE Evaluation_Agent SHALL create questions spanning Bloom's taxonomy levels (Remember, Understand, Apply, Analyze)
3. WHEN a student submits diagnostic responses, THE System SHALL analyze results to identify weak concept areas with accuracy scores per topic
4. THE Evaluation_Agent SHALL generate diagnostic assessments with 15-25 questions covering all prerequisite topics
5. WHEN diagnostic results are computed, THE System SHALL store baseline scores and identified gaps in the student profile

### Requirement 2: Adaptive Curriculum Roadmap Generation

**User Story:** As a student, I want a personalized learning roadmap based on my diagnostic results, so that I can focus on concepts I need to strengthen.

#### Acceptance Criteria

1. WHEN diagnostic assessment is completed, THE Curriculum_Agent SHALL generate a personalized learning roadmap addressing identified gaps
2. THE Curriculum_Agent SHALL structure roadmaps with prerequisite dependencies ensuring foundational concepts precede advanced topics
3. WHEN generating roadmaps, THE Curriculum_Agent SHALL retrieve relevant curriculum content from the Knowledge_Base using RAG
4. THE System SHALL present roadmaps as directed acyclic graphs showing topic dependencies and learning sequence
5. WHEN a roadmap is generated, THE System SHALL estimate time requirements for each topic based on complexity and student baseline

### Requirement 3: Syllabus-Grounded Content Delivery

**User Story:** As a student, I want learning explanations aligned with my institution's curriculum, so that I learn content relevant to my exams and program requirements.

#### Acceptance Criteria

1. WHEN a student requests topic explanation, THE Tutor_Agent SHALL retrieve relevant curriculum documents from the Knowledge_Base
2. THE Tutor_Agent SHALL ground all explanations in retrieved curriculum content using RAG to ensure syllabus alignment
3. WHEN generating explanations, THE Tutor_Agent SHALL cite source documents from the Knowledge_Base
4. THE System SHALL prevent hallucinated content by requiring Tutor_Agent responses to reference retrieved curriculum materials
5. WHEN curriculum documents are unavailable for a topic, THE System SHALL notify the student and suggest alternative resources

### Requirement 4: Formative Assessment and Gap Detection

**User Story:** As a student, I want continuous quizzes after learning each topic, so that I can verify my understanding before moving forward.

#### Acceptance Criteria

1. WHEN a student completes a learning module, THE Evaluation_Agent SHALL generate a formative assessment with 5-10 questions
2. THE Evaluation_Agent SHALL create questions testing conceptual understanding rather than rote memorization
3. WHEN formative assessment results fall below 70% accuracy, THE System SHALL flag the topic as requiring remediation
4. THE System SHALL provide immediate feedback on formative assessments explaining correct answers and common misconceptions
5. WHEN multiple formative assessments show consistent gaps, THE Progress_Coach_Agent SHALL trigger roadmap adaptation

### Requirement 5: Evaluation-Driven Roadmap Adaptation

**User Story:** As a student, I want my learning path to automatically adjust when I struggle with concepts, so that I receive targeted remediation.

#### Acceptance Criteria

1. WHEN formative assessment scores fall below mastery threshold (70%), THE Progress_Coach_Agent SHALL analyze the gap pattern
2. THE Progress_Coach_Agent SHALL identify root cause concepts requiring remediation by analyzing prerequisite dependencies
3. WHEN adaptation is triggered, THE Curriculum_Agent SHALL restructure the roadmap inserting remedial modules before blocked topics
4. THE System SHALL notify students when roadmap adaptations occur with explanations of why changes were made
5. WHEN roadmap is adapted, THE System SHALL log the adaptation event with triggering assessment scores and modified learning sequence

### Requirement 6: Summative Mastery Assessment

**User Story:** As a student, I want end-of-module mastery tests, so that I can demonstrate comprehensive understanding before progressing.

#### Acceptance Criteria

1. WHEN a student completes all topics in a module, THE Evaluation_Agent SHALL generate a summative assessment covering all module concepts
2. THE Evaluation_Agent SHALL create summative assessments with 20-30 questions spanning multiple difficulty levels
3. WHEN summative assessment score exceeds 80%, THE System SHALL mark the module as mastered and unlock dependent modules
4. WHEN summative assessment score falls below 80%, THE System SHALL require module review and allow reassessment after 24 hours
5. THE System SHALL limit summative assessment attempts to 3 per module to prevent gaming

### Requirement 7: Progress Tracking and Analytics

**User Story:** As a student, I want to view my learning progress and performance trends, so that I can understand my strengths and areas needing improvement.

#### Acceptance Criteria

1. THE System SHALL display student dashboard showing completed topics, current roadmap position, and upcoming assessments
2. WHEN a student views their dashboard, THE System SHALL present performance metrics including average assessment scores and time spent per topic
3. THE System SHALL visualize progress as percentage completion of personalized roadmap with estimated time to module completion
4. THE System SHALL track and display concept mastery levels across all topics using color-coded indicators (weak, developing, proficient, mastered)
5. WHEN students request detailed analytics, THE System SHALL provide topic-wise performance breakdowns and historical score trends

### Requirement 8: Faculty Cohort Monitoring Dashboard

**User Story:** As faculty, I want to monitor cohort-wide progress and identify struggling students, so that I can provide timely interventions.

#### Acceptance Criteria

1. THE System SHALL provide faculty dashboard displaying cohort-level statistics including average progress, assessment scores, and completion rates
2. WHEN faculty access the dashboard, THE System SHALL highlight students flagged as at-risk based on consecutive low assessment scores or stalled progress
3. THE System SHALL allow faculty to filter and view individual student roadmaps, assessment history, and adaptation events
4. WHEN faculty request cohort analytics, THE System SHALL generate reports showing common conceptual gaps across multiple students
5. THE System SHALL enable faculty to export student progress data in CSV format for institutional reporting

### Requirement 9: Multi-Agent Orchestration

**User Story:** As a system architect, I want coordinated multi-agent workflows using Amazon Bedrock, so that curriculum planning, tutoring, evaluation, and coaching operate cohesively.

#### Acceptance Criteria

1. THE System SHALL implement four specialized Bedrock agents (Curriculum_Agent, Tutor_Agent, Evaluation_Agent, Progress_Coach_Agent) with distinct responsibilities
2. WHEN an adaptation event occurs, THE Progress_Coach_Agent SHALL invoke the Curriculum_Agent to restructure roadmaps
3. WHEN generating evaluations, THE Evaluation_Agent SHALL query the Knowledge_Base to ensure questions align with curriculum content
4. THE System SHALL route student requests to appropriate agents based on request type (explanation → Tutor_Agent, assessment → Evaluation_Agent)
5. WHEN agents require curriculum context, THE System SHALL execute RAG queries against the Knowledge_Base and provide retrieved documents to agents

### Requirement 10: Knowledge Base Management

**User Story:** As an admin, I want to upload and manage curriculum documents, so that the system grounds all content in institutional syllabi.

#### Acceptance Criteria

1. THE System SHALL allow admins to upload curriculum PDFs to Amazon S3 designated for the Knowledge_Base
2. WHEN curriculum documents are uploaded, THE System SHALL automatically index them in the Bedrock Knowledge_Base using Titan Text Embeddings V2
3. THE System SHALL support versioning of curriculum documents allowing admins to update syllabi without losing historical content
4. WHEN documents are indexed, THE System SHALL validate successful embedding generation and notify admins of indexing completion
5. THE System SHALL allow admins to delete or archive outdated curriculum documents from the Knowledge_Base

### Requirement 11: Scalability and Performance

**User Story:** As an admin, I want the system to handle 500+ concurrent students per institution, so that all students receive responsive service during peak usage.

#### Acceptance Criteria

1. THE System SHALL utilize AWS Lambda for serverless compute ensuring automatic scaling based on concurrent request volume
2. WHEN concurrent student load increases, THE System SHALL scale Lambda functions horizontally without manual intervention
3. THE System SHALL respond to student requests within 3 seconds for dashboard loads and within 10 seconds for agent-generated content
4. THE System SHALL handle 1000 requests per minute during peak assessment periods without degradation
5. WHEN database queries execute, THE System SHALL use DynamoDB with on-demand capacity or provisioned capacity with auto-scaling

### Requirement 12: Security and Access Control

**User Story:** As an admin, I want role-based access control and data protection, so that student data remains confidential and system access is appropriately restricted.

#### Acceptance Criteria

1. THE System SHALL implement AWS IAM roles for service-to-service authentication between Lambda, Bedrock, S3, and DynamoDB
2. THE System SHALL enforce role-based access control with distinct permissions for Student, Faculty, and Admin roles
3. WHEN students authenticate, THE System SHALL restrict data access to only their own profiles, assessments, and roadmaps
4. WHEN faculty authenticate, THE System SHALL restrict data access to only students within their assigned cohorts
5. THE System SHALL encrypt student data at rest in DynamoDB and in transit using TLS 1.2 or higher

### Requirement 13: Cost Optimization

**User Story:** As an admin, I want cost-efficient infrastructure suitable for resource-constrained institutions, so that the system remains financially sustainable.

#### Acceptance Criteria

1. THE System SHALL use Amazon Nova Lite for high-frequency, low-complexity tasks (quiz generation, summaries) to minimize inference costs
2. THE System SHALL use Amazon Nova Pro only for complex reasoning tasks (roadmap restructuring, gap analysis) requiring advanced capabilities
3. WHEN generating embeddings, THE System SHALL batch curriculum documents to reduce Titan embedding API calls
4. THE System SHALL implement caching for frequently accessed curriculum content reducing redundant Knowledge_Base queries
5. THE System SHALL use DynamoDB on-demand billing for unpredictable workloads or provisioned capacity with auto-scaling for predictable patterns

### Requirement 14: Monitoring and Observability

**User Story:** As an admin, I want system monitoring and error tracking, so that I can identify and resolve issues proactively.

#### Acceptance Criteria

1. THE System SHALL log all Lambda function invocations, errors, and performance metrics to Amazon CloudWatch
2. WHEN agent invocations fail, THE System SHALL log error details including agent type, input parameters, and failure reason
3. THE System SHALL create CloudWatch alarms for critical metrics including Lambda error rates, API Gateway 5xx errors, and DynamoDB throttling
4. THE System SHALL track Bedrock agent invocation counts and latencies for cost and performance analysis
5. WHEN system errors occur, THE System SHALL send notifications to admin email or SNS topics for immediate attention

### Requirement 15: Data Persistence and State Management

**User Story:** As a student, I want my progress and assessment history saved reliably, so that I can resume learning across sessions without data loss.

#### Acceptance Criteria

1. THE System SHALL persist student profiles, roadmaps, assessment scores, and adaptation logs in DynamoDB or RDS
2. WHEN students complete assessments, THE System SHALL immediately write results to the database with transaction consistency
3. THE System SHALL maintain audit trails of all roadmap adaptations including timestamps, triggering events, and modified sequences
4. THE System SHALL implement database backups with point-in-time recovery to prevent data loss
5. WHEN database writes fail, THE System SHALL retry with exponential backoff and notify admins if persistent failures occur

### Requirement 16: API Design and Integration

**User Story:** As a frontend developer, I want well-defined REST APIs, so that I can build responsive user interfaces for students and faculty.

#### Acceptance Criteria

1. THE System SHALL expose REST APIs via Amazon API Gateway for all student, faculty, and admin operations
2. THE System SHALL implement API endpoints for authentication, roadmap retrieval, assessment submission, progress queries, and dashboard data
3. WHEN API requests are received, THE System SHALL validate request payloads and return appropriate HTTP status codes (200, 400, 401, 500)
4. THE System SHALL implement API rate limiting to prevent abuse and ensure fair resource allocation
5. THE System SHALL version APIs (e.g., /v1/students) to support backward compatibility during updates

### Requirement 17: Evaluation Quality and Diversity

**User Story:** As a student, I want varied and high-quality assessment questions, so that evaluations accurately measure my understanding.

#### Acceptance Criteria

1. THE Evaluation_Agent SHALL generate questions across multiple formats including multiple-choice, code analysis, and conceptual explanation prompts
2. WHEN generating assessments, THE Evaluation_Agent SHALL ensure question diversity avoiding repetitive patterns across attempts
3. THE Evaluation_Agent SHALL validate generated questions for clarity, correctness, and alignment with learning objectives
4. THE System SHALL maintain a question bank per topic preventing identical questions in consecutive assessments for the same student
5. WHEN assessment quality issues are detected, THE System SHALL flag questions for admin review and exclude them from future assessments

### Requirement 18: Institutional Configuration

**User Story:** As an admin, I want to configure institutional parameters, so that the system adapts to specific college requirements and policies.

#### Acceptance Criteria

1. THE System SHALL allow admins to configure mastery thresholds (default 70% formative, 80% summative) per institution
2. THE System SHALL support configuration of assessment attempt limits, reassessment cooldown periods, and roadmap adaptation sensitivity
3. WHEN admins update configuration, THE System SHALL apply changes to new student sessions without affecting in-progress assessments
4. THE System SHALL allow admins to define cohort structures mapping students to faculty for dashboard access control
5. THE System SHALL persist institutional configuration in DynamoDB with versioning to track policy changes over time

## User Personas

### Student Persona
- **Name:** Priya Sharma
- **Context:** Second-year Computer Science student at Tier-2 engineering college in Pune
- **Background:** Moderate programming aptitude, struggles with data structures, learns at slower pace than peers
- **Goals:** Master programming fundamentals, pass exams, build confidence
- **Pain Points:** Falls behind in lectures, lacks personalized attention, unclear where gaps exist
- **System Interaction:** Takes diagnostic, follows adaptive roadmap, completes formative assessments, reviews progress dashboard

### Faculty Persona
- **Name:** Dr. Rajesh Kumar
- **Context:** Assistant Professor teaching Data Structures to 120 students
- **Background:** Experienced educator, limited time for individual student mentoring
- **Goals:** Ensure cohort success, identify struggling students early, reduce remedial workload
- **Pain Points:** Cannot track 120 students individually, late detection of failures, manual assessment creation
- **System Interaction:** Monitors cohort dashboard, reviews at-risk student alerts, exports progress reports

### Admin Persona
- **Name:** Anjali Desai
- **Context:** IT Administrator managing learning systems for engineering college
- **Background:** Technical background, responsible for system uptime and cost management
- **Goals:** Maintain reliable system, control costs, ensure data security
- **Pain Points:** Budget constraints, need for scalable infrastructure, compliance requirements
- **System Interaction:** Uploads curriculum documents, configures institutional parameters, monitors CloudWatch metrics

## Non-Functional Requirements

### Scalability
- System SHALL support 500+ concurrent students per institution
- System SHALL scale to 50+ institutions (25,000+ students) without architectural changes
- Lambda functions SHALL auto-scale based on request volume

### Performance
- Dashboard loads SHALL complete within 3 seconds
- Agent-generated content SHALL return within 10 seconds
- Assessment submissions SHALL process within 2 seconds
- System SHALL handle 1000 requests per minute during peak periods

### Reliability
- System SHALL maintain 99.5% uptime during academic semesters
- Database SHALL implement automated backups with point-in-time recovery
- System SHALL gracefully handle agent failures with retry logic and fallback responses

### Security
- All data in transit SHALL use TLS 1.2 or higher
- All data at rest SHALL be encrypted using AWS-managed keys
- System SHALL implement role-based access control with IAM
- API endpoints SHALL require authentication tokens

### Cost Efficiency
- System SHALL optimize model selection (Nova Lite for simple tasks, Nova Pro for complex reasoning)
- System SHALL implement caching to reduce redundant API calls
- System SHALL use serverless architecture to eliminate idle compute costs
- Monthly operational cost SHALL remain under $500 per institution for 500 students

### Maintainability
- System SHALL use infrastructure-as-code (CloudFormation or Terraform)
- System SHALL implement comprehensive logging for debugging
- System SHALL version APIs for backward compatibility
- Code SHALL follow AWS Well-Architected Framework principles

## Constraints

1. **AWS-Native Architecture**: System MUST use AWS services exclusively (no third-party AI APIs)
2. **Serverless-First**: System MUST prioritize serverless services (Lambda, API Gateway, DynamoDB) over managed servers
3. **Bedrock Multi-Agent**: System MUST implement multi-agent orchestration using Amazon Bedrock Agents
4. **RAG-Based Content**: System MUST ground all content in curriculum documents using Bedrock Knowledge Base
5. **No Code Execution**: System SHALL NOT execute student-submitted code (security and scope constraint)
6. **English-Only (Phase 1)**: System SHALL support English language only in initial release
7. **Web-Only (Phase 1)**: System SHALL provide web interface only (no mobile apps in Phase 1)

## Assumptions

1. Institutions will provide curriculum documents in PDF format
2. Students have reliable internet access for web-based learning
3. Faculty will actively monitor dashboards and intervene for at-risk students
4. Institutions will handle student authentication (system integrates with existing auth)
5. Assessment questions generated by Evaluation_Agent will be reviewed periodically by faculty
6. Students will complete assessments honestly without external assistance
7. AWS Bedrock services (Nova Pro, Nova Lite, Titan Embeddings) will remain available and cost-stable

## Success Metrics

### Learning Outcomes
- 30% reduction in students with critical conceptual gaps by end of semester
- 25% improvement in average summative assessment scores compared to baseline
- 40% reduction in module failure rates

### System Adoption
- 80% of enrolled students complete diagnostic assessment within first week
- 70% of students actively engage with adaptive roadmaps (complete 60%+ of assigned topics)
- 90% of faculty access cohort dashboard at least weekly

### Operational Efficiency
- 50% reduction in faculty time spent on manual assessment creation
- 60% earlier detection of at-risk students (identified within 3 weeks vs. 8 weeks traditionally)
- System uptime of 99.5% during academic semesters

### Cost Efficiency
- Monthly operational cost under $500 per institution (500 students)
- 70% cost reduction compared to traditional LMS with third-party AI integrations

### Institutional Impact
- Deployment at 10+ Tier-2/Tier-3 colleges within 12 months
- 90% faculty satisfaction with cohort monitoring capabilities
- 85% student satisfaction with personalized learning experience
