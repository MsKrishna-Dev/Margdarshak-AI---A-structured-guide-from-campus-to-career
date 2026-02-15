# Campus-to-Hire Learning Coach: System Architecture Overview

## Problem-to-Solution Mapping

### Student Problems → System Solutions

| Student Problem | Root Cause | System Solution | Key Component |
|----------------|------------|-----------------|---------------|
| **"I don't know what to learn"** | Academic curriculum disconnected from job requirements | Curriculum-to-Job Mapping Engine | AI analyzes syllabus + job postings → generates personalized learning path |
| **"I skip basics and fail later"** | No accountability in self-learning | Readiness Gate System | Mandatory 80% pass threshold blocks progression until mastery verified |
| **"English technical content is hard"** | Language barrier + poor translation | Multilingual Reasoning Engine | AI explains concepts in Hindi/Tamil/Telugu using local analogies (not word translation) |
| **"I fail interviews repeatedly"** | Unfamiliar with Indian hiring patterns | Interview Simulation Engine | AI conducts mock interviews matching TCS/Infosys/startup patterns |
| **"I waste time on easy topics"** | No visibility into weak areas | Weak-Area Heatmap System | Color-coded skill map (red/yellow/green) auto-adjusts learning focus |
| **"Internet keeps disconnecting"** | Intermittent 2G/3G connectivity | Offline Learning Packs | Download <5MB packs, learn offline, sync when connected |

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        STUDENT (Mobile-First)                       │
│  Constraints: 2G/3G, Hindi/Regional Language, Limited Data Budget   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   MOBILE APP (Progressive Web App)                  │
│  • Text-first UI (minimal images)                                   │
│  • Offline storage (IndexedDB)                                      │
│  • Background sync when connected                                   │
│  • <3 second page loads on 2G/3G                                    │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     CDN LAYER (CloudFront)                          │
│  • Cache static content (1 year)                                    │
│  • Compress responses (70% reduction)                               │
│  • Edge locations across India                                      │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   API GATEWAY (Entry Point)                         │
│  • REST API for standard requests                                   │
│  • WebSocket for interview simulation                               │
│  • Rate limiting (100 req/sec per student)                          │
│  • JWT authentication                                               │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    CORE BUSINESS LOGIC (Lambda)                      │
│                                                                      │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐ │
│  │ Curriculum       │  │ Readiness Gate   │  │ Learning Path    │ │
│  │ Mapping Service  │  │ Assessment       │  │ Generator        │ │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘ │
│                                                                      │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐ │
│  │ Multilingual     │  │ Interview        │  │ Heatmap          │ │
│  │ Explanation      │  │ Simulation       │  │ Generator        │ │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘ │
│                                                                      │
│  ┌──────────────────┐  ┌──────────────────┐                        │
│  │ Offline Pack     │  │ Progress         │                        │
│  │ Generator        │  │ Tracker          │                        │
│  └──────────────────┘  └──────────────────┘                        │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     AI LAYER (Amazon Bedrock)                       │
│                                                                     │
│  Claude 3 Sonnet (Primary):                                         │
│  • Curriculum-to-job semantic mapping                               │
│  • Multilingual explanations with cultural analogies                │
│  • Interview conversation and evaluation                            │
│  • Adaptive question generation                                     │
│  • Learning path optimization                                       │
│                                                                     │
│  Vector Search (OpenSearch):                                        │
│  • Store curriculum topic embeddings                                │
│  • Store job skill embeddings                                       │
│  • Semantic similarity matching (cosine > 0.7)                      │
└─────────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA LAYER                                  │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ RDS PostgreSQL (Structured Data)                             │  │
│  │ • User profiles, progress, skill profiles                    │  │
│  │ • Learning paths, readiness gates                            │  │
│  │ • ACID compliance for critical data                          │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ DynamoDB (Session Data)                                      │  │
│  │ • Assessment attempts, interview sessions                    │  │
│  │ • Single-digit ms latency                                    │  │
│  │ • Auto-scaling for 10,000+ concurrent users                  │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ S3 (Content Storage)                                         │  │
│  │ • Learning materials, offline packs                          │  │
│  │ • Cultural analogy library (Hindi/Tamil/Telugu)              │  │
│  │ • Pre-validated multilingual explanations                    │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ ElastiCache Redis (Caching)                                  │  │
│  │ • Cache curriculum mappings (7 days)                         │  │
│  │ • Cache explanations (30 days)                               │  │
│  │ • Session management                                         │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Core Data Flows

