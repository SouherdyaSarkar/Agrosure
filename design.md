# Design Document

## AgroSure – AI-Driven Crop Loan & Insurance Platform

---

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Presentation Layer                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Farmer Web   │  │ Bank/Insurer │  │ Kiosk Mode   │      │
│  │ Portal       │  │ Dashboard    │  │ (Offline)    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                       │
│         (Authentication, Rate Limiting, Routing)             │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Backend    │  │  AI/ML       │  │ Integration  │
│   Services   │  │  Service     │  │  Layer       │
│              │  │              │  │              │
│ • Auth       │  │ • Yield      │  │ • Banking    │
│ • Loan Mgmt  │  │   Prediction │  │   APIs       │
│ • Insurance  │  │ • Risk Score │  │ • Satellite  │
│ • Reports    │  │ • Fraud Det. │  │   APIs       │
│ • Notif.     │  │ • Weather    │  │ • Weather    │
└──────────────┘  └──────────────┘  └──────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       Data Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ PostgreSQL   │  │ Redis Cache  │  │ Object Store │      │
│  │ (Primary DB) │  │              │  │ (S3/GCS)     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Component Breakdown

#### Presentation Layer
- React-based responsive web application
- Tailwind CSS for styling
- Web Speech API for voice input
- Progressive Web App (PWA) for offline capability

#### API Gateway
- Request routing and load balancing
- JWT token validation
- Rate limiting and throttling
- API versioning
- Request/response logging

#### Backend Services
- RESTful API architecture
- Microservices pattern
- Service-to-service authentication
- Event-driven architecture for async operations

#### AI/ML Service
- Isolated Python service
- Model serving with versioning
- Feature engineering pipeline
- Model monitoring and retraining triggers

#### Integration Layer
- Adapter pattern for external APIs
- Circuit breaker for fault tolerance
- Retry logic with exponential backoff
- API response caching

---

## 2. Technology Stack

### 2.1 Frontend
- **Framework**: React 18+
- **Styling**: Tailwind CSS
- **State Management**: Redux Toolkit / Zustand
- **Voice Input**: Web Speech API
- **Charts**: Recharts / Chart.js
- **Forms**: React Hook Form + Zod validation
- **HTTP Client**: Axios
- **Build Tool**: Vite

### 2.2 Backend
- **Runtime**: Node.js (Express) or Python (FastAPI)
- **Authentication**: JWT + Refresh Tokens
- **Validation**: Joi / Pydantic
- **ORM**: Prisma / SQLAlchemy
- **Task Queue**: Bull (Redis-based)
- **Logging**: Winston / Pino
- **API Documentation**: Swagger/OpenAPI

### 2.3 AI/ML Stack
- **Language**: Python 3.10+
- **ML Framework**: Scikit-learn, XGBoost
- **Deep Learning**: TensorFlow / PyTorch
- **Model Serving**: FastAPI + Uvicorn
- **Feature Store**: Feast (optional)
- **Experiment Tracking**: MLflow
- **Data Processing**: Pandas, NumPy
- **Geospatial**: GeoPandas, Rasterio

### 2.4 Database & Storage
- **Primary Database**: PostgreSQL 14+
- **Caching**: Redis 7+
- **Object Storage**: AWS S3 / Google Cloud Storage
- **Search**: Elasticsearch (optional for analytics)

### 2.5 DevOps & Infrastructure
- **Cloud Provider**: AWS / GCP
- **Containerization**: Docker
- **Orchestration**: Kubernetes / ECS
- **CI/CD**: GitHub Actions / GitLab CI
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack / CloudWatch
- **APM**: New Relic / Datadog

---

## 3. Database Design

### 3.1 Core Entities

#### Users Table
```sql
users (
  id UUID PRIMARY KEY,
  phone_number VARCHAR(15) UNIQUE NOT NULL,
  role ENUM('farmer', 'bank', 'insurance', 'admin'),
  language_preference VARCHAR(10),
  created_at TIMESTAMP,
  updated_at TIMESTAMP
)
```

#### Farmers Table
```sql
farmers (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  full_name VARCHAR(255),
  location GEOGRAPHY(POINT),
  state VARCHAR(100),
  district VARCHAR(100),
  village VARCHAR(100),
  total_land_size DECIMAL(10,2),
  verification_status ENUM('pending', 'verified', 'rejected'),
  created_at TIMESTAMP
)
```

#### Land Parcels Table
```sql
land_parcels (
  id UUID PRIMARY KEY,
  farmer_id UUID REFERENCES farmers(id),
  land_size DECIMAL(10,2),
  location GEOGRAPHY(POLYGON),
  soil_type VARCHAR(50),
  irrigation_type VARCHAR(50),
  ownership_document_url TEXT,
  created_at TIMESTAMP
)
```

