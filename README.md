# CampusResolve — Hackathon Demo

CampusResolve is a web-based campus issue reporting and resolution platform designed for a 24–48 hour hackathon. The included `index.html` is a functional front-end prototype with simulated AI triage and admin workflows.

## 1. Architecture

```text
 Student Web / PWA / QR / Voice
             |
        API Gateway
             |
  +----------+-----------+
  |                      |
Auth/RBAC             Issue Service
  |                      |
  |          +-----------+-----------+
  |          |                       |
  |      AI Triage              Assignment
  |      Duplicate              Engine
  |      Detection                 |
  |          |                 Staff Queue
  +----------+---------------------+
             |
      PostgreSQL + Redis
             |
   +---------+----------+
   |                    |
Analytics            Notifications
   |                 Email/SMS/Push
   |
IoT / Calendar / Maps
```

### Recommended stack
- Frontend: React + Vite + Tailwind/PWA
- Backend: FastAPI or Node.js/NestJS
- Database: PostgreSQL + PostGIS
- Cache/queues: Redis + Celery/BullMQ
- AI: sentence-transformers/OpenAI-compatible embeddings + classifier
- Storage: S3-compatible object storage
- Auth: University SSO/OIDC + email verification
- Notifications: Firebase Cloud Messaging + email/SMS provider
- Deployment: Docker + managed PostgreSQL; Kubernetes only after adoption

## 2. Three+ differentiators

1. **AI duplicate clustering** — embeddings compare new reports with recent issues and merge/relate duplicates instead of creating ticket floods.
2. **Predictive maintenance** — IoT/pattern signals create preventive work orders before students report failures.
3. **Academic-calendar-aware priority** — classroom issues get priority boosts around exams, practicals and high-occupancy periods.
4. **Community resolution + reputation** — verified student helpers can validate simple fixes; quality reports earn impact points and badges.
5. **Live heatmap + sustainability score** — operations teams see issue density and environmental impact by location.

## 3. Database schema

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  university_id VARCHAR(80) UNIQUE NOT NULL,
  name VARCHAR(160) NOT NULL,
  email VARCHAR(200) UNIQUE NOT NULL,
  role VARCHAR(30) NOT NULL,
  reputation INT DEFAULT 0,
  notification_prefs JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE locations (
  id UUID PRIMARY KEY,
  campus_id UUID NOT NULL,
  name VARCHAR(160) NOT NULL,
  geom GEOGRAPHY(POINT,4326),
  qr_code VARCHAR(120) UNIQUE
);

