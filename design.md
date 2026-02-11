# Design Document: MarketMind Platform

## Overview

MarketMind is a comprehensive web-based marketing automation platform built with Next.js 14, TypeScript, and Supabase. The platform integrates Groq's Llama 3.1 70B model to provide AI-powered content generation, sentiment analysis, engagement prediction, and A/B testing insights. The architecture follows a modern serverless approach with Next.js API routes handling backend logic, Supabase managing authentication and data persistence, and Vercel providing deployment infrastructure.

The platform serves three primary user roles (Marketer, Creator, Editor) with distinct permissions and workflows. Core functionality includes AI-driven content creation, multi-platform content adaptation, campaign management, content scheduling, asset management, approval workflows, and comprehensive analytics.

## Architecture

### System Architecture

The platform follows a three-tier architecture:

1. **Presentation Layer**: Next.js 14 with React Server Components and Client Components
   - Server Components for static content and initial page loads
   - Client Components for interactive features (editors, calendars, real-time collaboration)
   - Tailwind CSS for responsive, mobile-first styling
   - Progressive enhancement for optimal mobile experience

2. **Application Layer**: Next.js API Routes
   - RESTful API endpoints for all business logic
   - Middleware for authentication, authorization, and rate limiting
   - Integration layer for external services (Groq AI, Supabase)
   - WebSocket connections for real-time collaboration features

3. **Data Layer**: Supabase (PostgreSQL + Storage)
   - PostgreSQL database for structured data
   - Row Level Security (RLS) for data access control
   - Supabase Storage for asset management
   - Real-time subscriptions for collaborative features

### Technology Stack Rationale

- **Next.js 14**: Provides server-side rendering, API routes, and optimal performance with App Router
- **TypeScript**: Ensures type safety and reduces runtime errors
- **Supabase**: Offers integrated authentication, database, storage, and real-time capabilities
- **Groq API**: Delivers fast AI inference with Llama 3.1 70B for content generation
- **Vercel**: Provides seamless deployment with edge functions and global CDN


## Components and Interfaces

### Authentication Module

**Purpose**: Manages user authentication and session management via Supabase Auth.

**Key Components**:
- `AuthProvider`: React context provider for authentication state
- `useAuth()`: Custom hook exposing authentication methods and user state
- `ProtectedRoute`: Component wrapper enforcing authentication requirements
- `RoleGuard`: Component wrapper enforcing role-based access control

**Interfaces**:
```typescript
interface User {
  id: string;
  email: string;
  role: 'marketer' | 'creator' | 'editor';
  name: string;
  avatar_url?: string;
  created_at: string;
}

interface AuthContext {
  user: User | null;
  loading: boolean;
  signIn: (email: string, password: string) => Promise<void>;
  signOut: () => Promise<void>;
  hasRole: (role: string | string[]) => boolean;
}
```

### AI Engine Module

**Purpose**: Interfaces with Groq API for AI-powered content generation and analysis.

**Key Components**:
- `GroqClient`: Wrapper for Groq API interactions
- `ContentGenerator`: Handles content generation requests
- `SentimentAnalyzer`: Performs sentiment and tone analysis
- `EngagementPredictor`: Predicts content engagement metrics
- `ABTestingPredictor`: Analyzes A/B test variations
- `HashtagGenerator`: Generates relevant hashtags
- `ReadabilityAnalyzer`: Calculates readability scores

**Interfaces**:
```typescript
interface ContentGenerationRequest {
  prompt: string;
  contentType: 'blog' | 'social' | 'email' | 'ad';
  tone?: string;
  targetAudience?: string;
  maxLength?: number;
}

interface ContentGenerationResponse {
  content: string;
  metadata: {
    model: string;
    tokensUsed: number;
    generationTime: number;
  };
}

interface SentimentAnalysis {
  sentiment: 'positive' | 'negative' | 'neutral';
  score: number; // -1 to 1
  confidence: number; // 0 to 1
  tone: string[];
  emotions: { [key: string]: number };
}

interface EngagementPrediction {
  platform: string;
  estimatedLikes: number;
  estimatedShares: number;
  estimatedComments: number;
  estimatedReach: number;
  confidence: number;
  suggestions: string[];
}

interface ABTestPrediction {
  variantA: {
    predictedCTR: number;
    predictedEngagement: number;
    predictedConversion: number;
  };
  variantB: {
    predictedCTR: number;
    predictedEngagement: number;
    predictedConversion: number;
  };
  recommendation: 'A' | 'B' | 'inconclusive';
  confidence: number;
  keyDifferences: string[];
}

interface HashtagSuggestion {
  hashtag: string;
  estimatedReach: number;
  popularity: 'high' | 'medium' | 'low';
  relevanceScore: number;
}

interface ReadabilityScore {
  gradeLevel: number;
  readingEase: number; // Flesch Reading Ease
  complexSentences: number;
  passiveVoice: number;
  jargonCount: number;
  suggestions: string[];
}
```

