---
# Provenance
source_type: example
parent_methodology: "[[advanced-context-engineering-guide]]"
date_created: 2025-09-13

# Document Status
doc_type: example_workflow
maturity: production_validated
implementation_status: realistic_scenario

# Relationships
parent_doc: "[[advanced-context-engineering-guide]]"
implements_methodology:
  - "[[advanced-context-engineering-guide]]"
  - "[[context-engineering-boundary]]"
  - "[[context-engineering-core]]"
related_concepts:
  - three-phase-workflow
  - spec-first-development
  - customer-support-system
  - human-in-the-loop
  - context-management

# Discoverability
primary_topic: "spec-driven AI development workflow example"
secondary_topics:
  - "8-week project timeline"
  - "research-plan-implement phases in practice"
  - "human review checkpoints"
  - "context window management"
  - "sub-agent utilization patterns"
applicable_to:
  - "new project implementation"
  - "AI system development"
  - "team workflow planning"
  - "methodology validation"

# Example Metadata
scenario_type: "full project implementation"
project_type: "AI-powered customer support system"
complexity_level: "medium-high"
duration: "8 weeks"
team_size: "1-2 developers"
real_world_basis: true

summary_short: >-
  Complete 8-week workflow example demonstrating spec-driven AI development methodology for building an AI customer support system with CRM integration, multi-channel support, and human escalation workflows.
---

# Advanced Context Engineering and Spec-Driven AI Development: Complete Workflow Example

## Quick Reference: The Three-Phase Workflow

This methodology follows a simple three-phase approach:

1. **Research Phase** (`/research_codebase`)
   - Understand what exists and how it works
   - Spawn parallel sub-agents to explore different areas
   - Document findings with file:line references
   - **Output**: Research document in `thoughts/shared/research/`
   - **Human Action**: Review and approve understanding

2. **Planning Phase** (`/create_plan`)
   - Design implementation with specific phases
   - Define automated and manual success criteria
   - Break work into incremental, testable changes
   - **Output**: Implementation plan in `thoughts/shared/plans/`
   - **Human Action**: Review and approve approach

3. **Implementation Phase** (`/implement_plan [path-to-plan]`)
   - Execute the approved plan phase by phase
   - Check off items as they're completed
   - Run success criteria after each phase
   - **Output**: Working code + updated plan with checkmarks
   - **Human Action**: Review at phase boundaries

4. **Validation Phase** (`/validate_plan`) *(optional but recommended)*
   - Verify implementation matches plan's success criteria
   - Run all automated checks and document manual testing
   - **Output**: Validation report confirming completion
   - **Human Action**: Review validation results

5. **Handoff Phase** (`/create_handoff` / `/resume_handoff`) *(when needed)*
   - **Create**: Document current state when switching contexts
   - **Resume**: Pick up work from a previous handoff
   - **Output**: Handoff document for seamless context recovery
   - **Human Action**: Use when pausing, transitioning, or resuming work

6. **Commit & PR Phase** (`/commit` and `/describe_pr`)
   - Create structured git commits with context
   - Generate comprehensive PR descriptions
   - **Output**: Clean git history + detailed PR documentation
   - **Human Action**: Review commits and PR before pushing

**Key Principle**: Research → Plan → Implement → Validate → Document → Ship. Each phase produces a document that becomes the input for the next phase.

**Note**: After creating any document in `thoughts/`, run `humanlayer thoughts sync` to commit it to your thoughts repository.

---

## Scenario: Building a New AI-Powered Customer Support System

This document shows the complete workflow for implementing advanced context engineering on a brand new project from initial requirements through production deployment.

**Project Goal**: Build an AI customer support system that integrates with existing CRM, handles multiple communication channels (email, chat, phone), and escalates complex issues to humans.

---

## Phase 1: Project Initiation & Research

### Input: Initial Requirements
```
User Request: "We need an AI customer support system that can handle basic inquiries,
access customer data from our CRM, and escalate complex issues to human agents."

Business Context:
- Current team: 5 support agents handling 200 tickets/day
- Existing systems: Salesforce CRM, Zendesk, Slack
- Goal: Handle 70% of inquiries automatically
- Budget: 3 months development time
```

### Step 1.1: Create Research Command
**Command Used**: `/research_codebase`
**Input**: Project requirements document
**Human Action**: Review and clarify requirements

```markdown
Research Question: "What would be required to build an AI customer support system
that integrates with Salesforce CRM and Zendesk, handles multiple communication channels,
and includes human escalation workflows?"
```

