# Design Document: Evaluation-Driven Adaptive Learning System

## Overview

The Evaluation-Driven Adaptive Learning System is a serverless, AWS-native platform that implements institutional-scale adaptive learning through continuous assessment and intelligent curriculum restructuring. The system leverages Amazon Bedrock's multi-agent architecture to orchestrate four specialized agents that collaborate to deliver personalized learning experiences grounded in institutional curricula.

The architecture follows a layered design pattern:
- **Presentation Layer**: React/Next.js web interfaces for students, faculty, and admins
- **API Layer**: Amazon API Gateway providing RESTful endpoints
- **Orchestration Layer**: AWS Lambda functions coordinating multi-agent workflows
- **AI Layer**: Amazon Bedrock Agents (Curriculum, Tutor, Evaluation, Progress Coach) with RAG capabilities
- **Knowledge Layer**: Amazon Bedrock Knowledge Base with S3-backed curriculum documents
- **Data Layer**: DynamoDB for student profiles, progress, and assessments
- **Monitoring Layer**: Amazon CloudWatch for observability

The system implements the core learning cycle: **Assess → Plan → Learn → Reassess → Adapt**, where each phase is driven by measured outcomes rather than time-based progression.

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Presentation Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Student    │  │   Faculty    │  │    Admin     │         │
│  │  Dashboard   │  │  Dashboard   │  │   Console    │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└────────────────────────┬────────────────────────────────────────┘
                         │ HTTPS