#### Crops Table
```sql
crops (
  id UUID PRIMARY KEY,
  land_parcel_id UUID REFERENCES land_parcels(id),
  crop_type VARCHAR(100),
  variety VARCHAR(100),
  sowing_date DATE,
  expected_harvest_date DATE,
  actual_harvest_date DATE,
  status ENUM('planned', 'sown', 'growing', 'harvested'),
  created_at TIMESTAMP
)
```

#### Yield Predictions Table
```sql
yield_predictions (
  id UUID PRIMARY KEY,
  crop_id UUID REFERENCES crops(id),
  predicted_yield DECIMAL(10,2),
  confidence_score DECIMAL(5,2),
  model_version VARCHAR(50),
  prediction_date TIMESTAMP,
  factors JSONB,
  created_at TIMESTAMP
)
```

#### Loan Applications Table
```sql
loan_applications (
  id UUID PRIMARY KEY,
  farmer_id UUID REFERENCES farmers(id),
  crop_id UUID REFERENCES crops(id),
  requested_amount DECIMAL(12,2),
  recommended_amount DECIMAL(12,2),
  risk_score DECIMAL(5,2),
  status ENUM('draft', 'submitted', 'under_review', 'approved', 'rejected', 'disbursed'),
  bank_id UUID REFERENCES users(id),
  interest_rate DECIMAL(5,2),
  tenure_months INTEGER,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
)
```

#### Insurance Policies Table
```sql
insurance_policies (
  id UUID PRIMARY KEY,
  farmer_id UUID REFERENCES farmers(id),
  crop_id UUID REFERENCES crops(id),
  provider_id UUID REFERENCES users(id),
  policy_number VARCHAR(100) UNIQUE,
  premium_amount DECIMAL(10,2),
  coverage_amount DECIMAL(12,2),
  start_date DATE,
  end_date DATE,
  status ENUM('active', 'expired', 'claimed', 'cancelled'),
  created_at TIMESTAMP
)
```

#### Claims Table
```sql
claims (
  id UUID PRIMARY KEY,
  policy_id UUID REFERENCES insurance_policies(id),
  claim_amount DECIMAL(12,2),
  reason TEXT,
  evidence_urls JSONB,
  satellite_verification JSONB,
  status ENUM('submitted', 'under_review', 'approved', 'rejected', 'settled'),
  fraud_score DECIMAL(5,2),
  settlement_amount DECIMAL(12,2),
  created_at TIMESTAMP,
  updated_at TIMESTAMP
)
```

### 3.2 Indexing Strategy
- B-tree indexes on foreign keys
- GiST indexes on geography columns
- Composite indexes on (farmer_id, created_at) for queries
- Partial indexes on status columns for active records

---

## 4. API Design

### 4.1 Authentication Endpoints
```
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/verify-otp
POST   /api/v1/auth/refresh-token
POST   /api/v1/auth/logout
```

### 4.2 Farmer Endpoints
```
GET    /api/v1/farmers/profile
PUT    /api/v1/farmers/profile
POST   /api/v1/farmers/land-parcels
GET    /api/v1/farmers/land-parcels
POST   /api/v1/farmers/crops
GET    /api/v1/farmers/crops
GET    /api/v1/farmers/dashboard
```

### 4.3 Prediction Endpoints
```
POST   /api/v1/predictions/yield
GET    /api/v1/predictions/:id
GET    /api/v1/predictions/history
```

### 4.4 Loan Endpoints
```
POST   /api/v1/loans/applications
GET    /api/v1/loans/applications
GET    /api/v1/loans/applications/:id
PUT    /api/v1/loans/applications/:id/status
GET    /api/v1/loans/eligibility
```

### 4.5 Insurance Endpoints
```
GET    /api/v1/insurance/policies
POST   /api/v1/insurance/policies
GET    /api/v1/insurance/policies/:id
POST   /api/v1/insurance/claims
GET    /api/v1/insurance/claims
PUT    /api/v1/insurance/claims/:id/status
```

### 4.6 Bank/Insurance Endpoints
```
GET    /api/v1/admin/applications
GET    /api/v1/admin/applications/:id
PUT    /api/v1/admin/applications/:id/review
GET    /api/v1/admin/analytics
GET    /api/v1/admin/fraud-alerts
```

---

## 5. AI/ML Architecture

### 5.1 Yield Prediction Model

#### Input Features
- Land size (acres)
- Crop type (categorical)
- Soil type (categorical)
- Irrigation type (categorical)
- Location (lat, lon)
- Sowing date (day of year)
- Historical weather data (30-day average)
- Satellite indices (NDVI, EVI, NDWI)
- Historical yield data (regional)