### Step 1.2: Parallel Sub-Agent Research
**Agents Spawned**:

1. **web-search-researcher**: External API research
   - Salesforce API capabilities and authentication
   - Zendesk integration options
   - AI customer support best practices
   - Human-in-the-loop patterns for support

2. **codebase-pattern-finder**: Similar implementations
   - Look for existing customer support integrations in codebase
   - Find authentication patterns for third-party APIs
   - Identify communication channel handling patterns

3. **thoughts-locator**: Historical context
   - Previous customer support projects
   - Integration patterns and decisions
   - Architectural choices made by the team

### Step 1.3: Research Synthesis & Documentation
**Output**: `thoughts/shared/research/2025-09-18-ENG-2401-ai-customer-support-research.md`

```markdown
# Research: AI Customer Support System Architecture

## Research Question
What would be required to build an AI customer support system that integrates with
Salesforce CRM and Zendesk, handles multiple communication channels, and includes
human escalation workflows?

## Summary
AI customer support system requires: (1) Multi-channel input processing, (2) CRM integration
for customer context, (3) LLM-powered response generation, (4) Human escalation triggers,
(5) Audit trail and quality monitoring.

## Detailed Findings

### Integration Requirements
- **Salesforce API**: REST API with OAuth 2.0, rate limits of 100k calls/24hrs
- **Zendesk API**: Tickets API for case management, Webhooks for real-time updates
- **Communication Channels**: Email (SMTP/IMAP), Chat (WebSocket), Phone (Twilio integration)

### Architecture Components
- **Input Router**: Channel-agnostic message processing
- **Context Engine**: Customer data aggregation from multiple sources
- **AI Response Generator**: LLM integration with guardrails and templates
- **Escalation Engine**: Rule-based and ML-based escalation decisions
- **Audit System**: Full conversation logging and quality scoring

### Technical Stack Recommendations
- **Backend**: Python/FastAPI for API flexibility and AI library support
- **Database**: PostgreSQL for structured data + Redis for session management
- **LLM Integration**: OpenAI GPT-4 with custom fine-tuning for domain knowledge
- **Queue System**: Celery for background processing and escalation workflows
- **Monitoring**: Datadog for system metrics + custom quality dashboards

### Human-in-the-Loop Integration Points
1. **Confidence Threshold Escalation**: <70% confidence auto-escalates
2. **Complex Issue Detection**: Multi-turn conversations beyond 5 exchanges
3. **Sensitive Topic Handling**: Financial, legal, or complaint keywords
4. **Quality Review**: Random sampling of AI responses for training feedback

## Code References
- Similar CRM integration: `integrations/salesforce_client.py:45-120`
- Existing escalation patterns: `workflows/support_routing.py:78-145`
- Authentication handling: `auth/oauth_manager.py:23-67`

## Architecture Insights
- Use event-driven architecture for real-time response requirements
- Implement circuit breakers for third-party API reliability
- Design for horizontal scaling with stateless request processing
- Build comprehensive audit trails for compliance and quality improvement

## Open Questions
- Customer data privacy requirements and data retention policies
- Integration testing strategy for multiple third-party systems
- Fallback procedures when AI or integrations are unavailable
- Metrics and SLA definitions for response quality and speed
```

**Human Review**: ✅ Approved - Architecture approach confirmed, privacy requirements clarified

---

## Phase 2: Implementation Planning

### Step 2.1: Create Implementation Plan
**Command Used**: `/create_plan`
**Input**: Research document + clarified requirements
**Human Action**: Review plan structure and approve phases

### Step 2.2: Sub-Agent Planning Research
**Agents Spawned**:

1. **codebase-analyzer**: Deep dive into existing patterns
   - Analyze current authentication implementation
   - Review existing API integration patterns
   - Study current error handling and monitoring setup

2. **thoughts-analyzer**: Historical decision analysis
   - Extract lessons learned from similar projects
   - Identify architectural decisions to maintain consistency
   - Review past integration challenges and solutions

### Step 2.3: Plan Creation & Human Review
**Output**: `thoughts/shared/plans/2025-09-18-ENG-2401-ai-customer-support-implementation.md`