### Content Adapter Module

**Purpose**: Formats content for different social media platforms.

**Key Components**:
- `PlatformAdapter`: Base class for platform-specific adapters
- `TwitterAdapter`: Formats content for Twitter (280 char limit)
- `InstagramAdapter`: Optimizes for Instagram visual format
- `LinkedInAdapter`: Formats for professional LinkedIn tone
- `FacebookAdapter`: Optimizes for Facebook engagement

**Interfaces**:
```typescript
interface ContentAdaptation {
  platform: 'twitter' | 'instagram' | 'linkedin' | 'facebook';
  originalContent: string;
  adaptedContent: string;
  truncated: boolean;
  warnings: string[];
  metadata: {
    characterCount: number;
    hashtagCount: number;
    mentionCount: number;
  };
}

interface AdapterConfig {
  maxLength: number;
  allowHashtags: boolean;
  allowMentions: boolean;
  allowEmojis: boolean;
  preferredTone: string;
}
```

### Campaign Management Module

**Purpose**: Manages marketing campaigns and associated content.

**Key Components**:
- `CampaignService`: CRUD operations for campaigns
- `CampaignDashboard`: UI component for campaign overview
- `CampaignEditor`: Form component for campaign creation/editing
- `CampaignAnalytics`: Analytics view for campaign performance

**Interfaces**:
```typescript
interface Campaign {
  id: string;
  name: string;
  description: string;
  objectives: string[];
  targetAudience: string;
  startDate: string;
  endDate: string;
  status: 'draft' | 'active' | 'paused' | 'completed' | 'archived';
  budget?: number;
  owner_id: string;
  created_at: string;
  updated_at: string;
}

interface CampaignContent {
  campaign_id: string;
  content_id: string;
  order: number;
}

interface CampaignMetrics {
  campaign_id: string;
  totalImpressions: number;
  totalEngagements: number;
  totalClicks: number;
  totalConversions: number;
  engagementRate: number;
  clickThroughRate: number;
  conversionRate: number;
  roi?: number;
}
```

### Content Management Module

**Purpose**: Handles content creation, editing, and versioning.

**Key Components**:
- `ContentEditor`: Rich text editor component
- `ContentService`: CRUD operations for content
- `VersionControl`: Manages content versions
- `ContentPreview`: Platform-specific content preview

**Interfaces**:
```typescript
interface Content {
  id: string;
  title: string;
  body: string;
  contentType: 'blog' | 'social' | 'email' | 'ad';
  platforms: string[];
  status: 'draft' | 'review' | 'approved' | 'published' | 'archived';
  campaign_id?: string;
  author_id: string;
  created_at: string;
  updated_at: string;
  published_at?: string;
  scheduled_at?: string;
}

interface ContentVersion {
  id: string;
  content_id: string;
  version_number: number;
  body: string;
  author_id: string;
  created_at: string;
  change_summary?: string;
}

interface ContentMetadata {
  content_id: string;
  sentiment?: SentimentAnalysis;
  readability?: ReadabilityScore;
  engagement_prediction?: EngagementPrediction;
  hashtags: string[];
}
```

### Content Calendar Module

**Purpose**: Manages content scheduling and calendar visualization.

**Key Components**:
- `CalendarView`: Calendar UI component (daily, weekly, monthly views)
- `ScheduleService`: Handles content scheduling logic
- `ConflictDetector`: Identifies scheduling conflicts
- `NotificationService`: Sends scheduling notifications

**Interfaces**:
```typescript
interface ScheduledContent {
  id: string;
  content_id: string;
  scheduled_at: string;
  platforms: string[];
  status: 'scheduled' | 'published' | 'failed' | 'cancelled';
  auto_publish: boolean;
  timezone: string;
}

interface CalendarEvent {
  id: string;
  title: string;
  content_id: string;
  start: string;
  end: string;
  platforms: string[];
  color: string;
  status: string;
}
```