#### Model Pipeline
```
Data Collection → Feature Engineering → Model Training → Validation → Deployment
```

#### Model Architecture
- Ensemble approach: XGBoost + Random Forest
- Separate models per crop type
- Hyperparameter tuning with cross-validation
- Feature importance analysis

#### Output
- Predicted yield (kg/acre)
- Confidence score (0-100%)
- Contributing factors breakdown

### 5.2 Risk Scoring Model

#### Input Features
- Predicted yield
- Historical farmer data
- Weather risk factors
- Soil quality metrics
- Crop type risk profile
- Regional default rates

#### Output
- Risk score (0-100)
- Risk category (low/medium/high)
- Recommended loan amount
- Suggested interest rate adjustment

### 5.3 Fraud Detection Model

#### Input Features
- Application data consistency
- Satellite imagery verification
- Historical claim patterns
- Geospatial anomalies
- Document verification results
- Behavioral patterns

#### Model Type
- Anomaly detection (Isolation Forest)
- Classification (Gradient Boosting)
- Rule-based heuristics

#### Output
- Fraud probability score
- Anomaly flags
- Verification recommendations

### 5.4 Model Monitoring
- Prediction drift detection
- Performance metrics tracking
- A/B testing framework
- Automated retraining triggers
- Model versioning and rollback

---

## 6. Security Design

### 6.1 Authentication & Authorization
- JWT access tokens (15-minute expiry)
- Refresh tokens (7-day expiry)
- Role-based access control (RBAC)
- Multi-factor authentication for banks
- Session management with Redis

### 6.2 Data Security
- Encryption at rest (AES-256)
- Encryption in transit (TLS 1.3)
- PII data masking in logs
- Secure key management (AWS KMS / GCP KMS)
- Database encryption

### 6.3 API Security
- Rate limiting (100 req/min per user)
- Input validation and sanitization
- SQL injection prevention (parameterized queries)
- XSS protection
- CORS configuration
- API key rotation

### 6.4 Compliance
- GDPR data handling
- PCI-DSS for payment data
- Audit logging (immutable)
- Data retention policies
- Right to deletion implementation

---

## 7. Deployment Architecture

### 7.1 Cloud Infrastructure (AWS Example)

```
┌─────────────────────────────────────────────┐
│          Route 53 (DNS)                     │
└─────────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────────┐
│     CloudFront (CDN) + WAF                  │
└─────────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────────┐
│   Application Load Balancer                 │
└─────────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
┌──────────────┐        ┌──────────────┐
│   ECS/EKS    │        │   ECS/EKS    │
│   Frontend   │        │   Backend    │
│   Container  │        │   Container  │
└──────────────┘        └──────────────┘
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
            ┌──────────────┐        ┌──────────────┐
            │   RDS        │        │   ElastiCache│
            │  PostgreSQL  │        │   (Redis)    │
            └──────────────┘        └──────────────┘
```

### 7.2 Scaling Strategy
- Horizontal pod autoscaling (HPA)
- Database read replicas
- CDN for static assets
- Redis cluster for caching
- Async job processing with queues

### 7.3 Disaster Recovery
- Multi-AZ deployment
- Automated backups (daily)
- Point-in-time recovery
- Cross-region replication (optional)
- Backup retention: 30 days

---

## 8. Monitoring & Observability

### 8.1 Metrics
- Application metrics (request rate, latency, errors)
- Business metrics (predictions/day, loan approvals)
- Infrastructure metrics (CPU, memory, disk)
- ML model metrics (accuracy, drift)

### 8.2 Logging
- Structured JSON logs
- Centralized log aggregation
- Log levels: ERROR, WARN, INFO, DEBUG
- PII redaction in logs

### 8.3 Alerting
- Error rate thresholds
- Latency SLA violations
- Model performance degradation
- Infrastructure health checks
- On-call rotation with PagerDuty

---

## 9. Offline Support Design

### 9.1 Kiosk Mode
- Progressive Web App (PWA)
- Service Worker for offline caching
- IndexedDB for local data storage
- Background sync when online
- Simplified prediction model (cached)

### 9.2 Data Synchronization
- Queue-based sync mechanism
- Conflict resolution strategy
- Incremental sync
- Sync status indicators

---

## 10. Future Enhancements

### Phase 2
- GraphQL API for flexible queries
- Real-time notifications (WebSocket)
- Mobile native apps (React Native)
- Advanced analytics dashboard
- Blockchain integration for transparency

### Phase 3
- IoT sensor integration
- Drone imagery processing
- Predictive maintenance alerts
- Market price prediction
- Farmer community features