┌────────────────────────▼────────────────────────────────────────┐
│                   Amazon API Gateway                            │
│              (REST APIs + Authentication)                       │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────────┐
│                  AWS Lambda Functions                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  Diagnostic  │  │   Roadmap    │  │  Assessment  │         │
│  │   Handler    │  │   Handler    │  │   Handler    │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Progress   │  │   Content    │  │    Admin     │         │
│  │   Handler    │  │   Handler    │  │   Handler    │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────────┐
│              Amazon Bedrock Multi-Agent Layer                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  Curriculum  │  │    Tutor     │  │  Evaluation  │         │
│  │    Agent     │  │    Agent     │  │    Agent     │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│  ┌──────────────┐                                              │
│  │Progress Coach│  Models: Nova Pro, Nova Lite                │
│  │    Agent     │  Embeddings: Titan Text V2                  │
│  └──────────────┘                                              │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────────┐
│           Amazon Bedrock Knowledge Base (RAG)                   │
│  ┌──────────────────────────────────────────────────┐          │
│  │  Vector Store (Curriculum Embeddings)            │          │
│  │  - Syllabus PDFs                                 │          │
│  │  - Lecture Notes                                 │          │
│  │  - Institutional Policies                        │          │
│  └──────────────────────────────────────────────────┘          │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────────┐
│                    Amazon S3                                    │
│              (Curriculum Document Storage)                      │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    Amazon DynamoDB                              │
│  Tables: Students, Roadmaps, Assessments, AdaptationLogs       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   Amazon CloudWatch                             │
│         (Logs, Metrics, Alarms, Dashboards)                     │
└─────────────────────────────────────────────────────────────────┘
```

### Layered Architecture Breakdown

#### 1. Presentation Layer
- **Technology**: React with Next.js for server-side rendering
- **Components**:
  - Student Dashboard: Roadmap visualization, assessment interface, progress charts
  - Faculty Dashboard: Cohort analytics, at-risk student alerts, individual student drill-down
  - Admin Console: Curriculum upload, configuration management, system monitoring
- **Communication**: REST API calls to API Gateway with JWT authentication
- **State Management**: React Context API or Redux for client-side state

#### 2. API Layer
- **Service**: Amazon API Gateway (REST API)
- **Endpoints**:
  - `/v1/auth/*` - Authentication and authorization
  - `/v1/students/*` - Student profile and progress operations
  - `/v1/roadmaps/*` - Roadmap retrieval and updates
  - `/v1/assessments/*` - Assessment submission and results
  - `/v1/faculty/*` - Cohort monitoring and analytics
  - `/v1/admin/*` - System configuration and curriculum management
- **Security**: API keys, JWT token validation, IAM authorization
- **Rate Limiting**: Token bucket algorithm (100 requests/minute per user)

#### 3. Orchestration Layer
- **Service**: AWS Lambda (Node.js or Python runtime)
- **Functions**:
  - `DiagnosticHandler`: Orchestrates initial assessment flow
  - `RoadmapHandler`: Coordinates Curriculum_Agent for roadmap generation
  - `AssessmentHandler`: Manages formative/summative assessment lifecycle
  - `ProgressHandler`: Triggers Progress_Coach_Agent for adaptation analysis
  - `ContentHandler`: Routes content requests to Tutor_Agent with RAG
  - `AdminHandler`: Manages curriculum uploads and Knowledge Base indexing
- **Execution**: Event-driven, stateless, auto-scaling
- **Timeout**: 30 seconds for agent invocations, 5 seconds for database operations

#### 4. AI Layer (Amazon Bedrock Multi-Agent)

**Agent Architecture**:
Each agent is implemented as an Amazon Bedrock Agent with:
- **Foundation Model**: Nova Pro (complex reasoning) or Nova Lite (simple tasks)
- **Action Groups**: Lambda functions the agent can invoke
- **Knowledge Base**: Shared access to curriculum vector store
- **Guardrails**: Content filtering and response validation

**Agent Specifications**:

**Curriculum_Agent**:
- **Model**: Amazon Nova Pro (complex reasoning for dependency analysis)
- **Responsibilities**: Generate personalized roadmaps, restructure learning sequences, identify prerequisite gaps
- **Action Groups**:
  - `analyze_diagnostic_results`: Parse assessment scores and identify weak concepts
  - `retrieve_curriculum_structure`: Query Knowledge Base for topic dependencies
  - `generate_roadmap`: Create directed acyclic graph of learning sequence
  - `adapt_roadmap`: Insert remedial modules based on Progress_Coach analysis
- **Input**: Student diagnostic scores, current roadmap state, adaptation triggers
- **Output**: Structured roadmap JSON with topics, dependencies, estimated durations

**Tutor_Agent**:
- **Model**: Amazon Nova Lite (explanations and summaries)
- **Responsibilities**: Provide syllabus-grounded explanations, answer student questions, generate learning content
- **Action Groups**:
  - `retrieve_curriculum_content`: RAG query to Knowledge Base for relevant documents
  - `generate_explanation`: Create topic explanation grounded in retrieved content
  - `cite_sources`: Reference curriculum documents in responses
- **Input**: Student question, topic identifier, learning context
- **Output**: Explanation text with curriculum citations

**Evaluation_Agent**:
- **Model**: Amazon Nova Lite (question generation)
- **Responsibilities**: Generate diagnostic, formative, and summative assessments
- **Action Groups**:
  - `retrieve_learning_objectives`: Query Knowledge Base for topic objectives
  - `generate_questions`: Create diverse question types across Bloom's taxonomy
  - `validate_questions`: Check question quality and curriculum alignment
  - `score_assessment`: Evaluate student responses and provide feedback
- **Input**: Topic identifier, assessment type, difficulty level, question count
- **Output**: Structured assessment JSON with questions, options, correct answers, explanations

**Progress_Coach_Agent**:
- **Model**: Amazon Nova Pro (pattern analysis and decision-making)
- **Responsibilities**: Analyze learning trends, detect struggling patterns, trigger roadmap adaptations
- **Action Groups**:
  - `analyze_assessment_history`: Identify patterns in formative assessment scores
  - `detect_concept_gaps`: Determine root cause concepts for failures
  - `calculate_adaptation_trigger`: Decide if roadmap restructuring is needed
  - `generate_intervention_recommendations`: Suggest faculty interventions for at-risk students
- **Input**: Student assessment history, roadmap progress, time-on-task metrics
- **Output**: Adaptation decision with triggering reasons, recommended roadmap changes

#### 5. Knowledge Layer

**Amazon Bedrock Knowledge Base**:
- **Vector Store**: OpenSearch Serverless or Amazon Aurora with pgvector
- **Embedding Model**: Amazon Titan Text Embeddings V2
- **Document Types**: Curriculum PDFs, syllabus documents, lecture notes, institutional policies
- **Indexing**: Automatic chunking (500-1000 tokens), metadata tagging (course, topic, institution)
- **Retrieval**: Semantic search with configurable top-k (default k=5)

**RAG Pipeline**:
1. **Query Formulation**: Agent constructs semantic query from student request
2. **Embedding Generation**: Query embedded using Titan Text Embeddings V2
3. **Vector Search**: Similarity search in Knowledge Base returns top-k documents
4. **Context Augmentation**: Retrieved documents injected into agent prompt
5. **Grounded Generation**: Agent generates response citing retrieved sources
6. **Citation Validation**: System verifies response references actual retrieved content

**S3 Document Storage**:
- **Bucket Structure**: `s3://adaptive-learning-kb/{institution_id}/{course_id}/{document_id}.pdf`
- **Metadata**: Institution, course, topic tags, upload timestamp, version
- **Lifecycle**: Automatic archival of old versions after 90 days
- **Access Control**: IAM policies restricting access to AdminHandler Lambda

#### 6. Data Layer

**Amazon DynamoDB Tables**:

**Students Table**:
- **Partition Key**: `student_id` (UUID)
- **Attributes**: `name`, `email`, `institution_id`, `cohort_id`, `enrollment_date`, `baseline_scores` (map)
- **GSI**: `institution_id-enrollment_date-index` for cohort queries

**Roadmaps Table**:
- **Partition Key**: `student_id`
- **Sort Key**: `version` (timestamp)
- **Attributes**: `roadmap_json` (structured DAG), `current_topic`, `completion_percentage`, `estimated_completion_date`
- **Purpose**: Store current and historical roadmaps for adaptation tracking

**Assessments Table**:
- **Partition Key**: `assessment_id` (UUID)
- **Sort Key**: `student_id`
- **Attributes**: `type` (diagnostic/formative/summative), `topic_id`, `questions_json`, `responses_json`, `score`, `timestamp`, `feedback`
- **GSI**: `student_id-timestamp-index` for student history queries
- **GSI**: `topic_id-type-index` for topic-level analytics

**AdaptationLogs Table**:
- **Partition Key**: `student_id`
- **Sort Key**: `timestamp`
- **Attributes**: `trigger_event`, `old_roadmap_version`, `new_roadmap_version`, `modified_topics`, `reason`
- **Purpose**: Audit trail for roadmap adaptations

**InstitutionalConfig Table**:
- **Partition Key**: `institution_id`
- **Attributes**: `mastery_threshold_formative`, `mastery_threshold_summative`, `adaptation_sensitivity`, `max_assessment_attempts`, `cohort_definitions`

#### 7. Monitoring Layer

**Amazon CloudWatch**:
- **Logs**: All Lambda function logs, Bedrock agent invocation logs, API Gateway access logs
- **Metrics**:
  - Lambda: Invocation count, duration, error rate, concurrent executions
  - Bedrock: Agent invocation count, latency, token usage, cost
  - DynamoDB: Read/write capacity, throttled requests, latency
  - API Gateway: Request count, 4xx/5xx errors, latency
- **Alarms**:
  - Lambda error rate > 5%
  - Bedrock agent latency > 15 seconds
  - DynamoDB throttling events
  - API Gateway 5xx error rate > 1%
- **Dashboards**: Real-time operational metrics, cost tracking, usage patterns

## Components and Interfaces

### Component: Diagnostic Assessment Flow

**Purpose**: Conduct initial assessment to establish student baseline and identify conceptual gaps.

**Workflow**:
1. Student initiates diagnostic assessment via dashboard
2. API Gateway routes request to `DiagnosticHandler` Lambda
3. `DiagnosticHandler` invokes `Evaluation_Agent` with parameters:
   - Assessment type: "diagnostic"
   - Topics: All prerequisite concepts for course
   - Question count: 20
   - Difficulty distribution: 40% basic, 40% intermediate, 20% advanced
4. `Evaluation_Agent` queries Knowledge Base for learning objectives
5. `Evaluation_Agent` generates questions using Nova Lite
6. `DiagnosticHandler` stores assessment in DynamoDB and returns to student
7. Student completes assessment and submits responses
8. `DiagnosticHandler` invokes `Evaluation_Agent` to score responses
9. `DiagnosticHandler` stores results and triggers roadmap generation

**Interface**:
```
POST /v1/assessments/diagnostic/generate
Request: { student_id, course_id }
Response: { assessment_id, questions: [...] }

POST /v1/assessments/diagnostic/submit
Request: { assessment_id, student_id, responses: [...] }
Response: { score, topic_scores: {...}, identified_gaps: [...] }
```

### Component: Adaptive Roadmap Generation

**Purpose**: Create personalized learning sequence based on diagnostic results.

**Workflow**:
1. `RoadmapHandler` Lambda triggered by diagnostic completion event
2. `RoadmapHandler` retrieves diagnostic results from DynamoDB
3. `RoadmapHandler` invokes `Curriculum_Agent` with:
   - Student baseline scores per topic
   - Course curriculum structure from Knowledge Base
   - Institutional pacing guidelines
4. `Curriculum_Agent` queries Knowledge Base for topic dependencies
5. `Curriculum_Agent` uses Nova Pro to analyze gaps and generate roadmap DAG
6. Roadmap includes:
   - Topics ordered by prerequisite dependencies
   - Remedial modules for weak areas
   - Estimated time per topic
   - Mastery checkpoints
7. `RoadmapHandler` stores roadmap in DynamoDB (version 1)
8. `RoadmapHandler` returns roadmap to student dashboard

**Interface**:
```
POST /v1/roadmaps/generate
Request: { student_id, diagnostic_results }
Response: { roadmap_id, version, topics: [...], dependencies: [...], estimated_duration }

GET /v1/roadmaps/{student_id}/current
Response: { roadmap_id, version, current_topic, progress_percentage, topics: [...] }
```

### Component: Syllabus-Grounded Content Delivery

**Purpose**: Provide learning explanations aligned with institutional curriculum using RAG.

**Workflow**:
1. Student requests topic explanation via dashboard
2. API Gateway routes to `ContentHandler` Lambda
3. `ContentHandler` invokes `Tutor_Agent` with:
   - Topic identifier
   - Student's current knowledge level
   - Specific question (if any)
4. `Tutor_Agent` formulates semantic query for Knowledge Base
5. Knowledge Base performs vector search, returns top-5 curriculum documents
6. `Tutor_Agent` generates explanation using Nova Lite with retrieved context
7. `Tutor_Agent` includes citations to source documents
8. `ContentHandler` returns explanation to student with document references

**RAG Implementation**:
```python
# Pseudocode for RAG pipeline in Tutor_Agent
def generate_explanation(topic_id, student_question):
    # Step 1: Formulate query
    query = f"Explain {topic_id} concept: {student_question}"
    
    # Step 2: Retrieve from Knowledge Base
    retrieved_docs = knowledge_base.retrieve(
        query=query,
        top_k=5,
        filters={"topic": topic_id}
    )
    
    # Step 3: Construct prompt with context
    context = "\n".join([doc.content for doc in retrieved_docs])
    prompt = f"""
    Based on the following curriculum content:
    {context}
    
    Explain {topic_id} to answer: {student_question}
    Cite specific sections from the curriculum.
    """
    
    # Step 4: Generate grounded response
    response = bedrock_agent.invoke(prompt, model="nova-lite")
    
    # Step 5: Validate citations
    citations = extract_citations(response)
    validated_citations = validate_against_retrieved(citations, retrieved_docs)
    
    return {
        "explanation": response,
        "sources": validated_citations
    }
```

**Interface**:
```
POST /v1/content/explain
Request: { student_id, topic_id, question }
Response: { explanation, sources: [{doc_id, title, excerpt}] }
```

### Component: Formative Assessment and Gap Detection

**Purpose**: Continuously assess understanding after each learning module and detect gaps.

**Workflow**:
1. Student completes learning module and triggers formative assessment
2. `AssessmentHandler` invokes `Evaluation_Agent` to generate 5-10 questions
3. Student submits responses
4. `Evaluation_Agent` scores assessment and provides immediate feedback
5. `AssessmentHandler` stores results in DynamoDB
6. If score < 70%, `AssessmentHandler` triggers `Progress_Coach_Agent`
7. `Progress_Coach_Agent` analyzes if gap is isolated or pattern
8. If pattern detected (3+ consecutive low scores), trigger roadmap adaptation

**Interface**:
```
POST /v1/assessments/formative/generate
Request: { student_id, topic_id }
Response: { assessment_id, questions: [...] }

POST /v1/assessments/formative/submit
Request: { assessment_id, student_id, responses: [...] }
Response: { score, feedback: [...], gap_detected: boolean }
```

### Component: Evaluation-Driven Roadmap Adaptation

**Purpose**: Restructure learning path when persistent gaps are detected.

**Workflow**:
1. `Progress_Coach_Agent` detects adaptation trigger (consecutive low scores)
2. `Progress_Coach_Agent` analyzes assessment history to identify root cause concepts
3. `Progress_Coach_Agent` invokes `Curriculum_Agent` with adaptation request:
   - Current roadmap state
   - Identified weak concepts
   - Prerequisite dependencies
4. `Curriculum_Agent` uses Nova Pro to restructure roadmap:
   - Insert remedial modules for weak prerequisites
   - Adjust pacing for struggling topics
   - Reorder dependent topics
5. `Curriculum_Agent` generates new roadmap version
6. `RoadmapHandler` stores new version in DynamoDB
7. `RoadmapHandler` logs adaptation event in AdaptationLogs table
8. System notifies student of roadmap change with explanation

**Adaptation Logic**:
```python
# Pseudocode for adaptation decision
def should_adapt_roadmap(student_id):
    recent_assessments = get_recent_formative_assessments(student_id, limit=5)
    
    # Check for consecutive failures
    consecutive_failures = 0
    for assessment in recent_assessments:
        if assessment.score < 0.70:
            consecutive_failures += 1
        else:
            consecutive_failures = 0
    
    # Trigger adaptation if 3+ consecutive failures
    if consecutive_failures >= 3:
        return True, "consecutive_failures"
    
    # Check for topic-specific struggles
    topic_scores = aggregate_scores_by_topic(recent_assessments)
    struggling_topics = [t for t, s in topic_scores.items() if s < 0.60]
    
    if len(struggling_topics) >= 2:
        return True, "multiple_weak_topics"
    
    return False, None

def adapt_roadmap(student_id, trigger_reason):
    # Get current roadmap
    current_roadmap = get_current_roadmap(student_id)
    
    # Analyze gaps
    gap_analysis = progress_coach_agent.analyze_gaps(student_id)
    root_causes = gap_analysis.root_cause_concepts
    
    # Request roadmap restructuring
    new_roadmap = curriculum_agent.restructure(
        current_roadmap=current_roadmap,
        weak_concepts=root_causes,
        adaptation_reason=trigger_reason
    )
    
    # Store new version
    save_roadmap_version(student_id, new_roadmap)
    log_adaptation(student_id, trigger_reason, current_roadmap, new_roadmap)
    
    # Notify student
    notify_student(student_id, "roadmap_adapted", new_roadmap)
```

**Interface**:
```
POST /v1/roadmaps/adapt
Request: { student_id, trigger_reason, gap_analysis }
Response: { new_roadmap_id, version, modified_topics: [...], reason }

GET /v1/roadmaps/{student_id}/history
Response: { versions: [{version, timestamp, trigger, changes}] }
```

### Component: Faculty Cohort Monitoring

**Purpose**: Provide faculty with cohort-level analytics and at-risk student identification.

**Workflow**:
1. Faculty accesses dashboard
2. API Gateway routes to `FacultyHandler` Lambda
3. `FacultyHandler` queries DynamoDB for cohort students
4. `FacultyHandler` aggregates metrics:
   - Average progress percentage
   - Average assessment scores
   - Students with adaptation events (at-risk)
   - Common weak topics across cohort
5. `FacultyHandler` invokes `Progress_Coach_Agent` for at-risk analysis
6. Dashboard displays:
   - Cohort summary statistics
   - At-risk student list with intervention recommendations
   - Topic-level heatmap showing cohort strengths/weaknesses
7. Faculty can drill down to individual student roadmaps and assessment history

**Interface**:
```
GET /v1/faculty/cohorts/{cohort_id}/summary
Response: { 
    total_students, 
    avg_progress, 
    avg_score, 
    at_risk_count,
    common_weak_topics: [...]
}

GET /v1/faculty/cohorts/{cohort_id}/at-risk
Response: { 
    students: [{
        student_id, 
        name, 
        progress, 
        recent_scores, 
        adaptation_count,
        intervention_recommendation
    }]
}

GET /v1/faculty/students/{student_id}/details
Response: {
    profile,
    current_roadmap,
    assessment_history,
    adaptation_log
}
```

### Component: Admin Curriculum Management

**Purpose**: Enable admins to upload and manage curriculum documents for Knowledge Base.

**Workflow**:
1. Admin uploads PDF via admin console
2. API Gateway routes to `AdminHandler` Lambda
3. `AdminHandler` uploads PDF to S3 with metadata tags
4. `AdminHandler` triggers Knowledge Base sync job
5. Bedrock Knowledge Base:
   - Extracts text from PDF
   - Chunks content (500-1000 tokens)
   - Generates embeddings using Titan Text Embeddings V2
   - Indexes in vector store
6. `AdminHandler` polls sync job status
7. Once complete, `AdminHandler` updates InstitutionalConfig with new document reference
8. Admin console displays indexing success confirmation

**Interface**:
```
POST /v1/admin/curriculum/upload
Request: { institution_id, course_id, file: <PDF>, metadata: {...} }
Response: { document_id, s3_uri, indexing_job_id }

GET /v1/admin/curriculum/indexing-status/{job_id}
Response: { status: "in_progress" | "completed" | "failed", indexed_chunks }

GET /v1/admin/curriculum/documents
Response: { documents: [{doc_id, title, course, upload_date, status}] }

DELETE /v1/admin/curriculum/documents/{doc_id}
Response: { success: boolean }
```

## Data Models

### Student Profile
```json
{
  "student_id": "uuid",
  "name": "string",
  "email": "string",
  "institution_id": "string",
  "cohort_id": "string",
  "enrollment_date": "ISO8601",
  "baseline_scores": {
    "topic_id_1": 0.75,
    "topic_id_2": 0.60
  },
  "current_roadmap_version": "timestamp",
  "total_assessments_completed": "integer",
  "avg_assessment_score": "float"
}
```

### Roadmap Structure
```json
{
  "roadmap_id": "uuid",
  "student_id": "uuid",
  "version": "timestamp",
  "topics": [
    {
      "topic_id": "string",
      "title": "string",
      "prerequisites": ["topic_id_1", "topic_id_2"],
      "estimated_duration_hours": "integer",
      "status": "not_started" | "in_progress" | "completed",
      "mastery_score": "float | null"
    }
  ],
  "current_topic_index": "integer",
  "completion_percentage": "float",
  "estimated_completion_date": "ISO8601"
}
```

### Assessment Structure
```json
{
  "assessment_id": "uuid",
  "student_id": "uuid",
  "type": "diagnostic" | "formative" | "summative",
  "topic_id": "string",
  "timestamp": "ISO8601",
  "questions": [
    {
      "question_id": "uuid",
      "text": "string",
      "type": "multiple_choice" | "code_analysis" | "conceptual",
      "options": ["string"],
      "correct_answer": "string",
      "bloom_level": "remember" | "understand" | "apply" | "analyze",
      "difficulty": "basic" | "intermediate" | "advanced"
    }
  ],
  "responses": [
    {
      "question_id": "uuid",
      "student_answer": "string",
      "is_correct": "boolean",
      "time_spent_seconds": "integer"
    }
  ],
  "score": "float",
  "feedback": [
    {
      "question_id": "uuid",
      "explanation": "string",
      "misconception_addressed": "string"
    }
  ]
}
```

### Adaptation Log Entry
```json
{
  "student_id": "uuid",
  "timestamp": "ISO8601",
  "trigger_event": "consecutive_failures" | "multiple_weak_topics" | "time_threshold",
  "old_roadmap_version": "timestamp",
  "new_roadmap_version": "timestamp",
  "modified_topics": [
    {
      "action": "inserted" | "reordered" | "removed",
      "topic_id": "string",
      "reason": "string"
    }
  ],
  "gap_analysis": {
    "weak_concepts": ["string"],
    "root_causes": ["string"],
    "recommended_remediation": ["string"]
  }
}
```

### Institutional Configuration
```json
{
  "institution_id": "string",
  "name": "string",
  "mastery_threshold_formative": 0.70,
  "mastery_threshold_summative": 0.80,
  "adaptation_sensitivity": "low" | "medium" | "high",
  "max_assessment_attempts": 3,
  "reassessment_cooldown_hours": 24,
  "cohort_definitions": [
    {
      "cohort_id": "string",
      "name": "string",
      "faculty_ids": ["string"],
      "student_ids": ["string"]
    }
  ],
  "curriculum_documents": [
    {
      "doc_id": "string",
      "s3_uri": "string",
      "course_id": "string",
      "indexed_date": "ISO8601"
    }
  ]
}
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

Before defining correctness properties, I need to analyze the acceptance criteria from the requirements document to determine which are testable as properties, examples, or edge cases.

### Property Reflection

After analyzing all acceptance criteria, I've identified the following redundancies:
- Requirements 3.2, 3.3, and 3.4 all test citation/grounding - can be combined into one property
- Requirements 17.2 and 17.4 both test question diversity - redundant
- Several properties test data persistence which can be consolidated

The following properties provide unique validation value after eliminating redundancy:

### Correctness Properties

Property 1: Diagnostic Assessment Bloom's Taxonomy Coverage
*For any* generated diagnostic assessment, the questions SHALL span all four specified Bloom's taxonomy levels (Remember, Understand, Apply, Analyze)
**Validates: Requirements 1.2**

Property 2: Diagnostic Assessment Size Constraints
*For any* generated diagnostic assessment, the question count SHALL be between 15 and 25 inclusive, and SHALL cover all prerequisite topics for the course
**Validates: Requirements 1.4**

Property 3: Diagnostic Results Persistence
*For any* computed diagnostic result, querying the student profile SHALL return the stored baseline scores and identified gaps
**Validates: Requirements 1.5**

Property 4: Topic Accuracy Score Computation
*For any* submitted diagnostic assessment, the analysis results SHALL include accuracy scores for each individual topic
**Validates: Requirements 1.3**

Property 5: Roadmap Addresses Identified Gaps
*For any* completed diagnostic with identified gaps, the generated roadmap SHALL include topics that address those specific gaps
**Validates: Requirements 2.1**

Property 6: Roadmap Prerequisite Ordering
*For any* generated roadmap, if topic A is a prerequisite for topic B, then A SHALL appear before B in the learning sequence (topological ordering)
**Validates: Requirements 2.2**

Property 7: Roadmap Acyclicity
*For any* generated roadmap, the topic dependency graph SHALL be acyclic (no circular dependencies)
**Validates: Requirements 2.4**

Property 8: Roadmap Time Estimation Completeness
*For any* generated roadmap, every topic SHALL have an estimated duration in hours
**Validates: Requirements 2.5**

Property 9: Explanation Citation Requirement
*For any* generated topic explanation, the response SHALL include citations to source documents retrieved from the Knowledge Base
**Validates: Requirements 3.2, 3.3, 3.4**

Property 10: Formative Assessment Size Constraints
*For any* generated formative assessment, the question count SHALL be between 5 and 10 inclusive
**Validates: Requirements 4.1**

Property 11: Low Score Remediation Flagging
*For any* formative assessment with score below 70%, the topic SHALL be flagged as requiring remediation
**Validates: Requirements 4.3**

Property 12: Formative Assessment Feedback Provision
*For any* submitted formative assessment, the response SHALL include feedback explaining correct answers
**Validates: Requirements 4.4**

Property 13: Consecutive Failure Adaptation Trigger
*For any* student with 3 or more consecutive formative assessments scoring below 70%, the Progress_Coach_Agent SHALL trigger roadmap adaptation
**Validates: Requirements 4.5**

Property 14: Gap Analysis on Low Scores
*For any* formative assessment scoring below 70%, the Progress_Coach_Agent SHALL perform gap pattern analysis
**Validates: Requirements 5.1**

Property 15: Root Cause Identification
*For any* gap analysis, the output SHALL include root cause concepts identified through prerequisite dependency analysis
**Validates: Requirements 5.2**

Property 16: Remedial Module Insertion
*For any* triggered adaptation, the restructured roadmap SHALL include remedial modules positioned before topics that depend on the weak concepts
**Validates: Requirements 5.3**

Property 17: Adaptation Notification
*For any* roadmap adaptation event, a notification SHALL be sent to the student with explanation of changes
**Validates: Requirements 5.4**

Property 18: Adaptation Audit Logging
*For any* roadmap adaptation, an entry SHALL exist in the AdaptationLogs table containing triggering assessment scores and modified learning sequence
**Validates: Requirements 5.5**

Property 19: Summative Assessment Completeness
*For any* generated summative assessment for a module, the questions SHALL cover all topics within that module
**Validates: Requirements 6.1**

Property 20: Summative Assessment Size and Difficulty
*For any* generated summative assessment, the question count SHALL be between 20 and 30 inclusive, and SHALL include multiple difficulty levels
**Validates: Requirements 6.2**

Property 21: Module Mastery Unlocking
*For any* summative assessment with score exceeding 80%, the module SHALL be marked as mastered and dependent modules SHALL be unlocked
**Validates: Requirements 6.3**

Property 22: Reassessment Cooldown Enforcement
*For any* summative assessment with score below 80%, reassessment SHALL be blocked for 24 hours
**Validates: Requirements 6.4**

Property 23: Assessment Attempt Limiting
*For any* module, after 3 summative assessment attempts, no additional summative assessments SHALL be generated
**Validates: Requirements 6.5**

Property 24: Dashboard Metrics Completeness
*For any* student dashboard request, the response SHALL include average assessment scores and time spent per topic
**Validates: Requirements 7.2**

Property 25: Progress Calculation Accuracy
*For any* progress query, the response SHALL include completion percentage calculated as (completed topics / total topics) and estimated time to completion
**Validates: Requirements 7.3**

Property 26: Mastery Level Tracking
*For any* progress query, the response SHALL include mastery levels for all topics in the student's roadmap
**Validates: Requirements 7.4**

Property 27: Analytics Historical Data Provision
*For any* detailed analytics request, the response SHALL include topic-wise performance breakdowns and historical score trends
**Validates: Requirements 7.5**

Property 28: Faculty Dashboard Cohort Statistics
*For any* faculty cohort dashboard request, the response SHALL include average progress, average assessment scores, and completion rates for the cohort
**Validates: Requirements 8.1**

Property 29: At-Risk Student Identification
*For any* cohort containing students with 3+ consecutive low scores or no progress for 7+ days, those students SHALL appear in the at-risk list
**Validates: Requirements 8.2**

Property 30: Cohort Common Gaps Analysis
*For any* faculty cohort analytics request, the response SHALL identify topics where 30% or more of students scored below 70%
**Validates: Requirements 8.4**

Property 31: Faculty Data Export Format
*For any* faculty export request, the response SHALL be valid CSV format containing student IDs, progress percentages, and average scores
**Validates: Requirements 8.5**

Property 32: Knowledge Base Indexing Trigger
*For any* curriculum document uploaded to S3, an indexing job SHALL be automatically triggered in the Bedrock Knowledge Base
**Validates: Requirements 10.2**

Property 33: Document Version Preservation
*For any* curriculum document update, the previous version SHALL remain accessible and the new version SHALL be created with incremented version number
**Validates: Requirements 10.3**

Property 34: Indexing Completion Notification
*For any* completed Knowledge Base indexing job, an admin notification SHALL be sent
**Validates: Requirements 10.4**

Property 35: Role-Based Access Enforcement
*For any* API request, the response SHALL only include data that the authenticated user's role has permission to access
**Validates: Requirements 12.2**

Property 36: Student Data Isolation
*For any* student API request, the response SHALL only include that student's own profile, assessments, and roadmap data
**Validates: Requirements 12.3**

Property 37: Faculty Cohort Restriction
*For any* faculty API request for student data, the response SHALL only include students within the faculty member's assigned cohorts
**Validates: Requirements 12.4**

Property 38: Cache Hit Reduction
*For any* curriculum content requested multiple times within 5 minutes, the second and subsequent requests SHALL be served from cache without Knowledge Base queries
**Validates: Requirements 13.4**

Property 39: Lambda Invocation Logging
*For any* Lambda function invocation, an entry SHALL exist in CloudWatch Logs containing invocation timestamp, duration, and status
**Validates: Requirements 14.1**

Property 40: Agent Failure Error Logging
*For any* failed Bedrock agent invocation, the CloudWatch log entry SHALL contain agent type, input parameters, and failure reason
**Validates: Requirements 14.2**

Property 41: Agent Metrics Recording
*For any* Bedrock agent invocation, CloudWatch metrics SHALL record invocation count and latency
**Validates: Requirements 14.4**

Property 42: System Error Notification
*For any* system error (Lambda failure, agent timeout, database error), a notification SHALL be sent to admin SNS topic
**Validates: Requirements 14.5**

Property 43: Data Persistence Round-Trip
*For any* data write operation (student profile, roadmap, assessment), immediately querying the database SHALL return the written data
**Validates: Requirements 15.1, 15.2**

Property 44: Adaptation Audit Trail Completeness
*For any* roadmap adaptation, the audit log entry SHALL contain timestamp, triggering event, old roadmap version, new roadmap version, and modified topics
**Validates: Requirements 15.3**

Property 45: Database Write Retry Logic
*For any* failed database write operation, the system SHALL retry with exponential backoff (1s, 2s, 4s) and notify admins after 3 failures
**Validates: Requirements 15.5**

Property 46: API Request Validation
*For any* API request with invalid payload, the response SHALL have 400 status code with validation error details
**Validates: Requirements 16.3**

Property 47: API Rate Limit Enforcement
*For any* user exceeding 100 requests per minute, subsequent requests SHALL be rejected with 429 status code
**Validates: Requirements 16.4**

Property 48: Assessment Question Format Diversity
*For any* generated assessment with 10+ questions, at least two different question formats SHALL be present (multiple-choice, code analysis, conceptual)
**Validates: Requirements 17.1**

Property 49: Question Uniqueness Across Attempts
*For any* two consecutive assessments for the same student and topic, the question sets SHALL have at most 30% overlap
**Validates: Requirements 17.2, 17.4**

Property 50: Configuration Update Isolation
*For any* institutional configuration update, students with in-progress assessments SHALL continue using the old configuration, while new sessions SHALL use the new configuration
**Validates: Requirements 18.3**

Property 51: Configuration Versioning
*For any* institutional configuration update, a new version entry SHALL be created in DynamoDB preserving the previous version
**Validates: Requirements 18.5**


## Error Handling

### Error Categories and Strategies

#### 1. Agent Invocation Failures
**Scenarios**: Bedrock agent timeout, model throttling, invalid agent response

**Handling Strategy**:
- Implement retry logic with exponential backoff (1s, 2s, 4s)
- After 3 failed attempts, return fallback response to user
- Log detailed error information to CloudWatch
- Send admin notification for persistent failures
- Fallback responses:
  - Curriculum_Agent failure: Return default roadmap based on curriculum structure
  - Tutor_Agent failure: Return cached explanation or suggest manual resources
  - Evaluation_Agent failure: Return questions from pre-generated question bank
  - Progress_Coach_Agent failure: Defer adaptation to next assessment

**Implementation**:
```python
def invoke_agent_with_retry(agent_name, input_data, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = bedrock_client.invoke_agent(
                agentId=agent_ids[agent_name],
                inputText=input_data
            )
            return response
        except (ThrottlingException, TimeoutException) as e:
            if attempt < max_retries - 1:
                wait_time = 2 ** attempt
                time.sleep(wait_time)
                continue
            else:
                log_error(agent_name, input_data, str(e))
                send_admin_notification(f"Agent {agent_name} failed after {max_retries} attempts")
                return get_fallback_response(agent_name, input_data)
```

#### 2. Knowledge Base Retrieval Failures
**Scenarios**: No relevant documents found, vector search timeout, embedding generation failure

**Handling Strategy**:
- If no documents retrieved (empty result set), notify student that curriculum content is unavailable
- Suggest alternative resources or manual faculty consultation
- Log retrieval failures for admin review
- For embedding failures, retry with smaller chunk size
- Implement circuit breaker pattern to prevent cascading failures

**Implementation**:
```python
def retrieve_with_fallback(query, topic_id):
    try:
        results = knowledge_base.retrieve(query, filters={"topic": topic_id})
        if len(results) == 0:
            return {
                "status": "no_content",
                "message": "Curriculum content not available for this topic",
                "suggestion": "Please consult your instructor or course materials"
            }
        return {"status": "success", "documents": results}
    except Exception as e:
        log_error("knowledge_base_retrieval", query, str(e))
        return {
            "status": "error",
            "message": "Unable to retrieve curriculum content",
            "suggestion": "Please try again later"
        }
```

#### 3. Database Operation Failures
**Scenarios**: DynamoDB throttling, connection timeout, transaction conflict

**Handling Strategy**:
- Implement exponential backoff retry for throttling errors
- Use DynamoDB transactions for operations requiring consistency (assessment submission + score update)
- For read failures, return cached data if available
- For write failures after retries, queue operation for later processing
- Send admin alerts for persistent database issues

**Implementation**:
```python
def write_with_retry(table_name, item, max_retries=3):
    for attempt in range(max_retries):
        try:
            dynamodb.put_item(TableName=table_name, Item=item)
            return {"status": "success"}
        except ProvisionedThroughputExceededException:
            if attempt < max_retries - 1:
                wait_time = (2 ** attempt) + random.uniform(0, 1)
                time.sleep(wait_time)
                continue
            else:
                # Queue for later processing
                sqs.send_message(QueueUrl=retry_queue_url, MessageBody=json.dumps(item))
                send_admin_notification(f"Database write failed for {table_name}")
                return {"status": "queued"}
```

#### 4. API Gateway Errors
**Scenarios**: Invalid request payload, authentication failure, rate limit exceeded

**Handling Strategy**:
- Return appropriate HTTP status codes with descriptive error messages
- 400 Bad Request: Include validation errors in response body
- 401 Unauthorized: Prompt user to re-authenticate
- 429 Too Many Requests: Include Retry-After header
- 500 Internal Server Error: Log error details, return generic message to user

**Error Response Format**:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request payload",
    "details": [
      {
        "field": "student_id",
        "issue": "Required field missing"
      }
    ]
  }
}
```

#### 5. Assessment Scoring Errors
**Scenarios**: Invalid student response format, scoring logic failure, timeout

**Handling Strategy**:
- Validate response format before scoring
- For invalid responses, mark as incorrect with feedback explaining format requirements
- If scoring fails, flag assessment for manual review
- Store raw responses for audit trail
- Notify student of scoring issues and allow resubmission

#### 6. Roadmap Adaptation Errors
**Scenarios**: Circular dependency detected, invalid topic reference, adaptation logic failure

**Handling Strategy**:
- Validate roadmap structure before persisting (DAG check)
- If validation fails, retain current roadmap and log error
- Notify student that adaptation was deferred
- Send admin alert for manual review
- Implement roadmap rollback capability

**Validation**:
```python
def validate_roadmap(roadmap):
    # Check for cycles using DFS
    def has_cycle(node, visited, rec_stack):
        visited.add(node)
        rec_stack.add(node)
        for neighbor in roadmap.dependencies.get(node, []):
            if neighbor not in visited:
                if has_cycle(neighbor, visited, rec_stack):
                    return True
            elif neighbor in rec_stack:
                return True
        rec_stack.remove(node)
        return False
    
    visited = set()
    for topic in roadmap.topics:
        if topic.id not in visited:
            if has_cycle(topic.id, visited, set()):
                return False, "Circular dependency detected"
    
    return True, "Valid"
```

## Testing Strategy

### Dual Testing Approach

The system requires both unit testing and property-based testing to ensure comprehensive coverage:

**Unit Tests**: Validate specific examples, edge cases, and error conditions
**Property Tests**: Verify universal properties across all inputs

Together, these approaches provide complementary coverage where unit tests catch concrete bugs and property tests verify general correctness.

### Property-Based Testing Configuration

**Library Selection**:
- **Python**: Hypothesis (recommended for Lambda functions)
- **TypeScript/JavaScript**: fast-check (for frontend components)

**Configuration Requirements**:
- Minimum 100 iterations per property test (due to randomization)
- Each property test MUST reference its design document property
- Tag format: `# Feature: adaptive-learning-system, Property {number}: {property_text}`

**Example Property Test (Python with Hypothesis)**:
```python
from hypothesis import given, strategies as st
import pytest

# Feature: adaptive-learning-system, Property 2: Diagnostic Assessment Size Constraints
@given(st.integers(min_value=1, max_value=100))
def test_diagnostic_assessment_size_constraints(num_topics):
    """
    Property 2: For any generated diagnostic assessment, the question count 
    SHALL be between 15 and 25 inclusive, and SHALL cover all prerequisite topics.
    """
    # Generate diagnostic assessment
    assessment = evaluation_agent.generate_diagnostic(
        course_id="CS101",
        num_topics=num_topics
    )
    
    # Verify question count constraint
    assert 15 <= len(assessment.questions) <= 25, \
        f"Expected 15-25 questions, got {len(assessment.questions)}"
    
    # Verify all topics covered
    covered_topics = {q.topic_id for q in assessment.questions}
    expected_topics = get_prerequisite_topics("CS101")
    assert covered_topics == expected_topics, \
        f"Missing topics: {expected_topics - covered_topics}"

# Feature: adaptive-learning-system, Property 7: Roadmap Acyclicity
@given(st.lists(st.text(min_size=1, max_size=10), min_size=3, max_size=20))
def test_roadmap_acyclicity(topic_names):
    """
    Property 7: For any generated roadmap, the topic dependency graph 
    SHALL be acyclic (no circular dependencies).
    """
    # Generate roadmap with random topics
    diagnostic_results = generate_random_diagnostic_results(topic_names)
    roadmap = curriculum_agent.generate_roadmap(
        student_id="test_student",
        diagnostic_results=diagnostic_results
    )
    
    # Verify no cycles using DFS
    assert not has_cycle(roadmap.dependencies), \
        "Roadmap contains circular dependencies"

# Feature: adaptive-learning-system, Property 43: Data Persistence Round-Trip
@given(
    st.text(min_size=1, max_size=50),
    st.emails(),
    st.floats(min_value=0.0, max_value=1.0)
)
def test_data_persistence_round_trip(name, email, baseline_score):
    """
    Property 43: For any data write operation, immediately querying 
    the database SHALL return the written data.
    """
    # Create student profile
    student_id = str(uuid.uuid4())
    profile = {
        "student_id": student_id,
        "name": name,
        "email": email,
        "baseline_scores": {"topic1": baseline_score}
    }
    
    # Write to database
    db.write_student_profile(profile)
    
    # Immediately read back
    retrieved = db.get_student_profile(student_id)
    
    # Verify round-trip
    assert retrieved["name"] == name
    assert retrieved["email"] == email
    assert retrieved["baseline_scores"]["topic1"] == baseline_score
```

### Unit Testing Strategy

**Focus Areas**:
1. **Specific Examples**: Test concrete scenarios from requirements
2. **Edge Cases**: Empty inputs, boundary values, special characters
3. **Error Conditions**: Invalid inputs, timeout scenarios, failure modes
4. **Integration Points**: Agent communication, database transactions, API contracts

**Example Unit Tests**:
```python
def test_first_access_presents_diagnostic():
    """Validates Requirement 1.1: First access presents diagnostic"""
    student_id = create_new_student()
    response = api_client.get(f"/v1/students/{student_id}/dashboard")
    assert response.status_code == 200
    assert "diagnostic_assessment" in response.json()
    assert response.json()["diagnostic_assessment"]["status"] == "pending"

def test_empty_curriculum_documents_notification():
    """Validates Requirement 3.5: Notify when curriculum unavailable"""
    response = api_client.post("/v1/content/explain", json={
        "student_id": "test_student",
        "topic_id": "nonexistent_topic",
        "question": "Explain this topic"
    })
    assert response.status_code == 200
    assert "no_content" in response.json()["status"]
    assert "suggestion" in response.json()

def test_rate_limit_enforcement():
    """Validates Property 47: Rate limit enforcement"""
    student_id = "test_student"
    # Make 101 requests rapidly
    for i in range(101):
        response = api_client.get(f"/v1/students/{student_id}/dashboard")
        if i < 100:
            assert response.status_code == 200
        else:
            assert response.status_code == 429
            assert "Retry-After" in response.headers
```

### Integration Testing

**Scenarios**:
1. **End-to-End Learning Flow**: Diagnostic → Roadmap → Learning → Formative → Adaptation
2. **Multi-Agent Orchestration**: Verify agents communicate correctly
3. **RAG Pipeline**: Verify retrieval and grounding work together
4. **Database Transactions**: Verify consistency across related writes

**Example Integration Test**:
```python
def test_complete_adaptation_flow():
    """Integration test for evaluation-driven adaptation"""
    # Create student and complete diagnostic
    student_id = create_student_and_diagnostic()
    
    # Get initial roadmap
    roadmap_v1 = get_current_roadmap(student_id)
    
    # Simulate 3 consecutive low-scoring formative assessments
    for i in range(3):
        assessment = generate_formative_assessment(student_id, "topic1")
        submit_low_score_responses(assessment.id, score=0.50)
    
    # Verify adaptation triggered
    roadmap_v2 = get_current_roadmap(student_id)
    assert roadmap_v2.version > roadmap_v1.version
    
    # Verify remedial modules inserted
    assert any(t.type == "remedial" for t in roadmap_v2.topics)
    
    # Verify adaptation logged
    logs = get_adaptation_logs(student_id)
    assert len(logs) > 0
    assert logs[0].trigger_event == "consecutive_failures"
```

### Test Coverage Goals

- **Unit Test Coverage**: 80% code coverage for Lambda functions
- **Property Test Coverage**: All 51 correctness properties implemented
- **Integration Test Coverage**: All critical user flows (diagnostic, adaptation, assessment)
- **API Test Coverage**: All endpoints with valid/invalid inputs

### Continuous Testing

- Run unit tests on every code commit
- Run property tests (100 iterations) on pull requests
- Run integration tests on staging environment before production deployment
- Monitor property test failures in production using canary deployments

## Security Model

### Authentication and Authorization

**Authentication**:
- Students, faculty, and admins authenticate via institutional SSO (SAML 2.0 or OAuth 2.0)
- API Gateway validates JWT tokens issued by identity provider
- Tokens include user ID, role, and institution ID claims
- Token expiration: 1 hour (refresh tokens valid for 7 days)

**Authorization**:
- IAM roles for service-to-service communication (Lambda → Bedrock, Lambda → DynamoDB)
- Role-based access control (RBAC) enforced at API Gateway and Lambda layers
- Permission matrix:

| Resource | Student | Faculty | Admin |
|----------|---------|---------|-------|
| Own profile | Read/Write | Read | Read/Write |
| Other student profiles | None | Read (cohort only) | Read/Write |
| Roadmaps | Own only | Read (cohort only) | Read/Write |
| Assessments | Own only | Read (cohort only) | Read/Write |
| Cohort analytics | None | Read (assigned cohorts) | Read |
| Curriculum upload | None | None | Write |
| System config | None | None | Read/Write |

### Data Protection

**Encryption at Rest**:
- DynamoDB tables encrypted using AWS-managed keys (AES-256)
- S3 buckets encrypted using SSE-S3 or SSE-KMS
- Bedrock Knowledge Base vector store encrypted

**Encryption in Transit**:
- All API communication uses TLS 1.2 or higher
- Certificate pinning for mobile apps (future)
- Internal service communication within VPC uses TLS

**Data Minimization**:
- Store only necessary student data (no sensitive personal information beyond email)
- Anonymize data for analytics and research purposes
- Implement data retention policies (delete inactive accounts after 2 years)

### API Security

**Rate Limiting**:
- Token bucket algorithm: 100 requests per minute per user
- Burst allowance: 20 requests
- Rate limits enforced at API Gateway level

**Input Validation**:
- Validate all request payloads against JSON schemas
- Sanitize inputs to prevent injection attacks
- Reject requests with unexpected fields

**CORS Configuration**:
- Whitelist only institutional domains
- Restrict allowed methods (GET, POST, PUT, DELETE)
- Include credentials in CORS policy for authenticated requests

### Monitoring and Auditing

**Audit Logging**:
- Log all authentication attempts (success and failure)
- Log all data access and modifications with user ID and timestamp
- Log all administrative actions (config changes, curriculum uploads)
- Store audit logs in CloudWatch Logs with 90-day retention

**Security Monitoring**:
- CloudWatch alarms for suspicious patterns (repeated auth failures, unusual access patterns)
- AWS GuardDuty for threat detection
- Regular security audits and penetration testing

## Cost Optimization Strategy

### Model Selection Strategy

**Amazon Nova Lite** (Low cost, fast inference):
- Formative assessment generation (high frequency)
- Topic summaries and simple explanations
- Question validation
- Estimated cost: $0.0006 per 1K input tokens, $0.0024 per 1K output tokens

**Amazon Nova Pro** (Higher cost, advanced reasoning):
- Roadmap generation and restructuring (complex dependency analysis)
- Gap analysis and root cause identification
- Adaptation decision-making
- Estimated cost: $0.008 per 1K input tokens, $0.032 per 1K output tokens

**Cost Projection** (500 students per institution):
- Diagnostic assessments: 500 students × 1 diagnostic × $0.05 = $25/month
- Formative assessments: 500 students × 20 assessments × $0.02 = $200/month
- Roadmap generation: 500 students × 2 roadmaps × $0.15 = $150/month
- Content explanations: 500 students × 50 queries × $0.01 = $250/month
- **Total Bedrock cost**: ~$625/month

### Caching Strategy

**Curriculum Content Caching**:
- Cache Knowledge Base retrieval results in ElastiCache (Redis)
- TTL: 5 minutes for frequently accessed content
- Cache key: hash(query + topic_id)
- Estimated cache hit rate: 60% → 40% reduction in retrieval costs

**Assessment Caching**:
- Cache generated assessments for 24 hours
- Reuse questions across students (with randomization)
- Reduces Evaluation_Agent invocations by 50%

**Roadmap Template Caching**:
- Cache default roadmap structures per course
- Personalize from cached template rather than generating from scratch
- Reduces Curriculum_Agent invocations by 30%

### Database Optimization

**DynamoDB Capacity Mode**:
- Use on-demand billing for unpredictable workloads (initial deployment)
- Switch to provisioned capacity with auto-scaling after usage patterns stabilize
- Estimated cost: $50-100/month for 500 students

**Query Optimization**:
- Use Global Secondary Indexes (GSI) for common query patterns
- Batch read/write operations where possible
- Implement pagination for large result sets

### Lambda Optimization

**Memory Allocation**:
- Right-size Lambda memory based on profiling (128MB-1024MB)
- Higher memory = faster execution = lower cost (up to a point)
- Estimated cost: $30-50/month for all Lambda functions

**Cold Start Reduction**:
- Use provisioned concurrency for critical functions (DiagnosticHandler, AssessmentHandler)
- Keep functions warm during peak hours (8 AM - 10 PM)

### Total Cost Projection

**Per Institution (500 students)**:
- Bedrock (agents + embeddings): $625/month
- DynamoDB: $75/month
- Lambda: $40/month
- API Gateway: $20/month
- S3 + CloudWatch: $15/month
- ElastiCache (caching): $50/month
- **Total**: ~$825/month → **$1.65 per student per month**

**Cost Reduction Strategies**:
- Implement aggressive caching → 30% reduction
- Optimize model selection → 20% reduction
- Use reserved capacity for predictable workloads → 15% reduction
- **Optimized cost**: ~$450/month → **$0.90 per student per month**

## Scalability Strategy

### Horizontal Scaling

**Serverless Auto-Scaling**:
- Lambda functions scale automatically to handle concurrent requests
- API Gateway scales transparently
- DynamoDB on-demand capacity scales automatically

**Multi-Institution Deployment**:
- Single AWS account with logical isolation per institution
- Shared infrastructure (Lambda functions, Bedrock agents)
- Isolated data (DynamoDB partition key includes institution_id)
- Shared Knowledge Base with institution-specific document filtering

### Performance Optimization

**Latency Targets**:
- Dashboard loads: < 3 seconds
- Agent-generated content: < 10 seconds
- Assessment submission: < 2 seconds

**Optimization Techniques**:
- Use CloudFront CDN for static frontend assets
- Implement lazy loading for dashboard components
- Paginate large result sets (assessments, roadmap history)
- Use DynamoDB batch operations for bulk reads

### Load Testing

**Scenarios**:
- 500 concurrent students accessing dashboards
- 200 concurrent assessment submissions
- 50 concurrent roadmap adaptations

**Tools**:
- AWS Load Testing Solution or Locust
- Simulate realistic user behavior patterns
- Monitor CloudWatch metrics during load tests

### Disaster Recovery

**Backup Strategy**:
- DynamoDB point-in-time recovery enabled
- S3 versioning enabled for curriculum documents
- Daily snapshots of institutional configuration

**Recovery Objectives**:
- RTO (Recovery Time Objective): 4 hours
- RPO (Recovery Point Objective): 1 hour

## Future Enhancements

### Phase 2 Features

1. **Code Execution Environment**: Integrate AWS Lambda or AWS Cloud9 for students to write and test code
2. **Collaborative Learning**: Peer discussion forums, study groups, collaborative problem-solving
3. **Mobile Applications**: Native iOS and Android apps for on-the-go learning
4. **Multi-Language Support**: Hindi, Tamil, Telugu, Bengali for regional accessibility
5. **Advanced Analytics**: Predictive modeling for dropout risk, career path recommendations

### Phase 3 Features

1. **Live Tutoring**: Video conferencing integration for faculty office hours
2. **Gamification**: Badges, leaderboards, achievement tracking
3. **Industry Integration**: Real-world projects from partner companies
4. **Certification**: Verified skill certificates upon module completion
5. **AI-Powered Code Review**: Automated feedback on student code submissions

### Research Opportunities

1. **Adaptive Difficulty**: Dynamic question difficulty adjustment based on student performance
2. **Learning Style Detection**: Identify visual/auditory/kinesthetic preferences and adapt content
3. **Emotional Intelligence**: Detect frustration or disengagement and adjust pacing
4. **Curriculum Optimization**: Use cohort data to improve curriculum structure and content

## Deployment Architecture

### Infrastructure as Code

**Tool**: AWS CloudFormation or Terraform

**Resources**:
- API Gateway REST API
- Lambda functions (6 handlers)
- DynamoDB tables (5 tables)
- S3 buckets (curriculum storage)
- Bedrock agents (4 agents)
- Bedrock Knowledge Base
- CloudWatch log groups and alarms
- IAM roles and policies
- ElastiCache cluster (Redis)

### CI/CD Pipeline

**Stages**:
1. **Source**: GitHub repository
2. **Build**: AWS CodeBuild (run tests, package Lambda functions)
3. **Test**: Run unit tests, property tests, integration tests
4. **Deploy to Staging**: CloudFormation stack update
5. **Smoke Tests**: Verify critical endpoints
6. **Deploy to Production**: Blue-green deployment with canary analysis
7. **Monitor**: CloudWatch dashboards and alarms

### Environment Strategy

- **Development**: Individual developer environments (local + AWS)
- **Staging**: Shared environment for integration testing
- **Production**: Multi-region deployment (primary: ap-south-1 Mumbai, backup: ap-southeast-1 Singapore)

## Conclusion

The Evaluation-Driven Adaptive Learning System represents a comprehensive, AWS-native solution for institutional-scale programming education. By leveraging Amazon Bedrock's multi-agent architecture, RAG-based curriculum grounding, and continuous assessment-driven adaptation, the system addresses critical challenges in Indian engineering colleges while maintaining cost efficiency and scalability.

The design emphasizes:
- **Evaluation-first approach**: Personalization driven by measured understanding
- **Institutional deployment**: Built for 500+ concurrent students per college
- **AWS-native architecture**: Serverless, scalable, cost-optimized
- **Correctness properties**: 51 testable properties ensuring system reliability
- **Faculty empowerment**: Cohort monitoring and early intervention capabilities

This system is positioned to transform programming education outcomes in Tier-2 and Tier-3 institutions through intelligent, adaptive, and scalable learning infrastructure.