### Asset Library Module

**Purpose**: Manages media assets stored in Supabase Storage.

**Key Components**:
- `AssetUploader`: Handles file uploads with progress tracking
- `AssetBrowser`: Grid/list view for browsing assets
- `AssetService`: CRUD operations for assets
- `ThumbnailGenerator`: Generates thumbnails for visual assets

**Interfaces**:
```typescript
interface Asset {
  id: string;
  filename: string;
  file_type: string;
  file_size: number;
  storage_path: string;
  thumbnail_path?: string;
  campaign_id?: string;
  tags: string[];
  uploaded_by: string;
  created_at: string;
}

interface AssetUploadProgress {
  asset_id: string;
  filename: string;
  progress: number; // 0-100
  status: 'uploading' | 'processing' | 'complete' | 'error';
  error?: string;
}
```

### Approval Workflow Module

**Purpose**: Manages multi-stage content approval process.

**Key Components**:
- `WorkflowEngine`: Orchestrates approval workflow
- `ApprovalQueue`: UI component showing pending approvals
- `ApprovalService`: Handles approval operations
- `NotificationService`: Notifies approvers of pending reviews

**Interfaces**:
```typescript
interface ApprovalWorkflow {
  id: string;
  campaign_id: string;
  stages: ApprovalStage[];
  created_at: string;
}

interface ApprovalStage {
  stage_number: number;
  approver_role: string;
  approver_id?: string;
  required: boolean;
}

interface ApprovalRequest {
  id: string;
  content_id: string;
  workflow_id: string;
  current_stage: number;
  status: 'pending' | 'approved' | 'rejected' | 'cancelled';
  submitted_by: string;
  submitted_at: string;
}

interface ApprovalAction {
  id: string;
  request_id: string;
  stage_number: number;
  approver_id: string;
  action: 'approve' | 'reject' | 'request_changes';
  feedback?: string;
  created_at: string;
}
```

### Analytics Module

**Purpose**: Collects, processes, and visualizes performance metrics.

**Key Components**:
- `AnalyticsDashboard`: Main analytics UI component
- `MetricsCollector`: Aggregates performance data
- `ChartComponents`: Reusable chart components (line, bar, pie)
- `ReportGenerator`: Generates exportable reports

**Interfaces**:
```typescript
interface PerformanceMetrics {
  content_id: string;
  platform: string;
  impressions: number;
  engagements: number;
  clicks: number;
  shares: number;
  comments: number;
  likes: number;
  conversions: number;
  recorded_at: string;
}

interface AggregatedMetrics {
  period: 'day' | 'week' | 'month' | 'quarter' | 'year';
  start_date: string;
  end_date: string;
  totalImpressions: number;
  totalEngagements: number;
  avgEngagementRate: number;
  totalClicks: number;
  avgClickThroughRate: number;
  totalConversions: number;
  avgConversionRate: number;
  topPerformingContent: string[];
  platformBreakdown: { [platform: string]: number };
}

interface AnalyticsReport {
  id: string;
  title: string;
  report_type: 'campaign' | 'content' | 'platform' | 'custom';
  date_range: { start: string; end: string };
  metrics: AggregatedMetrics;
  generated_by: string;
  generated_at: string;
  format: 'pdf' | 'csv' | 'json';
}
```

### Template Library Module

**Purpose**: Manages reusable content templates.

**Key Components**:
- `TemplateLibrary`: UI component for browsing templates
- `TemplateEditor`: Component for creating/editing templates
- `TemplateService`: CRUD operations for templates
- `TemplateRenderer`: Renders templates with user data

**Interfaces**:
```typescript
interface Template {
  id: string;
  name: string;
  description: string;
  category: string;
  contentType: 'blog' | 'social' | 'email' | 'ad';
  platforms: string[];
  structure: TemplateStructure;
  placeholders: string[];
  created_by: string;
  is_public: boolean;
  created_at: string;
  updated_at: string;
}

interface TemplateStructure {
  sections: TemplateSection[];
  styling?: { [key: string]: string };
}

interface TemplateSection {
  id: string;
  type: 'text' | 'image' | 'video' | 'cta' | 'custom';
  content: string;
  placeholder?: string;
  required: boolean;
  order: number;
}
```

### Real-Time Collaboration Module