### Flow 1: Curriculum-to-Job Mapping (Solves "I don't know what to learn")

```
Student selects "Backend Developer" role
    ↓
Check Redis cache for mapping
    ↓ (cache miss)
Fetch student's curriculum from RDS
    ↓
Generate embeddings for curriculum topics (Bedrock)
    ↓
Generate embeddings for job skills (Bedrock)
    ↓
Query OpenSearch for similar vectors (threshold > 0.7)
    ↓
AI generates explanations with Indian industry examples
    ↓
Prioritize topics: Critical / Important / Supplementary
    ↓
Cache result in Redis (7 days)
    ↓
Return personalized learning path to student
```

**Why this works for Tier-2/3 students:**
- Connects their existing knowledge (OS, DBMS) to real jobs
- Uses AI to find semantic connections, not keyword matching
- Provides concrete examples from TCS, Infosys, Flipkart
- Cached to work even with poor connectivity

---

### Flow 2: Readiness Gate Assessment (Solves "I skip basics and fail later")

```
Student completes module → requests gate assessment
    ↓
Create assessment in DynamoDB
    ↓
AI generates 3 initial questions (difficulty: 5/10)
    ↓
Student answers → evaluate correctness
    ↓
Adaptive difficulty adjustment:
  • 2 correct in a row → increase difficulty
  • 2 incorrect in a row → decrease difficulty
    ↓
Continue for 15-20 questions
    ↓
Calculate final score
    ↓
IF score >= 80%:
  → Mark gate as PASSED in RDS
  → Unlock next module
  → Award mastery certificate
ELSE:
  → AI analyzes weak concepts
  → Generate remediation plan
  → Block progression until retry
```

**Why this works for Tier-2/3 students:**
- Forces mastery before progression (no skipping)
- Adaptive difficulty prevents frustration
- Immediate feedback in their language
- Builds confidence through verified competency

---

### Flow 3: Multilingual Explanation (Solves "English technical content is hard")

```
Student requests explanation for "mutex" in Hindi
    ↓
Check ElastiCache for cached explanation
    ↓ (cache miss)
Check S3 for pre-validated content
    ↓ (not found)
Load cultural analogy library from S3
    ↓
AI generates explanation by reasoning in Hindi
  • Uses railway ticket counter analogy
  • Preserves technical reasoning
  • Includes code example
    ↓
AI validates technical accuracy
    ↓
IF validation passes:
  → Store in S3 for future use
  → Cache in Redis (30 days)
ELSE:
  → Fallback to English + Amazon Translate
    ↓
Return explanation to student
```

**Why this works for Tier-2/3 students:**
- Not just translation - reasoning in native language
- Uses familiar analogies (railway, cricket, festivals)
- Preserves technical depth
- Cached for offline access

---

### Flow 4: Offline Learning Pack (Solves "Internet keeps disconnecting")

```
Student requests learning pack for next 3 modules
    ↓
Queue pack generation job in SQS
    ↓
ECS task fetches content:
  • Lessons from S3
  • Practice problems from S3
  • Assessment questions from DynamoDB
    ↓
Bundle with offline validation script
    ↓
Compress with Brotli (target < 5MB)
    ↓
Upload to S3
    ↓
Notify student via SNS
    ↓
Student downloads pack
    ↓
[OFFLINE MODE]
Student completes assessments locally
Validation runs in browser (JavaScript)
Results encrypted and stored in IndexedDB
    ↓
[BACK ONLINE]
App syncs encrypted results to Lambda
    ↓
Decrypt and merge with server state
    ↓
Update progress in RDS
Update heatmap in DynamoDB
```