CREATE TABLE issues (
  id UUID PRIMARY KEY,
  reporter_id UUID REFERENCES users(id),
  location_id UUID REFERENCES locations(id),
  title VARCHAR(220) NOT NULL,
  description TEXT NOT NULL,
  category VARCHAR(60),
  priority VARCHAR(20),
  status VARCHAR(30) DEFAULT 'SUBMITTED',
  ai_confidence NUMERIC(5,2),
  predicted_eta_minutes INT,
  duplicate_group UUID,
  sustainability_score NUMERIC(5,2),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE issue_media (
  id UUID PRIMARY KEY,
  issue_id UUID REFERENCES issues(id) ON DELETE CASCADE,
  storage_url TEXT NOT NULL,
  media_type VARCHAR(30)
);

CREATE TABLE assignments (
  id UUID PRIMARY KEY,
  issue_id UUID REFERENCES issues(id) ON DELETE CASCADE,
  staff_id UUID REFERENCES users(id),
  assigned_at TIMESTAMPTZ DEFAULT now(),
  completed_at TIMESTAMPTZ
);

CREATE TABLE issue_events (
  id BIGSERIAL PRIMARY KEY,
  issue_id UUID REFERENCES issues(id) ON DELETE CASCADE,
  actor_id UUID REFERENCES users(id),
  event_type VARCHAR(60) NOT NULL,
  payload JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE sensor_alerts (
  id UUID PRIMARY KEY,
  location_id UUID REFERENCES locations(id),
  sensor_type VARCHAR(60),
  risk_score NUMERIC(5,2),
  reading JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_issue_status_priority ON issues(status, priority);
CREATE INDEX idx_issue_location_created ON issues(location_id, created_at DESC);
CREATE INDEX idx_issue_reporter_created ON issues(reporter_id, created_at DESC);
CREATE INDEX idx_issue_duplicate_group ON issues(duplicate_group);
CREATE INDEX idx_issue_geom ON locations USING GIST(geom);
CREATE INDEX idx_events_issue_created ON issue_events(issue_id, created_at DESC);
```

### Relationship rationale
- One student can submit many issues.
- One issue belongs to one physical location but can have many media files/events.
- Assignments are separate so reassignment history is preserved.
- Sensor alerts connect physical campus signals to preventive workflows.
- PostGIS enables radius/proximity searches for routing and heatmaps.

## 4. API specification

### Authentication
`POST /api/v1/auth/login`
- Input: `{email, otp}` or OIDC callback
- Output: `{access_token, user}`

### Issues
`POST /api/v1/issues`
```json
{
  "description":"Projector flickers in Lab 2",
  "location_id":"loc-123",
  "media_ids":["media-1"]
}
```

`GET /api/v1/issues?status=OPEN&priority=HIGH&location_id=loc-123`

`GET /api/v1/issues/{issue_id}`

`PATCH /api/v1/issues/{issue_id}`
```json
{"status":"IN_PROGRESS","assignee_id":"staff-42"}
```

`POST /api/v1/issues/{issue_id}/feedback`
```json
{"rating":5,"comment":"Resolved quickly"}
```

### AI
`POST /api/v1/ai/triage`
- Returns category, priority, confidence, duplicate candidates and predicted ETA.

`GET /api/v1/ai/clusters`
- Returns duplicate/related issue clusters.

### Admin
`POST /api/v1/admin/bulk-assign`
`POST /api/v1/admin/issues/{id}/escalate`
`GET /api/v1/admin/analytics`
`GET /api/v1/admin/heatmap`

### Preventive maintenance
`POST /api/v1/sensors/events`
`GET /api/v1/preventive-alerts`
`POST /api/v1/preventive-alerts/{id}/work-order`

## 5. Wireframes

### Student
```text
+------------------------------------------------+
| CampusResolve      Dashboard  Report  My Issues|
+------------------------------------------------+
| Good afternoon                                  |
| [Open 24] [Avg 6.4h] [420 points] [7 alerts]   |
|                                                |
| Campus Heatmap          Recent Issues          |
| +------------------+   +--------------------+  |
| |   •     •        |   | Projector  Assigned | |
| |      •      •    |   | Wi-Fi      Progress | |
| +------------------+   +--------------------+  |
|                                                |
|              [ REPORT AN ISSUE ]               |
+------------------------------------------------+
```

### Report screen
```text
+----------------------+-------------------------+
| Description          | AI TRIAGE PREVIEW       |
| [................]   | Category: Equipment    |
| Location [QR/Map]    | Priority: High        |
| Photo / Video        | Duplicate: 1 nearby   |
| Voice-to-text        | ETA: 2–6 hours        |
| [AI Analyze] [Submit]| Confidence: 87%       |
+----------------------+-------------------------+
```

### Admin
```text
+------------------------------------------------------+
| ADMIN COMMAND CENTER                                 |
| Critical 3 | Unassigned 8 | SLA 94% | CSAT 4.6/5   |
+------------------------------------------------------+
| Filters | Bulk Assign | Escalate                    |
+------------------------------------------------------+
| ID | Issue | Location | Priority | Staff | Status  |
|----|-------|----------|----------|-------|---------|
|1042|Proj.  |CSE Lab 2 | HIGH     |Tech A |Assigned|
+------------------------------------------------------+
| Heatmap | Bottlenecks | Predictive Maintenance     |
+------------------------------------------------------+
```

## 6. 24–48 hour roadmap

### Hours 0–6: MVP foundation
- React/Vite UI
- PostgreSQL schema
- Auth mock/SSO interface
- Issue create/list/detail APIs
- Student/admin roles

### Hours 6–14: Smart reporting
- Photo upload
- QR location
- AI category/priority
- Duplicate similarity
- Status lifecycle

### Hours 14–22: Operations
- Assignment engine
- Admin filters
- Bulk assignment
- Heatmap
- ETA calculation
- Email/push notification hooks

### Hours 22–32: Differentiators
- Academic calendar priority
- Gamification
- Predictive maintenance mock IoT stream
- Community fixer workflow
- Sustainability score

### Hours 32–40: Analytics + polish
- KPI dashboard
- Bottleneck analytics
- Accessibility
- Mobile/PWA polish
- Demo data and seeded scenarios

### Hours 40–48: Stretch
- Real university SSO
- Real IoT MQTT stream
- SMS integration
- Production observability
- Blockchain/hash-anchored audit export

**MVP for judging:** authentication/roles, issue submission, AI triage, duplicate detection, assignment, status tracking, admin dashboard, heatmap, gamification, and predictive alert demo.

## 7. Technical challenges and mitigation

| Challenge | Mitigation |
|---|---|
| AI misclassification | Confidence threshold + human override |
| Duplicate false positives | Similarity threshold + location/time filters |
| Notification failures | Queue + retry + in-app notification fallback |
| Sensitive media | Private object storage + signed URLs |
| Location privacy | Store building/zone instead of precise student location by default |
| Staff overload | Workload-aware assignment + SLA escalation |
| IoT noise | Rolling averages and anomaly thresholds |
| Multi-campus differences | Tenant/campus IDs and configurable categories/SLA |
| Vendor lock-in | Adapter interfaces for auth, maps, notifications and AI |

## 8. Multi-campus scalability

Use a tenant-aware data model:
`University → Campus → Building → Floor → Room → Asset`.

Partition analytics by campus and time window. Keep issue data in PostgreSQL/PostGIS, media in object storage, and asynchronous AI/notification jobs in Redis-backed workers. Use read replicas for analytics, CDN/object storage for media, and campus-specific configuration for SLAs, teams, calendars and escalation rules.

## 9. Demo script

1. Student scans a classroom QR code.
2. Types “projector is flickering before tomorrow's lab exam”.
3. AI classifies it as Equipment + High priority.
4. Academic calendar increases priority.
5. Duplicate engine finds a nearby related report.
6. Assignment engine routes it to the nearest available AV technician.
7. Admin sees the ticket on the live heatmap.
8. Student receives status/ETA updates.
9. Technician resolves it and student rates the fix.
10. Repeated projector failures trigger a predictive maintenance alert.

## 10. Suggested project name

**CampusResolve — AI Campus Operations Copilot**

Tagline: **Report once. Route smart. Fix faster.**