```markdown
# AI Customer Support System Implementation Plan

## Overview
Build production-ready AI customer support system with CRM integration, multi-channel
support, and human escalation workflows in 3 phases over 8 weeks.

## Current State Analysis
- No existing customer support automation
- Salesforce CRM with custom fields for customer context
- Zendesk for ticket management with custom workflows
- Team experienced with Python/FastAPI and PostgreSQL

## Desired End State
- AI handles 70% of support inquiries automatically
- Sub-2-minute response time for all channels
- Seamless escalation to human agents with full context
- Comprehensive audit trail and quality monitoring

## What We're NOT Doing
- Custom LLM training (using OpenAI GPT-4 with prompt engineering)
- Real-time voice processing (phone transcription only)
- Advanced sentiment analysis (basic keyword detection only)

## Phase 1: Core Infrastructure (Week 1-2)

### Overview
Establish foundation with API integrations, database schema, and basic message processing.

### Changes Required:

#### 1. Database Schema Setup
**File**: `migrations/001_customer_support_schema.sql`
**Changes**: Customer support tables and indexes
```sql
CREATE TABLE conversations (
    id UUID PRIMARY KEY,
    customer_id VARCHAR(255),
    channel VARCHAR(50),
    status VARCHAR(50),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE TABLE messages (
    id UUID PRIMARY KEY,
    conversation_id UUID REFERENCES conversations(id),
    sender_type VARCHAR(20), -- 'customer', 'ai', 'agent'
    content TEXT,
    metadata JSONB,
    created_at TIMESTAMP
);
```

#### 2. API Integration Layer
**File**: `integrations/salesforce_client.py`
**Changes**: Salesforce API client with customer data fetching
```python
class SalesforceClient:
    def __init__(self, client_id, client_secret, instance_url):
        self.auth_manager = OAuth2Manager(client_id, client_secret)
        self.instance_url = instance_url

    async def get_customer_context(self, customer_id):
        """Fetch customer data for AI context"""
        # Implementation per research findings
```

### Success Criteria:

#### Automated Verification:
- [ ] Database migration applies cleanly: `make migrate`
- [ ] All unit tests pass: `make test-integrations`
- [ ] API endpoints respond correctly: `make test-api`
- [ ] Type checking passes: `mypy src/`
- [ ] Linting passes: `make lint`

#### Manual Verification:
- [ ] Salesforce integration retrieves customer data correctly
- [ ] Zendesk ticket creation works end-to-end
- [ ] Error handling gracefully manages API failures
- [ ] Authentication flow works for all third-party services

## Phase 2: AI Processing Engine (Week 3-4)

### Overview
Implement AI response generation with context awareness and quality controls.

### Changes Required:

#### 1. AI Response Engine
**File**: `ai/response_generator.py`
**Changes**: LLM integration with customer context
```python
class ResponseGenerator:
    def __init__(self, llm_client, context_builder):
        self.llm = llm_client
        self.context_builder = context_builder

    async def generate_response(self, message, customer_context):
        """Generate contextualized AI response"""
        # Implementation with confidence scoring
```

### Success Criteria:

#### Automated Verification:
- [ ] AI responses generated for test cases: `make test-ai`
- [ ] Context integration works: `pytest tests/test_context.py`
- [ ] Confidence scoring functions correctly: `make test-confidence`

#### Manual Verification:
- [ ] AI responses are relevant and helpful
- [ ] Customer context properly influences responses
- [ ] Escalation triggers work for low-confidence cases

## Phase 3: Human Escalation & Production (Week 5-8)

### Overview
Complete human escalation workflows, monitoring, and production deployment.

### Changes Required:

#### 1. Escalation Engine
**File**: `workflows/escalation_engine.py`
**Changes**: Human-in-the-loop workflow implementation
```python
@hl.require_approval()
async def escalate_to_human(conversation_id, escalation_reason):
    """Escalate conversation to human agent"""
    # Implementation with full context transfer
```

### Success Criteria:

#### Automated Verification:
- [ ] End-to-end tests pass: `make test-e2e`
- [ ] Performance tests meet SLA: `make test-performance`
- [ ] Security audit passes: `make security-audit`

#### Manual Verification:
- [ ] Human agents receive full context on escalation
- [ ] Quality monitoring dashboard shows accurate metrics
- [ ] System handles peak load without degradation

## Testing Strategy

### Unit Tests
- API integration reliability and error handling
- AI response generation and confidence scoring
- Escalation logic and human handoff procedures

### Integration Tests
- End-to-end conversation flows across all channels
- Third-party API failure scenarios and recovery
- Human escalation with context preservation

### Manual Testing Steps
1. Process customer inquiry through each communication channel
2. Verify AI response quality and relevance for various inquiry types
3. Test escalation flow with human agent takeover
4. Validate audit trail completeness and accuracy

## Performance Considerations
- Target <2-second response time for AI-generated responses
- Handle 1000 concurrent conversations without degradation
- Maintain 99.9% uptime for customer-facing endpoints

## Migration Notes
- Gradual rollout starting with email channel only
- A/B testing with 10% of traffic for quality validation
- Human agent training on new escalation interface required
```

**Human Review**: ✅ Approved - Implementation phases and success criteria confirmed

---

## Phase 3: Implementation Execution (Week 1-8)

### Week 1: Phase 1 Implementation

#### Step 3.1: Execute Phase 1 Plan
**Command Used**: `/implement_plan thoughts/shared/plans/2025-09-18-ENG-2401-ai-customer-support-implementation.md`

**Context Management**: Plan loaded, Phase 1 marked as in-progress

**Implementation Process**:
1. Create database migration files
2. Implement Salesforce integration client
3. Build Zendesk API wrapper
4. Set up basic FastAPI endpoints
5. Write unit tests for all components

**Outputs Created**:
- `migrations/001_customer_support_schema.sql`
- `integrations/salesforce_client.py`
- `integrations/zendesk_client.py`
- `api/conversation_endpoints.py`
- `tests/test_integrations.py`

**Progress Tracking**: Plan file updated with checkmarks
```markdown
## Phase 1: Core Infrastructure (Week 1-2)
- [✓] Database schema setup - 2025-09-18 14:30
- [✓] Salesforce API integration - 2025-09-18 16:45
- [✓] Zendesk API integration - 2025-09-19 10:15
- [✓] Basic API endpoints - 2025-09-19 15:20
- [✓] Unit tests - 2025-09-19 17:00

## Success Criteria:
#### Automated Verification:
- [✓] Database migration applies cleanly: `make migrate`
- [✓] All unit tests pass: `make test-integrations`
- [✓] API endpoints respond correctly: `make test-api`
- [✓] Type checking passes: `mypy src/`
- [✓] Linting passes: `make lint`

#### Manual Verification:
- [✓] Salesforce integration retrieves customer data correctly
- [✓] Zendesk ticket creation works end-to-end
- [✓] Error handling gracefully manages API failures
- [✓] Authentication flow works for all third-party services
```

**Context Refresh**: At 38% utilization, trigger intentional compaction
**Action**: Update progress file, start fresh session with Phase 2 context

### Week 3: Phase 2 Implementation

#### Step 3.2: Context Recovery & Phase 2 Start
**Session Start**: New agent instance with compacted context from progress file
**Current State**: Phase 1 complete, Phase 2 in-progress

**Key Context Preserved**:
- Phase 1 completed successfully with all tests passing
- Database schema established with conversation and message tables
- API integrations working for Salesforce and Zendesk
- Ready to implement AI processing engine

**Implementation Process**:
1. Implement OpenAI client wrapper
2. Build customer context aggregation system
3. Create AI response generation engine
4. Add confidence scoring mechanism
5. Implement basic escalation triggers

**Outputs Created**:
- `ai/llm_client.py`
- `ai/context_builder.py`
- `ai/response_generator.py`
- `workflows/confidence_scorer.py`
- `workflows/escalation_triggers.py`

### Week 5: Phase 3 Implementation

#### Step 3.3: Human Layer Integration
**Human-in-the-Loop Implementation**:

**Command Used**: Continue `/implement_plan` with Phase 3 focus

**Key Integrations**:
```python
# Human approval required for sensitive escalations
@hl.require_approval()
async def escalate_sensitive_issue(conversation_id, issue_type):
    """Handle escalation of sensitive customer issues"""
    context = await build_escalation_context(conversation_id)
    return await transfer_to_human_agent(context)

# Quality review sampling
@hl.require_approval(sample_rate=0.1)  # 10% sampling
async def log_ai_response(response, confidence_score):
    """Log AI responses for quality review"""
    if confidence_score < 0.8:
        return await request_quality_review(response)
```

**Slack Integration Setup**:
- Escalation notifications sent to `#customer-support` channel
- Human agents can approve/reject escalations directly from Slack
- Context and conversation history included in escalation messages

### Week 6-8: Production Deployment

#### Step 3.4: Gradual Rollout
**Deployment Strategy**:
1. **Week 6**: Deploy to staging with full test suite
2. **Week 7**: 10% production traffic (email channel only)
3. **Week 8**: 50% production traffic, add chat channel

**Quality Monitoring**:
```markdown
## Production Metrics (Week 7)
- **Automated Resolution Rate**: 72% (target: 70%) ✅
- **Average Response Time**: 1.8 seconds (target: <2s) ✅
- **Escalation Rate**: 28% (expected based on confidence thresholds) ✅
- **Customer Satisfaction**: 4.2/5 (89% positive) ✅
- **Human Agent Workload**: Reduced by 65% ✅

## Quality Indicators
- **False Escalations**: 3% (AI was actually correct)
- **Missed Escalations**: 1% (AI should have escalated)
- **Response Relevance**: 94% (human evaluation)
- **Context Accuracy**: 97% (customer data correctly used)
```

---

## Final Outcomes & Lessons Learned

### Project Success Metrics

**Technical Achievements**:
- ✅ 72% automated resolution rate (exceeded 70% target)
- ✅ Sub-2-second response times maintained
- ✅ Zero production outages during rollout
- ✅ Seamless human escalation with full context preservation

**Business Impact**:
- ✅ Support team capacity increased 3x with same headcount
- ✅ Customer satisfaction improved from 3.8 to 4.2/5
- ✅ First response time improved from 4 hours to 2 seconds
- ✅ Complex issue resolution improved (agents get better context)

### Context Engineering Benefits Realized

**Context Utilization**:
- Maintained under 40% throughout 8-week implementation
- Zero "shouting match" incidents with AI agents
- Context refresh triggered only 3 times during implementation
- Sub-agent utilization reduced main context pollution by 75%

**Knowledge Preservation**:
- Complete audit trail of all decisions and reasoning
- New team members onboarded in 2 days vs 2 weeks previously
- Architecture decisions fully documented and searchable
- Implementation patterns reusable for future AI projects

**Human Review Efficiency**:
- Research phase review: 2 hours vs 20 hours of code review
- Planning phase review: 3 hours vs 40 hours of debugging rework
- Implementation errors caught early: 95% fewer production bugs

### Key Success Factors

1. **Systematic Research Phase**: Parallel sub-agents gathered comprehensive context without overwhelming main session
2. **Detailed Planning**: Human review at planning stage prevented architectural mistakes that would have cost weeks
3. **Incremental Implementation**: Context management allowed complex project to maintain coherence across 8 weeks
4. **Human-in-the-Loop Integration**: Quality improved through strategic human review points
5. **Specification-First Approach**: Documentation became primary asset, enabling team scaling

### Workflow Artifacts Created

**Research Phase**:

- `thoughts/shared/research/2025-09-18-ENG-2401-ai-customer-support-research.md`

**Planning Phase:

- `thoughts/shared/plans/2025-09-18-ENG-2401-ai-customer-support-implementation.md`

**Implementation Phase:

- `thoughts/shared/progress/2025-09-18-ENG-2401-ai-customer-support-progress.md` (updated continuously)
- Complete codebase with 47 files, 12,000 lines of production code
- Comprehensive test suite with 95% coverage
- Production monitoring and alerting system

**Knowledge Assets**:
- Architecture decision records for future projects
- Integration patterns for Salesforce, Zendesk, and OpenAI
- Human escalation workflow templates
- Quality monitoring dashboards and metrics

---

## Template for Future Projects

This workflow can be replicated for any complex AI project:

### Phase Templates
1. **Research Phase** (10-15% of project time)
   - Spawn parallel sub-agents for comprehensive investigation
   - Create structured research document with file:line references
   - Human review focused on architectural understanding

2. **Planning Phase** (15-20% of project time)
   - Break implementation into weekly phases
   - Define automated and manual success criteria
   - Human review focused on implementation approach

3. **Implementation Phase** (65-75% of project time)
   - Follow plan with real-time progress tracking
   - Context refresh at 40% utilization
   - Continuous specification updates

### Success Indicators
- Context utilization stays under 40%
- Human review focused on high-leverage phases
- Knowledge assets created for team learning
- Implementation matches specification 90%+
- Zero "vibe coding" - all changes traceable to specifications

### Scaling Benefits
- **Individual Developer**: 3x faster complex problem resolution
- **Team Onboarding**: 80% reduction in ramp-up time
- **Code Quality**: 95% fewer production bugs
- **Knowledge Sharing**: Searchable decision history across all projects