**Purpose**: Enables multiple users to collaborate on content simultaneously.

**Key Components**:
- `CollaborationProvider`: WebSocket connection manager
- `PresenceIndicator`: Shows active collaborators
- `ConflictResolver`: Handles concurrent edit conflicts
- `ActivityFeed`: Displays real-time activity updates

**Interfaces**:
```typescript
interface CollaborationSession {
  content_id: string;
  active_users: ActiveUser[];
  started_at: string;
}

interface ActiveUser {
  user_id: string;
  name: string;
  avatar_url?: string;
  cursor_position?: number;
  last_activity: string;
}

interface CollaborationEvent {
  type: 'join' | 'leave' | 'edit' | 'cursor_move';
  user_id: string;
  content_id: string;
  data?: any;
  timestamp: string;
}
```


## Data Models

### Database Schema

The platform uses PostgreSQL via Supabase with the following core tables:

#### users
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email TEXT UNIQUE NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('marketer', 'creator', 'editor')),
  name TEXT NOT NULL,
  avatar_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### campaigns
```sql
CREATE TABLE campaigns (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  description TEXT,
  objectives JSONB,
  target_audience TEXT,
  start_date DATE,
  end_date DATE,
  status TEXT NOT NULL CHECK (status IN ('draft', 'active', 'paused', 'completed', 'archived')),
  budget DECIMAL(10, 2),
  owner_id UUID REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### content
```sql
CREATE TABLE content (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  content_type TEXT NOT NULL CHECK (content_type IN ('blog', 'social', 'email', 'ad')),
  platforms TEXT[] NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('draft', 'review', 'approved', 'published', 'archived')),
  campaign_id UUID REFERENCES campaigns(id) ON DELETE SET NULL,
  author_id UUID REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  published_at TIMESTAMPTZ,
  scheduled_at TIMESTAMPTZ
);
```

#### content_versions
```sql
CREATE TABLE content_versions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  content_id UUID REFERENCES content(id) ON DELETE CASCADE,
  version_number INTEGER NOT NULL,
  body TEXT NOT NULL,
  author_id UUID REFERENCES users(id) ON DELETE CASCADE,
  change_summary TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(content_id, version_number)
);
```

#### content_metadata
```sql
CREATE TABLE content_metadata (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  content_id UUID REFERENCES content(id) ON DELETE CASCADE UNIQUE,
  sentiment JSONB,
  readability JSONB,
  engagement_prediction JSONB,
  hashtags TEXT[],
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### assets
```sql
CREATE TABLE assets (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  filename TEXT NOT NULL,
  file_type TEXT NOT NULL,
  file_size BIGINT NOT NULL,
  storage_path TEXT NOT NULL,
  thumbnail_path TEXT,
  campaign_id UUID REFERENCES campaigns(id) ON DELETE SET NULL,
  tags TEXT[],
  uploaded_by UUID REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### scheduled_content
```sql
CREATE TABLE scheduled_content (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  content_id UUID REFERENCES content(id) ON DELETE CASCADE,
  scheduled_at TIMESTAMPTZ NOT NULL,
  platforms TEXT[] NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('scheduled', 'published', 'failed', 'cancelled')),
  auto_publish BOOLEAN DEFAULT false,
  timezone TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### approval_workflows
```sql
CREATE TABLE approval_workflows (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  campaign_id UUID REFERENCES campaigns(id) ON DELETE CASCADE,
  stages JSONB NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### approval_requests
```sql
CREATE TABLE approval_requests (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  content_id UUID REFERENCES content(id) ON DELETE CASCADE,
  workflow_id UUID REFERENCES approval_workflows(id) ON DELETE CASCADE,
  current_stage INTEGER NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('pending', 'approved', 'rejected', 'cancelled')),
  submitted_by UUID REFERENCES users(id) ON DELETE CASCADE,
  submitted_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### approval_actions
```sql
CREATE TABLE approval_actions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  request_id UUID REFERENCES approval_requests(id) ON DELETE CASCADE,
  stage_number INTEGER NOT NULL,
  approver_id UUID REFERENCES users(id) ON DELETE CASCADE,
  action TEXT NOT NULL CHECK (action IN ('approve', 'reject', 'request_changes')),
  feedback TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### templates
```sql
CREATE TABLE templates (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  description TEXT,
  category TEXT NOT NULL,
  content_type TEXT NOT NULL CHECK (content_type IN ('blog', 'social', 'email', 'ad')),
  platforms TEXT[],
  structure JSONB NOT NULL,
  placeholders TEXT[],
  created_by UUID REFERENCES users(id) ON DELETE CASCADE,
  is_public BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### performance_metrics
```sql
CREATE TABLE performance_metrics (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  content_id UUID REFERENCES content(id) ON DELETE CASCADE,
  platform TEXT NOT NULL,
  impressions INTEGER DEFAULT 0,
  engagements INTEGER DEFAULT 0,
  clicks INTEGER DEFAULT 0,
  shares INTEGER DEFAULT 0,
  comments INTEGER DEFAULT 0,
  likes INTEGER DEFAULT 0,
  conversions INTEGER DEFAULT 0,
  recorded_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Row Level Security (RLS) Policies

Supabase RLS policies enforce data access control:

- **users**: Users can read all users, update only their own profile
- **campaigns**: Users can read campaigns they're associated with; Marketers can create/update/delete
- **content**: Users can read content from their campaigns; Creators can create; Editors can update; Marketers can delete
- **assets**: Users can read assets from their campaigns; all authenticated users can upload
- **templates**: All users can read public templates; users can CRUD their own templates
- **approval_requests**: Approvers can read requests assigned to them; submitters can read their own requests
- **performance_metrics**: Read-only for all authenticated users

### Storage Buckets

Supabase Storage buckets:

- **assets**: Stores uploaded media files (images, videos, documents)
  - Public read access for published content assets
  - Authenticated write access with file size limits
  - Automatic thumbnail generation for images

- **exports**: Stores generated reports and exports
  - Private access, user-specific folders
  - Automatic cleanup after 30 days


## Correctness Properties

A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.

### Property Reflection

After analyzing all acceptance criteria, I identified several areas of redundancy:

- Authentication properties (1.1-1.5) can be consolidated into fewer comprehensive properties
- Platform adapter properties (3.2-3.5) are edge cases covered by the general platform formatting property (3.1)
- Multiple AI analysis properties share similar patterns (sentiment, engagement, A/B testing) and can be streamlined
- Campaign and content management properties have overlapping data persistence concerns
- Several UI display properties are not automatically testable and should be validated through manual testing

The following properties represent the unique, testable correctness guarantees after eliminating redundancy:

### Authentication and Authorization Properties

Property 1: Valid credentials authenticate successfully
*For any* valid user credentials, authentication via Supabase Auth should succeed and return a user session with one of the three valid roles (Marketer, Creator, Editor).
**Validates: Requirements 1.1, 1.2**

Property 2: Invalid credentials are rejected
*For any* invalid credentials (wrong password, non-existent user, malformed input), authentication should fail and return a descriptive error message.
**Validates: Requirements 1.1**

Property 3: Role-based access control enforces permissions
*For any* user and protected resource, access should be granted if and only if the user's role has the required permissions for that resource.
**Validates: Requirements 1.3**

Property 4: Expired sessions require re-authentication
*For any* expired session, attempts to access protected resources should be rejected and require re-authentication.
**Validates: Requirements 1.4**

Property 5: Session state persists across navigation
*For any* authenticated user session, navigating between pages should maintain the session state without requiring re-authentication.
**Validates: Requirements 1.5**

### AI Content Generation Properties

Property 6: Valid prompts generate content
*For any* valid content prompt and content type (blog, social, email, ad), the AI_Engine should generate content using Groq's Llama 3.1 70B model.
**Validates: Requirements 2.1, 2.5**

Property 7: Invalid inputs return descriptive errors
*For any* invalid AI_Engine input (empty prompt, unsupported content type, malformed request), the system should return a descriptive error message.
**Validates: Requirements 2.3**

Property 8: Regeneration produces new content
*For any* generated content, regenerating with a modified prompt should produce different content than the original.
**Validates: Requirements 2.4**

### Content Adaptation Properties

Property 9: Platform adapters format content correctly
*For any* content and target platform (Twitter, Instagram, LinkedIn, Facebook), the Content_Adapter should format the content according to platform-specific requirements (character limits, structure, tone).
**Validates: Requirements 3.1, 3.2, 3.3, 3.4, 3.5**

Property 10: Oversized content is truncated appropriately
*For any* content exceeding platform limits, the Content_Adapter should provide truncation suggestions that preserve message integrity.
**Validates: Requirements 3.6**