**Why this works for Tier-2/3 students:**
- Learn without internet (2G/3G unreliable)
- <5MB packs fit limited data budgets
- Progress preserved even offline
- Automatic sync when connected

---

### Flow 5: Weak-Area Heatmap (Solves "I waste time on easy topics")

```
Student completes assessment
    ↓
Fetch all assessment attempts from DynamoDB
    ↓
Group by skill area, calculate average score
    ↓
Apply color coding:
  • < 60% = RED (weak)
  • 60-75% = YELLOW (moderate)
  • >= 75% = GREEN (strong)
    ↓
Detect trends (improving/stable/declining)
    ↓
Store heatmap in RDS skill_profiles
    ↓
IF RED zones detected:
  → Trigger learning path adjustment
  → Allocate 60% time to RED, 30% to YELLOW, 10% to GREEN
    ↓
Display visual heatmap to student
```

**Why this works for Tier-2/3 students:**
- Makes weaknesses visible (can't hide)
- Forces focus on difficult topics
- Visual feedback motivates improvement
- Automatic path adjustment (no manual planning)

---

### Flow 6: Interview Simulation (Solves "I fail interviews repeatedly")

```
Student starts mock interview for "Backend Developer"
    ↓
Fetch skill profile from RDS
    ↓
AI generates interview plan (8-12 questions)
  • Behavioral (STAR method)
  • Technical (CS fundamentals)
  • Puzzle-solving (Indian company patterns)
    ↓
Open WebSocket connection
    ↓
AI: "Tell me about yourself"
    ↓
Student responds via WebSocket
    ↓
AI evaluates response:
  • Technical accuracy
  • Communication clarity
  • Answer structure
  • Confidence level
    ↓
AI generates contextual follow-up question
    ↓
[Repeat for 8-12 exchanges]
    ↓
Generate comprehensive evaluation:
  • Overall score
  • Dimension scores (behavioral/technical/communication)
  • Top 3 strengths
  • Top 3 improvement areas
  • Actionable next steps
    ↓
Store evaluation in DynamoDB
Update skill profile in RDS
```

**Why this works for Tier-2/3 students:**
- Matches real Indian hiring patterns (TCS, Infosys, startups)
- Provides specific feedback (not generic)
- Safe practice environment (no real consequences)
- Improves with each attempt

---

## Why This Architecture Works for Tier-2/3 Students

### 1. Mobile-First Design
- **Problem**: Students only have smartphones, limited laptop access
- **Solution**: Progressive Web App with text-first UI, touch-optimized
- **Impact**: Works on any Android 8.0+ or iOS 12+ device

### 2. Low-Bandwidth Optimization
- **Problem**: Intermittent 2G/3G, limited data budgets
- **Solution**: 
  - CloudFront CDN caches content at edge locations
  - Aggressive compression (70% reduction)
  - Offline-first architecture with sync
  - <5MB learning packs
- **Impact**: <3 second page loads even on 2G, works offline

### 3. Multilingual Support
- **Problem**: Limited English fluency, poor translation quality
- **Solution**:
  - AI reasons directly in Hindi/Tamil/Telugu (not translation)
  - Cultural analogy library (railway, cricket, festivals)
  - Pre-validated explanations cached in S3
- **Impact**: Deep understanding in native language

### 4. AI-Powered Personalization
- **Problem**: Generic content doesn't match individual needs
- **Solution**:
  - Curriculum mapping uses semantic embeddings
  - Adaptive assessments adjust difficulty
  - Heatmaps drive personalized learning paths
  - Interview simulation matches skill level
- **Impact**: Every student gets customized journey

### 5. Accountability Mechanisms
- **Problem**: Self-learning fails without checkpoints
- **Solution**:
  - Readiness Gates block progression (80% threshold)
  - Gamification (badges, streaks, journey map)
  - Weekly progress summaries
- **Impact**: Forces mastery, maintains motivation

### 6. Scalability for 10,000+ Students
- **Problem**: Need to serve thousands concurrently
- **Solution**:
  - Serverless Lambda (auto-scales)
  - DynamoDB (single-digit ms latency)
  - ElastiCache Redis (caching reduces AI calls)
  - CloudFront CDN (offloads static content)
- **Impact**: Consistent performance under load

### 7. Cost Optimization
- **Problem**: Students can't afford expensive platforms
- **Solution**:
  - Serverless architecture (pay per use)
  - Aggressive caching (reduces AI costs)
  - Offline packs (reduces bandwidth costs)
  - Free tier for basic usage
- **Impact**: Affordable for target users

---

## Key Differentiators from Existing Platforms

| Feature | Generic AI Tutor | Campus-to-Hire Coach |
|---------|------------------|----------------------|
| **Learning Path** | Generic curriculum | Curriculum-to-job mapping with Indian industry examples |
| **Progression** | Self-paced (can skip) | Readiness Gates enforce 80% mastery |
| **Language** | English or word translation | Reasoning-based explanations with cultural analogies |
| **Interview Prep** | Generic questions | Indian hiring patterns (TCS, Infosys, startups) |
| **Weak Areas** | Manual tracking | Automatic heatmap with adaptive path adjustment |
| **Connectivity** | Requires internet | Offline packs + background sync |
| **Transparency** | Black-box recommendations | Explainable AI with confidence levels |

---

## Technology Choices Explained

### Why AWS Serverless?
- **Auto-scaling**: Handles 10,000+ concurrent users without manual intervention
- **Cost-effective**: Pay only for actual usage (critical for student affordability)
- **Low maintenance**: No server management, focus on features

### Why Amazon Bedrock (Claude)?
- **Multilingual quality**: Best-in-class for reasoning in regional languages
- **Context understanding**: Handles complex curriculum-to-job mapping
- **Conversational AI**: Realistic interview simulation
- **Managed service**: No model hosting complexity

### Why PostgreSQL + DynamoDB?
- **PostgreSQL**: ACID compliance for critical data (user progress, gates)
- **DynamoDB**: Single-digit ms latency for real-time (assessments, interviews)
- **Right tool for right job**: Relational for structured, NoSQL for sessions

### Why CloudFront CDN?
- **Edge caching**: Content served from nearest location in India
- **Compression**: 70% bandwidth reduction (critical for 2G/3G)
- **Low latency**: <3 second page loads even on slow networks

### Why Offline-First PWA?
- **Intermittent connectivity**: Students can learn without internet
- **Data budget**: Download once, use many times
- **App-like experience**: Install on home screen, works like native app

---

## Success Metrics

### User Impact
- **Time to job readiness**: 6 weeks (vs. 6-month baseline)
- **Interview confidence**: 75%+ feel confident
- **Weak-area improvement**: 80%+ of red zones improve to yellow/green in 3 weeks

### System Performance
- **Response time**: <30 seconds for AI operations, <10 seconds for feedback
- **Uptime**: 99.5% during business hours (9 AM - 9 PM IST)
- **Concurrent users**: 10,000+ without degradation

### Engagement
- **Daily active users**: 60%+ of enrolled students
- **Learning streak**: Average 20+ consecutive days
- **Content completion**: 70%+ of recommended modules

---

## Conclusion

This architecture solves the specific problems of Tier-2 and Tier-3 college students through:

1. **Direction**: Curriculum-to-job mapping shows what to learn
2. **Accountability**: Readiness Gates enforce mastery
3. **Language**: Multilingual reasoning with cultural context
4. **Interview Prep**: Indian hiring pattern simulation
5. **Focus**: Heatmaps drive adaptive learning
6. **Connectivity**: Offline packs work without internet
7. **Trust**: Explainable AI builds confidence

The serverless AWS architecture ensures scalability, cost-effectiveness, and performance while the mobile-first, offline-capable design addresses the real constraints of the target users.
