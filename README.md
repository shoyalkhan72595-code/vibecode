VIBECODER — COMPREHENSIVE PRODUCT SPECIFICATION
Cloud-Native SaaS AI Coding Assistant (Mobile-First)
Version 3.1 — Terminal & Preview Updated (Free-Tier Compatible)
---
> ## ⚠️ CRITICAL BUILD INSTRUCTION — READ THIS FIRST BEFORE WRITING ANY CODE
>
> **Firebase aur Payment integration abhi implement mat karna.**
>
> ### Firebase (Firestore, Firebase Storage, Firebase Auth, Firebase Hosting):
> - Sari UI screens, components, file structure, folder names, environment variable placeholders — **sab banao bilkul spec ke hisaab se.**
> - Lekin **koi bhi actual Firebase API call mat karna.** Jahan bhi Firebase call hoti (data fetch, save, auth, preview deploy), wahan **mock data ya in-memory state use karo.**
> - Firebase environment variable fields ko `.env` mein blank hi rakho — koi value mat daalo.
> - Har Firebase call ki jagah ek comment likho: `// TODO: Wire up Firebase — credentials will be added manually later`
> - Example: Agar chat messages Firestore se load hone chahiye, to abhi ek hardcoded mock array return karo. UI exactly waise hi dikhna chahiye jaise real data hoti.
>
> ### Payments (Razorpay aur Stripe):
> - Subscription UI, billing screens, upgrade buttons, invoice list, plan badges — **sab bilkul spec ke hisaab se banao.**
> - Lekin **koi bhi actual payment processing logic implement mat karo.** Sare payment functions stubbed rakho.
> - Example stub:
>   ```js
>   // TODO: Integrate Razorpay/Stripe — API keys will be added manually later
>   async function createSubscription() { return { status: 'stub' }; }
>   ```
> - Payment environment variable fields ko `.env` mein blank hi rakho.
>
> ### Baaki sab kuch:
> - **Poora project fully functional banana hai** — sirf Firebase aur payments ko stub karna hai.
> - Streaming, terminal, file explorer, code editor, agent mode, preview UI, settings — **sab kuch kaam karna chahiye.**
> - Jab project complete ho jaye, owner manually Firebase credentials aur payment keys add karke wire-up karega.
---
1. PRODUCT OVERVIEW
VibeCoder is a mobile-first, cloud-hosted SaaS AI coding assistant — similar to Cursor or Windsurf — that allows users to write software through natural language AI interaction. Users chat with AI, the AI creates and edits files, manages a project workspace, runs terminal commands, and shows live previews — all from a mobile browser with no local device execution required.
Target Users: Mobile developers, indie hackers, students, vibe coders who build from Android/iOS browsers.
Core Philosophy:
AI-first workflow: everything starts from a chat prompt
Mobile-first UI: thumb-friendly, bottom-nav, gesture-based
Cloud-hosted execution: no Termux, no local device required
Real-time streaming: all AI responses and operations streamed live
Clean, stable, professional: stability and speed over flashy effects
Deployment Stack:
Frontend: Next.js → hosted on Vercel
Backend: Node.js (Express/Fastify) → hosted on Railway or Fly.io (free tier, no cold start)
Storage: Firebase (Firestore + Firebase Storage)
Database: Firestore (primary)
Auth: NextAuth.js or Firebase Auth
Task Queue: Upstash Redis + BullMQ (background jobs, free tier)
Preview: Firebase Hosting Preview Channels (free, no Docker needed)
Payments: Razorpay + Stripe (modular, switchable)
Why Railway/Fly.io over Render:
Render free tier spins down after 15 min inactivity causing ~60s cold starts — fatal for SSE streaming.
Railway gives $5/month free credit with always-on containers. Fly.io free tier gives 3 shared VMs with no spin-down.
---
2. ARCHITECTURE OVERVIEW
2.1 Separation of Concerns
Frontend Responsibilities (Next.js on Vercel):
Chat UI with real-time streaming rendering
File Explorer UI (read/write via API)
Monaco-based Code Editor
Preview UI (iframe embedding backend preview URLs)
Terminal UI (displays streamed logs from backend)
Settings screens
Auth screens (login, signup, dashboard)
Subscription/billing UI
State management (Zustand or Redux Toolkit)
Offline cache (IndexedDB/localStorage for read-only access)
Mobile interactions and gestures
Bottom navigation dock
Backend Responsibilities (Node.js on Railway/Fly.io):
AI engine (API calls to OpenRouter/NVIDIA/custom providers)
Streaming engine (SSE relay to frontend)
Agent engine (multi-step planning and execution)
Filesystem manager (per-user, per-project workspace isolation)
Whitelisted command executor (npm, git, node, python — NO arbitrary execution)
Preview builder (runs build command, deploys to Firebase Hosting Preview Channel)
GitHub integration
Context/token management
Background task manager (Upstash Redis + BullMQ)
Security validation layer
Usage tracking engine
Admin engine
2.2 Module Architecture
Every major system must be a fully separated, independently testable module:
```
/backend
  /api              → Express routes
  /engines
    /ai-engine        → AI API calls, model routing, provider switching
    /agent-engine     → Agent mode planning and step execution
    /stream-engine    → SSE streaming, reconnect, chunk management
    /preview-engine   → Firebase Hosting preview channel deploy/manage
    /command-executor → Whitelisted command runner (npm/git/node/python only)
  /managers
    /filesystem-manager → Workspace CRUD, safe paths, snapshots
    /context-manager    → Token budgeting, relevant file selection
    /task-manager       → BullMQ job queue, progress, cancellation
    /storage-manager    → Firebase read/write abstraction
    /security-manager   → Command whitelist validation, workspace boundary checks
  /saas
    /auth-service         → JWT, sessions, OAuth
    /subscription-service → Plan management, limits
    /usage-service        → Token/request/storage tracking
    /billing-service      → Razorpay/Stripe integration
    /admin-service        → Admin panel data and controls
  /utils
    /logger
    /error-handler
    /validators
    /sanitizers

/frontend
  /app               → Next.js App Router pages
  /components
    /chat            → ChatScreen, MessageBubble, StreamRenderer, CodeBlock
    /explorer        → FileTree, FileItem, SlideOverExplorer
    /editor          → MonacoEditor, EditorTabs, SearchInFile
    /preview         → PreviewFrame, PreviewStatus, PreviewControls
    /terminal        → TerminalView, CommandInput, LogStream
    /settings        → APISettings, EditorSettings, AISettings, StorageSettings
    /auth            → LoginScreen, SignupScreen, ForgotPassword
    /dashboard       → UsageDashboard, BillingPanel, InvoiceList
    /admin           → AdminPanel, UsersTable, MetricsView
    /shared          → StatusBar, BottomNav, LoadingStates, ErrorBoundary
  /engines
    /stream-client   → Frontend SSE consumer, chunk renderer
    /offline-cache   → IndexedDB manager for offline viewing
  /state             → Zustand store slices
  /services          → API call wrappers (never raw fetch in components)
  /hooks             → Custom React hooks
  /utils
```
No circular dependencies. No component should call raw fetch directly — all API communication goes through `/services/`.
---
3. AUTHENTICATION SYSTEM
3.1 Auth Methods
Email + Password (with email verification)
Google OAuth
GitHub OAuth
Forgot Password + Reset Password flow
Persistent session management (JWT + refresh tokens)
3.2 Session Behavior
Sessions persist across devices
On login, user's projects, chats, settings, and usage automatically load
Session restore on app open (crash recovery)
Secure HttpOnly cookies for tokens
3.3 Auth Screens Required
`/login` — Email/password + social login
`/signup` — Registration with plan selection
`/forgot-password` — Send reset email
`/reset-password` — Token-based reset form
`/verify-email` — Email verification landing
3.4 Environment Variables Required
```env
NEXTAUTH_SECRET=
NEXTAUTH_URL=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=
JWT_SECRET=
JWT_REFRESH_SECRET=
```
---
4. SUBSCRIPTION & PLAN SYSTEM
4.1 Plans (Database-Driven, Not Hardcoded)
All plan limits stored in Firestore `plans` collection so they can be modified without code changes.
Free Plan:
AI Requests: 50/day
Projects: 3 max
Storage: 100 MB (Firebase Storage)
Preview Deploys: 5/day (Firebase Hosting preview channels)
Terminal Commands: 20 whitelisted commands/day
Agent Mode: Disabled
Context Size: 16K tokens
Concurrent background jobs: 1
Pro Plan:
AI Requests: 1000/day
Projects: Unlimited
Storage: 5 GB
Preview Deploys: Unlimited
Terminal Commands: Unlimited whitelisted commands
Agent Mode: Enabled
Context Size: 128K tokens
Priority queue
Concurrent background jobs: 3
Team Plan:
Everything in Pro
Shared workspace support
Team member invitations
Shared projects
Role-based permissions (Owner, Editor, Viewer)
Team usage dashboard
4.2 Plan Features
Upgrade / Downgrade / Cancel subscription
Monthly and yearly billing options
Billing history with downloadable invoices
Proration on plan changes
Grace period on failed payments (3 days)
---
5. PAYMENT SYSTEM
> ⚠️ **STUB ONLY — DO NOT IMPLEMENT:** Saari payment UI banao (checkout screen, plan cards, billing history table, upgrade buttons) lekin koi bhi actual Razorpay/Stripe API call mat karo. Har payment function mein sirf ek `// TODO: Add Razorpay/Stripe — will be wired up manually` comment rakho aur stub response return karo. Environment variables blank rakho.
5.1 Supported Providers
Razorpay (primary for Indian users)
Stripe (international users)
Payment provider is selected based on user geography or user preference. Architecture is modular — providers can be switched or added without rewriting billing logic.
5.2 Required Features
Checkout flow (monthly/yearly)
Webhook handling (payment success, failure, refund, cancellation)
Invoice generation and email delivery
Subscription pause/resume
Failed payment retry with notification
5.3 Environment Variables (Blank Placeholders)
```env
RAZORPAY_KEY_ID=
RAZORPAY_KEY_SECRET=
RAZORPAY_WEBHOOK_SECRET=
STRIPE_SECRET_KEY=
STRIPE_PUBLISHABLE_KEY=
STRIPE_WEBHOOK_SECRET=
STRIPE_PRICE_ID_PRO_MONTHLY=
STRIPE_PRICE_ID_PRO_YEARLY=
```
---
6. USER DASHBOARD
6.1 Dashboard Sections
Current Plan badge + expiry date
Upgrade button (if on Free plan)
Usage Statistics panel:
AI Requests used today / limit
Token usage this month
Storage used / limit
Preview deploys used today / limit
Terminal commands used today / limit
Agent runs this month
Billing Status (active, trialing, past_due, cancelled)
Invoice History table (date, amount, status, download PDF)
Referral section (invite link, credits earned)
6.2 Usage Tracking (Backend)
Track per-user, per-day, per-month:
`aiRequestsCount`
`promptTokensUsed`
`completionTokensUsed`
`totalTokensUsed`
`projectCount`
`storageUsedBytes`
`previewDeploysCount`
`terminalCommandsCount`
`agentRunsCount`
`githubExportsCount`
Usage updated in real-time after each operation. Limits enforced before operation starts — never after.
---
7. ADMIN PANEL
7.1 Admin Routes
Protected by admin role check. Separate `/admin` route group.
7.2 Admin Sections
Users: List all users, search, filter by plan, block/unblock, impersonate
Subscriptions: Active subscriptions, revenue metrics, churn rate
Usage Metrics: Platform-wide token usage, popular models, peak hours
Error Logs: Centralized error tracking, stack traces, user context
Announcements: Push system-wide banners to all users or specific plans
Plan Management: Edit plan limits, prices, feature flags without code deploy
System Health: Service uptime, API response times, queue depth
---
8. MULTI-TENANT ARCHITECTURE
Every user's data is fully isolated at the database and storage level:
Projects namespaced by `userId/projectId`
Firebase Storage paths: `users/{userId}/projects/{projectId}/...`
Firestore collections: `users/{userId}/projects`, `users/{userId}/chats`
API keys stored encrypted, per user
No cross-user data leakage possible at backend query level
Rate limiting applied per userId, not per IP
---
9. FIREBASE STORAGE INTEGRATION
> ⚠️ **STUB ONLY — DO NOT IMPLEMENT:** Saari Firebase-related services, managers, aur collection schemas ka code structure banao lekin koi bhi actual Firebase SDK call mat karo. Jahan bhi Firestore ya Firebase Storage call hoti, wahan mock/in-memory data return karo aur ek `// TODO: Wire up Firebase — credentials will be added manually later` comment rakho. Environment variables blank rakho.
9.1 Firebase Services Used
Firestore: Users, projects, chats, messages, snapshots, settings, usage, subscriptions, billing records, referrals
Firebase Storage: Project files, file snapshots, exports, uploaded assets
9.2 Firestore Collections Schema
```
users/{userId}
  - email, name, plan, createdAt, lastLoginAt, referralCode

users/{userId}/projects/{projectId}
  - name, description, framework, createdAt, updatedAt, snapshotCount

users/{userId}/projects/{projectId}/chats/{chatId}
  - title, createdAt, updatedAt, messageCount

users/{userId}/projects/{projectId}/chats/{chatId}/messages/{messageId}
  - role, content, timestamp, streamComplete, filesModified[]

users/{userId}/projects/{projectId}/snapshots/{snapshotId}
  - createdAt, label, fileList[], triggerChatId

users/{userId}/settings
  - apiProvider, apiKey (encrypted), model, theme, fontSize, etc.

users/{userId}/usage/{YYYY-MM-DD}
  - aiRequests, tokens, previewRuntime, terminalRuntime, agentRuns

users/{userId}/subscription
  - plan, status, currentPeriodStart, currentPeriodEnd, provider, customerId

plans/{planId}
  - name, aiRequestsPerDay, maxProjects, storageGB, etc.
```
9.3 Environment Variables
```env
FIREBASE_API_KEY=
FIREBASE_AUTH_DOMAIN=
FIREBASE_PROJECT_ID=
FIREBASE_STORAGE_BUCKET=
FIREBASE_APP_ID=
FIREBASE_MESSAGING_SENDER_ID=
FIREBASE_SERVICE_ACCOUNT_JSON=
```
---
10. RATE LIMITING
Rate limiting applied in backend middleware before any operation
Limits read from Firestore `plans` collection dynamically
Redis used for real-time rate limit counters (requests/minute, requests/day)
When limit hit: return HTTP 429 with clear message, remaining quota, reset time
UI shows friendly error: "Daily limit reached. Upgrade to Pro for more."
Premium users get priority queue position in BullMQ
---
11. REFERRAL SYSTEM
Each user gets a unique referral code on signup
Referral tracking: `referredBy`, `referralsCount`, `creditsEarned`
Rewards: extra AI credits, bonus storage, extended trial
Invite link shareable from dashboard
Architecture ready even if rewards UI is not immediately active
---
12. CORE UI/UX SPECIFICATION
12.1 Design Language
Dark theme (primary: `#0A0A0F`, surface: `#111118`, accent: `#6366F1` indigo)
Typography: Inter for UI, JetBrains Mono for code
Border radius: 8–12px (consistent, not over-rounded)
Spacing: 8px grid system
Shadows: soft, single-layer, no heavy drop shadows
Animations: 200ms ease transitions max
NO heavy glassmorphism, NO extreme neon colors, NO RGB effects
Professional and minimal — inspired by Linear, Vercel dashboard, VS Code
12.2 Bottom Navigation Dock
Primary navigation lives at the bottom of the screen (thumb-friendly):
```
[ Chat ] [ Files ] [ Preview ] [ Terminal ] [ Settings ]
```
Active tab highlighted with accent color
Icons with labels
Safe area inset aware (iOS home bar, Android nav bar)
Dock height: 60px minimum touch target
12.3 Floating AI Status Bar
Persistent bar at the top of screen showing live AI state:
```
[ ● Thinking... ] or [ ● Editing files... ] or [ ● Running npm install... ] or [ ✓ Preview ready ]
```
Dismissible after task completes
Tapping it expands to show full action log
Color-coded: blue = working, green = done, red = error
12.4 Responsive Layout
Mobile: single-panel, tab-based navigation
Tablet (optional): split-panel (chat left, editor right)
No forced desktop layout on mobile
Min touch target: 44×44px everywhere
---
13. CHAT SECTION
13.1 Chat Screen Layout
```
┌─────────────────────────────────────┐
│ [Project Name]  [Model]  [● Status] │  ← Top bar
├─────────────────────────────────────┤
│                                     │
│  [Chat messages area]               │  ← Scrollable, auto-scroll
│  [AI action logs inline]            │
│  [Streaming response]               │
│                                     │
├─────────────────────────────────────┤
│ [📎] [🎤] [Input field...] [▶Send] │  ← Bottom input bar
└─────────────────────────────────────┘
```
13.2 Chat Features
Create new chat / delete existing chats
Chat list sidebar accessible via swipe or button
Chat titles auto-generated from first prompt
Each chat tied to a project
Attach files, images, paste code snippets
Voice input button (Web Speech API)
Messages persist in Firestore
Offline: read past chats, no new generation
13.3 Code Blocks in Chat
Each code block in AI responses must have:
Syntax highlighting (Prism.js or Shiki)
Copy button
"Apply to file" button (writes to existing file)
"Open in editor" button (opens in code editor tab)
"Create new file" button
Diff preview before apply: shows what changes will be made
Language label displayed
13.4 AI Action Log (Inline in Chat)
When AI is working, show inline action steps:
```
  ✓ Reading project structure
  ✓ Planning changes
  ▶ Creating src/components/Button.jsx...
  ▶ Editing src/App.jsx...
  ○ Running npm install...
```
These are visible in chat, scrollable, collapsible after task completes.
13.5 Keep / Undo Bar
After AI completes a task and modifies files, show a persistent bar above the input:
```
[ AI modified 4 files ]  [ ✓ Keep ]  [ ↩ Undo ]
```
"Keep": Saves a snapshot of current file state to Firestore, marks chat as committed
"Undo": Restores files to state before AI made changes (using previous snapshot)
This bar disappears after user makes a choice
13.6 Multi-File Modification Approval
Before AI modifies multiple files (≥3), show approval popup:
```
┌────────────────────────────────────┐
│ AI wants to modify 5 files:        │
│  • src/App.jsx                     │
│  • src/components/Header.jsx       │
│  • src/styles/globals.css          │
│  • package.json                    │
│  • README.md                       │
│                                    │
│  [ Approve ]      [ Reject ]       │
└────────────────────────────────────┘
```
---
14. FILE EXPLORER
14.1 Explorer Access
NOT a permanently visible sidebar on mobile
Accessible as a slide-over panel from left edge swipe or "Files" tab
Smooth 250ms slide-in animation
Semi-transparent backdrop, tap outside to close
14.2 Explorer Features
Folder collapse/expand with animation
File type icons (based on extension)
Search files (fuzzy search, real-time)
Recent files section at top
Active file highlight
Unsaved changes dot indicator
Tab system (open files as tabs in editor)
File creation animation:
New file: glows green for 1 second on first appear
Edited file: subtle left-border accent color pulse
Deleted file: fade-out animation before removal
14.3 Long-Press Context Menu (per file/folder)
```
[ Rename ] [ Duplicate ] [ Delete ] [ Copy Path ] [ Copy Code ] [ Move ]
```
Confirmation dialog for delete operations
Deleted files go to Trash (recoverable for 24 hours)
14.4 VS Code-Inspired Structure
File tree indentation consistent with VS Code style
Chevron expand/collapse on folders
Breadcrumb path display in editor
Support for deeply nested folder structures
---
15. CODE EDITOR
15.1 Editor Features
Monaco Editor (same engine as VS Code)
Syntax highlighting for all major languages
Line numbers
Word wrap toggle
Auto-scroll to active edit point
Tab system (multiple open files)
Search inside file (Ctrl+F / mobile search button)
Quick file switch
Copy/paste optimized for mobile (no selection issues)
Mobile keyboard safe spacing (input doesn't hide behind keyboard)
15.2 Real-Time File Update Streaming
When AI edits a file, the editor shows the changes streaming in real-time:
```
creating app.js...         (file appears in explorer)
editing index.html...      (file opens, content streams in line by line)
updating package.json...   (diff visible before full write)
```
Uses SSE stream from backend
Partial content visible as AI writes
Editor is read-only while AI is writing (shows "AI editing..." overlay)
After AI done: editor becomes editable again
---
16. STREAMING SYSTEM
16.1 Architecture
Backend sends Server-Sent Events (SSE) to frontend
SSE stream carries: text tokens, file operation events, terminal output, status events
Frontend SSE client handles: partial rendering, auto-scroll, stream interruption recovery
16.2 Stream Event Types
```json
{ "type": "token", "data": "Hello" }
{ "type": "file_create", "data": { "path": "src/App.jsx" } }
{ "type": "file_edit", "data": { "path": "src/App.jsx", "chunk": "..." } }
{ "type": "terminal_output", "data": "Installing packages...\n" }
{ "type": "status", "data": "Thinking" }
{ "type": "status", "data": "Preview ready" }
{ "type": "done", "data": null }
{ "type": "error", "data": { "message": "..." } }
```
16.3 Streaming Requirements
Incremental rendering: each token appended, not full re-render
Auto-scroll follows new content unless user has scrolled up manually
Stream buffering for mobile performance (debounce render at 50ms)
Reconnect: if SSE disconnects, client auto-reconnects with last event ID
Duplicate chunk prevention using event sequence numbers
Abort handling: user can cancel ongoing stream (sends abort signal to backend)
Typing animation: tokens render with subtle cursor blink
16.4 Streaming Interruption Recovery
Backend stores last N chunks in Redis per stream session
On reconnect, client sends `Last-Event-ID`, backend replays missed chunks
If stream is aborted mid-file-write, filesystem manager rolls back partial write
---
17. TERMINAL SECTION
17.1 Architecture (No Docker — Whitelisted Execution)
The terminal does NOT use Docker containers or arbitrary command execution. Instead, backend runs only a predefined whitelist of safe development commands directly in the project workspace using Node.js `child_process`.
Terminal UI lives entirely in frontend (display + input only)
User types command → frontend sends to backend via API
Backend validates command against whitelist → executes inside `/workspace/{userId}/{projectId}/`
Output streamed back to frontend via SSE in real-time
No Docker, no VM, no sandbox container — just safe controlled execution
17.2 Whitelisted Commands (Only These Execute)
```
# Package managers
npm install
npm install <package-name>
npm run dev
npm run build
npm run start
npm run <any-script>
npx <tool>

# Python
pip install -r requirements.txt
pip install <package>
python app.py
python <filename>.py
python -m <module>

# Node.js
node server.js
node <filename>.js

# Git operations
git clone <url>
git status
git add .
git add <file>
git commit -m "<message>"
git push
git pull
git log --oneline
git diff

# Filesystem (workspace-only)
ls
ls -la
mkdir <dirname>
cat <filename>
echo <text>
pwd
```
Any command NOT in this list is rejected immediately with a clear error message: "Command not allowed. VibeCoder only supports safe development commands."
17.3 Terminal UI
Mobile-friendly: large touch targets, scrollable output
Command input bar at bottom with send button
Quick-command chips above input for common commands:
```
  [ npm install ] [ npm run dev ] [ npm run build ] [ git status ]
  ```
Output log area (auto-scroll, selectable text)
Running process indicator with spinner
Cancel button for long-running processes (sends SIGTERM to backend process)
Progress detection: parse npm install output and show percentage bar
17.4 Security: Command Validation Layer
Before any command executes on backend:
Whitelist check: Command must exactly match an allowed pattern. Any deviation = rejected.
Path validation: All file arguments must resolve within `/workspace/{userId}/{projectId}/`. Path traversal blocked.
Argument sanitization: No shell metacharacters (`; | && || > < $( )` etc.) allowed in arguments. Prevents shell injection.
Workspace boundary lock: Backend spawns process with `cwd` set to project workspace. Process cannot cd outside.
AI-generated command approval: If AI suggests a command to run, show confirmation popup before executing:
```
   ⚠️ AI wants to run:
   npm install express cors dotenv
   [ Approve ]   [ Reject ]
   ```
Process timeout: All commands have a 10-minute maximum. After that, process is killed and user is notified.
What NEVER executes regardless of request:
```
rm -rf /         → blocked
rm -rf ~         → blocked
sudo / su        → blocked
mkfs / dd        → blocked
shutdown/reboot  → blocked
chmod 777 /      → blocked
curl | bash      → blocked
wget | sh        → blocked
Any command with shell injection patterns → blocked
Any path outside workspace → blocked
```
17.5 Background Task System
Long-running whitelisted commands (npm install, git clone, build) run as BullMQ background jobs:
Job added to queue → returns `taskId` immediately, frontend does not wait
Frontend subscribes to task progress via SSE using `taskId`
UI shows: task name, elapsed time, live log stream, cancel button
Task states: `queued → running → completed / failed`
Failed tasks: full error log visible, retry button shown
BullMQ Queue Config (Upstash Redis — Free Tier):
Queues: `command-tasks`, `preview-tasks`, `agent-tasks`
Concurrency: Free plan: 1 concurrent job per user. Pro: 3 concurrent.
Retry: 2 automatic retries on transient failures (network errors etc.)
Timeout: 10 minutes per job maximum
Upstash free tier: 10,000 commands/day (sufficient for small user base at launch)
17.6 Project Environment Auto-Detection
Before running any command, backend reads:
`package.json` → detects framework, available scripts, dependencies
`requirements.txt` / `pyproject.toml` → detects Python project
`Dockerfile` → noted but not executed (Docker not supported)
Backend informs AI of detected project type so AI can suggest correct commands.
---
18. PREVIEW SECTION
18.1 Architecture (Firebase Hosting Preview Channels — Free)
Preview does NOT use Docker containers or isolated VM environments. Instead, backend:
Runs the project's build command inside the workspace (e.g., `npm run build`)
Deploys the build output to a Firebase Hosting Preview Channel
Firebase returns a unique HTTPS preview URL (e.g., `https://vibecoder--preview-abc123.web.app`)
Frontend embeds this URL in an `<iframe>` — fully live, fully HTTPS, no Docker needed
Firebase Hosting Preview Channels — Free Tier:
Available on Spark (free) plan
Each preview gets a unique subdomain URL
Preview channels expire after 7 days (auto-deleted)
10GB free hosting storage, sufficient for many projects
No compute cost — just static file hosting
18.2 Supported Project Types
Project Type	Preview Works?	How
HTML / CSS / JS	✅ Yes	Deploy directly, no build needed

React (Vite/CRA)	✅ Yes	`npm run build` → deploy `/dist` or `/build`
Vue.js	✅ Yes	`npm run build` → deploy `/dist`
Svelte / SvelteKit (static)	✅ Yes	Build → deploy
Next.js (static export)	✅ Yes	`next build && next export` → deploy `/out`
Vanilla JS with bundler	✅ Yes	Build → deploy
Node.js server	⚠️ No	Backend-only, no preview (show message)
Python Flask/FastAPI	⚠️ No	Backend-only, no preview (show message)
Full-stack (frontend + backend)	⚠️ Partial	Frontend preview only
When project type doesn't support preview, show clear message:
```
ℹ️ Preview not available for backend-only projects.
   Your files are ready — run "node server.js" in the terminal.
```
18.3 Preview Flow
```
User clicks "Run Preview"
        ↓
Backend reads project type from package.json
        ↓
Backend runs build command in workspace
(npm run build / npx vite build / etc.)
        ↓
Build output detected (/dist, /build, /out, or root for HTML)
        ↓
Backend deploys to Firebase Hosting Preview Channel
firebase hosting:channel:deploy preview-{projectId}-{timestamp}
        ↓
Firebase returns preview URL
        ↓
Backend streams URL to frontend via SSE
        ↓
Frontend embeds URL in <iframe>
Preview ready ✓
```
18.4 Preview Trigger
Preview ONLY updates when:
User explicitly clicks "Run Preview" button
AI completes a task and emits a `preview_ready` stream event
User manually clicks reload in preview controls
Preview does NOT auto-refresh on every file save or AI token. This is intentional.
18.5 Preview Loading States
Must always show one of these — never a blank white screen:
```
○ Detecting project type...
○ Building project...         (shows build log stream)
○ Deploying preview...
✓ Preview ready               (iframe loads URL)
✗ Build failed — [View logs] [Fix with AI] [Retry]
✗ Preview type not supported — [View message]
```
Build log streams in real-time during the build step so user sees progress.
18.6 Preview Controls
Reload button — re-deploys latest files to same preview channel
Open in browser button — opens preview URL in external browser tab
Build logs panel — collapsible, shows last build output
Viewport selector — Mobile (375px) / Tablet (768px) / Desktop (100%)
Preview URL — copyable, shareable link (valid for 7 days)
Preview status — Building / Ready / Failed / Expired
18.7 Preview URL Management
Each project maintains its latest preview channel URL in Firestore
URL stored at: `users/{userId}/projects/{projectId}.previewUrl`
On preview tab open: if URL exists and not expired, load immediately without rebuilding
If URL expired (>7 days): prompt user to re-run preview
Preview channel name pattern: `preview-{projectId}` (consistent, overwrites old channel)
18.8 Error Auto-Detection
Build errors (from npm run build stdout/stderr) automatically captured
If build fails, "Fix with AI" button appears in preview screen:
```
  ⚠️ Build failed
  ERROR in src/App.jsx: Cannot find module './components/Header'
  [ Fix with AI ]   [ View full log ]   [ Retry ]
  ```
Clicking "Fix with AI" sends full error log to chat as user message
AI reads error and proposes fix automatically
18.9 Preview in Agent Mode
When Agent Mode runs a full task:
Agent creates/edits all required files
Agent triggers build + preview deploy as final step
Preview URL streamed to frontend
Preview tab automatically activates and shows result
Keep/Undo bar appears in chat tab
---
19. SETTINGS SECTION
19.1 Settings Categories
API Settings:
Select provider: OpenRouter / NVIDIA / OpenAI-compatible / Custom
Custom Base URL field
API Key input (masked, show/hide toggle)
Model selection dropdown (fetched from provider's models endpoint)
API Test Button: Makes a minimal test call, shows: ✓ Connected (model name) or ✗ Failed (error message)
Save / Reset buttons
Editor Settings:
Font size slider (12–20px)
Theme: Dark / Light / High Contrast
Word wrap toggle
Tab size: 2 / 4 spaces
Auto-save toggle
Line numbers toggle
AI Settings:
Temperature slider (0.0–1.0)
Context window size (based on plan)
Streaming toggle (on/off)
Agent Mode toggle (Pro+ only)
Context mode: Auto / Selected files only / Full project
Storage Settings:
Storage used / limit progress bar
Export all projects as ZIP
Clear cache button
Restore session button
Auto-snapshot interval setting
Account Settings:
Profile (name, email, avatar)
Change password
Connected accounts (Google, GitHub)
Danger zone: Delete account
About:
App version
Changelog
Terms of Service
Privacy Policy
---
20. AGENT MODE
20.1 Overview
Agent Mode is an advanced autonomous workflow where AI plans and executes multi-step tasks independently.
20.2 Agent Workflow
User provides high-level goal: "Build a React todo app with local storage"
AI shows plan:
```
   Plan:
   1. Analyze project structure
   2. Create component files
   3. Install dependencies
   4. Write application logic
   5. Run preview
   6. Fix any errors
   ```
User approves plan
AI executes step-by-step, streaming progress to UI
After each step, status shown in floating bar and action log
On any error: AI auto-attempts fix (configurable: auto-fix or ask user)
On completion: preview shown, keep/undo bar appears
20.3 Agent Execution
Agent runs as a background BullMQ job on backend
Frontend subscribes to agent's SSE stream
Agent can: read files, write files, run terminal commands, call API, run preview
Agent cannot: access outside workspace, run dangerous commands, delete project root
20.4 Agent Live Status (Floating Bar)
```
[ ● Agent: Creating components (Step 3 of 6) ]
```
Expandable to full step log.
---
21. CONTEXT MANAGEMENT
AI is NOT given the entire project blindly
Context manager selects relevant files based on:
Currently open file in editor
Files referenced in prompt
Recently modified files
`package.json`, `requirements.txt`, `README.md` always included
Files explicitly selected by user
Conversation summarization for long chats (rolling summary when token budget exceeded)
Duplicate context removed before each API call
User can see "Context used: 12K / 128K tokens" in settings or chat header
Selected context mode in settings: Auto / Manual selection / Full project
Built-in Docs Reader
AI automatically reads and uses:
`README.md`
`package.json` / `requirements.txt` / `pyproject.toml`
Any `.md` files in `/docs` folder
Framework-specific config files (`next.config.js`, `vite.config.ts`, etc.)
---
22. PROJECT MANAGEMENT
22.1 Home / Project Screen
Default landing after login:
Recent projects as cards:
```
  ┌────────────────────────────────┐
  │  [Project Icon]                │
  │  My Todo App                   │
  │  Last edited: 2 hours ago      │
  │  React • 12 files              │
  │  [ Continue ]  [ ⋯ Options ]  │
  └────────────────────────────────┘
  ```
Create new project button (prominent)
Search projects
Sort by: recent / name / size
22.2 Project Options (Long Press / ⋯ Menu)
Rename
Duplicate
Export as ZIP
Export to GitHub
Delete (with confirmation)
View snapshots
---
23. SNAPSHOT & VERSION SYSTEM
Auto-snapshot taken every 10 minutes during active editing
Snapshot taken on: "Keep" button press, before agent mode runs, before any AI multi-file modification
Snapshots stored in Firebase Storage under `users/{userId}/projects/{projectId}/snapshots/`
Snapshot metadata in Firestore: timestamp, label, triggering chat ID, file list
Snapshot viewer: list of snapshots with timestamps, restore button, diff preview
Restore: replaces current workspace files with snapshot (with confirmation)
Trash system: deleted files moved to trash folder, recoverable for 24 hours
Max snapshots: 20 (Free), 100 (Pro), FIFO rotation
---
24. PROMPT HISTORY & REUSABLE PROMPTS
All prompts automatically saved per user
Prompt history screen: searchable list of past prompts
Favorite/star prompts for quick reuse
Prompt templates: user-created reusable prompt snippets
Access via: long-press on input field → "Prompt History" / "Templates"
---
25. GITHUB INTEGRATION
25.1 Features
Connect GitHub account (OAuth)
One-click export: push current project to new GitHub repo
Push changes to existing connected repo
`git clone` a repo into a new project (via terminal or clone dialog)
Auto-generate `.gitignore` based on project type
25.2 Export Flow
```
[ Export to GitHub ]
  → Select: New repo / Existing repo
  → Set repo name, visibility (public/private)
  → Commit message input
  → [ Push ] button
  → Progress: Preparing... Pushing... Done ✓
  → Link to GitHub repo shown
```
25.3 ZIP Export
Always available regardless of GitHub connection
Generates ZIP of entire project workspace on backend
Returns download link (temporary signed Firebase Storage URL)
One-click download from any project
---
26. OFFLINE MODE
26.1 What Works Offline
View past chat history
Read project files (from IndexedDB cache)
View file explorer
Read previous preview snapshots (static HTML cache)
View settings
26.2 What Requires Connectivity
AI generation (all providers require internet)
Terminal execution (backend sandbox)
Preview creation
File sync
Auth operations (except reading cached session)
26.3 Offline UI Behavior
Banner shown: "You're offline — viewing cached data"
Disabled buttons are clearly grayed out with tooltip: "Requires internet"
App opens and loads cached state even with zero connectivity
On reconnect: auto-sync pending writes, resume interrupted streams
---
27. CRASH RECOVERY SYSTEM
27.1 On App Crash / Close
App state (active project, open files, scroll position, active chat) saved to IndexedDB every 30 seconds
On next open: detect saved state, show:
```
  ┌──────────────────────────────────┐
  │  Restore previous session?       │
  │  Project: My Todo App            │
  │  Last active: 5 minutes ago      │
  │                                  │
  │  [ Restore ]      [ Start fresh ]│
  └──────────────────────────────────┘
  ```
27.2 Recovery Targets
Terminal session: reconnect to background task if still running
Preview: reload last preview URL if still live
Chat: restore scroll position and message history
Editor: restore open tabs and cursor positions
Unsaved changes: restore from IndexedDB buffer
---
28. SECURITY MODEL
28.1 Workspace Isolation
Every user's project files live in: `/workspace/{userId}/{projectId}/`
AI and terminal engine can ONLY operate within this path
Path traversal prevention: all paths normalized and validated before any filesystem operation
Symlink abuse prevention: symlinks not followed outside workspace
28.2 Safe Mode (Default ON)
Enabled by default on every session
Dangerous commands blocked (see Terminal section)
External storage inaccessible
System paths inaccessible
User can toggle off in advanced settings (with warning)
28.3 AI Command Approval
High-risk AI-generated commands require explicit user approval before execution. This is NOT optional and cannot be disabled.
28.4 API Key Security
API keys encrypted before storage (AES-256 or Firebase encryption)
Keys displayed masked: `sk-...xxxxx`
Show/hide toggle for verification
Keys never logged, never sent to third parties, never stored in plaintext
Keys stored per-user in encrypted Firestore field
28.5 Preview Security (Firebase Hosting)
Preview URLs are public but unguessable (random channel ID in subdomain)
Preview contains only the build output — no server-side logic, no database access
Preview channel is scoped to one project; no cross-user data possible
Firebase Hosting serves static files only — no code execution in preview environment
Preview channel expires automatically after 7 days
If user deletes project, all associated preview channels are also deleted via backend cleanup
28.6 Backend Security
All API routes authenticated (JWT middleware)
Admin routes require `role: admin` in JWT
Rate limiting at API gateway level (Redis-backed)
Input sanitization on all user-supplied data
CORS: only frontend domain whitelisted
---
29. PERFORMANCE & OPTIMIZATION
29.1 Frontend Performance
Lazy loading: heavy components (Monaco Editor, Preview iframe) loaded on first access
Virtualized lists: chat messages, file list use virtual scrolling (react-window)
Debounced events: search, resize, scroll handlers debounced at 100–200ms
Minimal rerenders: Zustand store slices, React.memo where appropriate
Tab suspension: inactive tabs (Preview, Terminal) suspend heavy processes
Image optimization: Next.js `<Image>` component throughout
Bundle splitting: each tab/section in separate Next.js chunk
29.2 Mobile RAM Optimization
Large files: lazy loaded in editor (only visible portion rendered)
Preview cache: max 3 snapshots kept in memory
Background process limits: Free plan 1 background job, Pro 3
Auto memory cleanup: listeners removed on component unmount
Unused preview/terminal processes terminated after 5 min idle
29.3 Backend Performance
Incremental file scanning for large projects
Background metadata generation (file index, project structure)
Smart context: only relevant files included per request
Redis caching: model list, user plan data, rate limit counters
29.4 File System Safety
Atomic file writes: write to temp file → rename (prevents corruption)
Write buffers: batch small writes before flushing to Firebase Storage
Auto-backup before any destructive operation
Safe recovery: if write fails mid-operation, restore from buffer
---
30. MOBILE GESTURES & INTERACTIONS
Left edge swipe → right: Opens file explorer slide-over
Long press on file: Shows context menu (rename, delete, duplicate, copy path)
Long press on message: Copy, select, share options
Swipe left on chat item: Quick delete option
Pull to refresh: Refreshes file list, chat list
Pinch on editor: Adjusts font size
Tap outside explorer: Closes slide-over
Keyboard safe area: Editor and input always above keyboard, no layout shift
---
31. ERROR HANDLING & RECOVERY
Every module must handle errors gracefully. No silent failures.
For every error type, show:
Clear human-readable message (not raw error code)
Retry button where applicable
Log accessible for debugging
Suggested fix or next step
Error Types and Responses:
Error	User Message	Action
API key invalid	"API key rejected. Check your key in Settings."	Link to Settings
Rate limit hit	"Daily limit reached. Upgrade to Pro for more."	Upgrade button
Stream disconnected	"Connection lost. Reconnecting..."	Auto-reconnect
Preview crashed	"Preview crashed. View logs or retry."	Retry + Logs
Terminal command failed	"Command failed. [View error] [Fix with AI]"	AI fix option
File write failed	"Couldn't save file. Retrying..."	Auto-retry 3x
Network offline	"You're offline. Showing cached data."	Banner
Storage full	"Storage limit reached. Delete files or upgrade."	Storage manager
---
32. ENVIRONMENT VARIABLES MASTER LIST
```env
# === DEPLOYMENT ===
NEXT_PUBLIC_APP_URL=
NEXT_PUBLIC_API_URL=
NODE_ENV=production

# === AUTH ===
NEXTAUTH_SECRET=
NEXTAUTH_URL=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=
JWT_SECRET=
JWT_REFRESH_SECRET=
JWT_EXPIRES_IN=7d

# === FIREBASE ===
FIREBASE_API_KEY=
FIREBASE_AUTH_DOMAIN=
FIREBASE_PROJECT_ID=
FIREBASE_STORAGE_BUCKET=
FIREBASE_APP_ID=
FIREBASE_MESSAGING_SENDER_ID=
FIREBASE_SERVICE_ACCOUNT_JSON=

# === FIREBASE HOSTING (for preview channels) ===
# Service account must have Firebase Hosting Admin role
# Same FIREBASE_SERVICE_ACCOUNT_JSON is reused — no extra key needed
FIREBASE_HOSTING_SITE_ID=        # Your Firebase Hosting site name
PREVIEW_CHANNEL_EXPIRY_DAYS=7    # How long preview URLs stay alive

# === REDIS (Upstash — Free Tier) ===
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=

# === PAYMENTS ===
RAZORPAY_KEY_ID=
RAZORPAY_KEY_SECRET=
RAZORPAY_WEBHOOK_SECRET=
STRIPE_SECRET_KEY=
STRIPE_PUBLISHABLE_KEY=
STRIPE_WEBHOOK_SECRET=
STRIPE_PRICE_ID_PRO_MONTHLY=
STRIPE_PRICE_ID_PRO_YEARLY=

# === EMAIL ===
EMAIL_FROM=
RESEND_API_KEY=
# OR
SMTP_HOST=
SMTP_PORT=
SMTP_USER=
SMTP_PASS=

# === GITHUB OAUTH (for project export) ===
GITHUB_EXPORT_CLIENT_ID=
GITHUB_EXPORT_CLIENT_SECRET=

# === ADMIN ===
ADMIN_EMAIL=

# === ENCRYPTION ===
ENCRYPTION_KEY=
ENCRYPTION_IV=

# === COMMAND EXECUTION LIMITS ===
MAX_COMMAND_TIMEOUT_MS=600000    # 10 minutes
MAX_CONCURRENT_JOBS_FREE=1
MAX_CONCURRENT_JOBS_PRO=3
WORKSPACE_BASE_PATH=/workspace   # Base path for all user project workspaces
```
---
33. DEVELOPMENT PHASES (Recommended)
Phase 1: Foundation
Auth system (email, Google, GitHub)
Project and chat CRUD
Firebase storage integration
Basic streaming AI chat (no file ops)
File explorer (read-only)
Basic settings (API key, model)
Bottom navigation dock
Phase 2: Core Coding Features
Code editor (Monaco)
File create/edit/delete via AI
Streaming file operation events
Keep/Undo bar
Multi-file approval popup
ZIP export
Basic whitelisted terminal (npm install, npm run build, ls, cat)
Phase 3: Advanced Features
Full whitelisted terminal (all approved commands + BullMQ background jobs)
Preview system via Firebase Hosting Preview Channels
Agent mode (with build + preview as final step)
Snapshot system
GitHub export
Quick-command chips in terminal UI
Phase 4: SaaS Layer
Subscription plans (Razorpay/Stripe)
Usage tracking dashboard
Rate limiting enforcement
Admin panel
Referral system
Billing history and invoices
Phase 5: Polish
Offline mode
Crash recovery
Gesture system
Performance optimization
Error auto-detection + "Fix this error"
Prompt history and templates
Animations and loading states
---
34. FINAL PRINCIPLES
Stability first, effects later. A working, stable product beats a flashy broken one.
Mobile is the primary target. Every decision tested on a 6-inch phone screen first.
Streaming is non-negotiable. No operation should feel synchronous or blocking.
Security is never optional. Workspace isolation and command whitelist enforced unconditionally.
Clean architecture always. No monolithic files. No circular dependencies. No mixed concerns.
User data is sacred. Encrypted, isolated, never exposed to other users.
Graceful degradation. Every failure state has a recovery path visible to the user.
Performance on low-end Android. If it's smooth on a mid-range phone, it's ready.
No placeholder UI. Every screen must work, even if the feature is queued for later phases.
AI is a tool, not a god. AI can only run whitelisted commands. No arbitrary execution ever.
Free-tier first. Architecture chosen so the entire product runs at $0/month at launch. Scale costs only when revenue justifies it.
35. DESKTOP LAYOUT ARCHITECTURE
35.1 Layout Philosophy
Desktop layout follows a VS Code-inspired multi-panel workspace model. The goal is maximum information density with zero clutter. User should be able to see chat, code, and preview simultaneously without switching tabs.
Desktop layout is a resizable multi-column workspace, not a scaled-up version of mobile.
35.2 Primary Desktop Layout Structure
```
┌──────────────────────────────────────────────────────────────────────────────┐
│  TITLE BAR: [VibeCoder Logo] [Project Name ▾] [Model ▾] [● AI Status]  [⚙] │
├────────┬─────────────────────────────────────────┬───────────────────────────┤
│        │                                         │                           │
│  LEFT  │         CENTER PANEL                   │     RIGHT PANEL           │
│ SIDEBAR│                                         │                           │
│        │  [Editor Tabs ──────────────────────]  │  [Chat / Preview Toggle]  │
│ [📁]   │  ┌──────────────────────────────────┐  │  ┌─────────────────────┐  │
│ [💬]   │  │                                  │  │  │                     │  │
│ [🔍]   │  │   Monaco Code Editor             │  │  │  Chat Messages      │  │
│ [🔀]   │  │                                  │  │  │  or                 │  │
│ [📦]   │  │                                  │  │  │  Live Preview       │  │
│ [⚙]   │  │                                  │  │  │                     │  │
│        │  └──────────────────────────────────┘  │  └─────────────────────┘  │
│        │                                         │  [Input area — Chat mode] │
│        ├─────────────────────────────────────────┤                           │
│        │  TERMINAL PANEL (collapsible)           │                           │
│        │  $ npm run dev ▌                        │                           │
└────────┴─────────────────────────────────────────┴───────────────────────────┘
│ STATUS BAR: [Branch: main] [12 files] [Context: 8K/128K] [Ln 42, Col 18] [●] │
└──────────────────────────────────────────────────────────────────────────────┘
```
35.3 Panel Definitions
Left Sidebar (narrow, icon-only or expanded):
Width: 48px collapsed / 240px expanded
Toggle: click any icon or press `Ctrl+B`
Icons (top to bottom): Explorer, Chat History, Search, Git, Extensions/Plugins, Settings
Active section highlighted with accent color left border
Center Panel (primary workspace):
Takes remaining width after sidebar and right panel
Contains: editor tabs at top, Monaco editor, collapsible terminal at bottom
Default width: ~55% of viewport
Minimum width: 400px (prevents crushing)
Right Panel (chat + preview):
Default width: 380px
Resizable: drag left border to resize (min 300px, max 600px)
Contains toggle: [Chat] [Preview] tabs at top
Chat and Preview are separate tabs in this panel, NOT separate main panels
On desktop, chat input is always visible at bottom of right panel
Terminal Panel (bottom of center):
Default height: 200px
Collapsible: drag top border or press `Ctrl+``
Minimum height: 120px, Maximum: 50% of viewport height
When collapsed: shows only status bar strip with last output line
Status Bar (bottom, full width):
Height: 22px
Shows: git branch, open file count, token context usage, cursor position, AI status dot
Clicking sections opens relevant panel (e.g., clicking "Context" opens context manager)
35.4 Panel Resize System
All panel borders are draggable:
Left sidebar ↔ Center panel: drag vertical divider
Center panel ↔ Right panel: drag vertical divider
Terminal ↔ Editor: drag horizontal divider
Resize persists to localStorage (remembered on next session)
Double-click divider: reset that panel to default size
Minimum sizes enforced so no panel can be accidentally hidden
35.5 Panel Collapse / Expand
```
Keyboard shortcuts:
Ctrl+B          → Toggle left sidebar
Ctrl+`          → Toggle terminal panel
Ctrl+Shift+P    → Toggle right panel (chat/preview)
Ctrl+Shift+E    → Focus explorer in sidebar
Ctrl+Shift+F    → Focus search in sidebar
Ctrl+Shift+G    → Focus git in sidebar
```
All panels remember their last state between sessions.
---
36. DESKTOP NAVIGATION
36.1 Left Sidebar Sections
Explorer (📁)
Full file tree (same as mobile but always visible, not slide-over)
Folder collapse/expand
Right-click context menu (replaces mobile long-press)
Drag and drop files between folders
New file / New folder buttons at top of explorer
File search inline (click magnifier icon inside explorer)
Chat History (💬)
List of all chats for current project
Click to switch chat
Hover to show delete / rename button
New chat button at top
Chat search
Search (🔍)
Global search across all project files
Replace functionality
Filter by file type
Case-sensitive / regex toggles
Results show file path + matching line with highlight
Git (🔀)
Shows changed files (modified, added, deleted)
Stage / unstage individual files
Commit message input + commit button
Push / pull buttons
Branch name + switch branch dropdown
Dependencies (📦)
Reads `package.json` or `requirements.txt`
Lists installed packages with versions
Quick install button: type package name, click install
Outdated packages highlighted
Settings (⚙)
Opens settings panel in right area (replaces right panel temporarily)
36.2 Top Title Bar
```
[VibeCoder] [ProjectName ▾] [ModelName ▾] [● Thinking... / ✓ Ready] [👤 User] [⚙]
```
ProjectName dropdown: Switch between projects without going to home screen
ModelName dropdown: Live model switcher (changes mid-conversation if needed)
AI Status: Always visible, real-time, clickable (opens action log)
User avatar: Dropdown: Dashboard, Settings, Billing, Logout
⚙: Quick settings panel toggle
36.3 No Bottom Navigation on Desktop
Bottom dock (used on mobile) is hidden on desktop. All navigation moves to left sidebar icons and keyboard shortcuts.
---
37. DESKTOP CODE EDITOR ENHANCEMENTS
37.1 Editor Tab System
Multiple files open simultaneously as tabs
Tab bar shows: file icon, file name, unsaved dot indicator
Drag tabs to reorder
Right-click tab: Close, Close Others, Close to the Right, Copy Path, Reveal in Explorer
`Ctrl+W` close active tab
`Ctrl+Tab` cycle through open tabs
`Ctrl+Shift+T` reopen last closed tab
Overflow: tabs scroll horizontally with left/right scroll arrows
37.2 Split Editor (Desktop Only)
Right-click any tab → "Split Right" or "Split Down"
View two files side-by-side (e.g., `.jsx` and its `.css` file)
Diff view: open same file in two panes to compare versions
Maximum 2 split panes (no 3-way split for simplicity)
37.3 Editor Features (Desktop-Specific Additions)
Find & Replace: `Ctrl+F` (find), `Ctrl+H` (replace), `Ctrl+Shift+H` (replace all)
Go to line: `Ctrl+G`
Command palette: `Ctrl+Shift+P` — searchable list of all actions
Multi-cursor: `Ctrl+Click` to place multiple cursors
Column selection: `Shift+Alt+drag` for block selection
Code folding: Click gutter arrows to fold/unfold functions and blocks
Minimap: Right side scrollable minimap of file (toggle in view settings)
Breadcrumb path: Above editor showing file path hierarchy (click to navigate)
Go to definition: `Ctrl+Click` on function/variable name
Hover docs: Hover over symbol shows type information and docs
Format document: `Shift+Alt+F` — auto-format using Prettier
Auto-save: Configurable (immediate / on focus loss / off)
37.4 Editor Context Menu (Right-Click)
```
Cut
Copy
Paste
─────────────────
Format Selection
Apply AI Fix
Send to Chat
─────────────────
Go to Definition
Peek Definition
Find All References
─────────────────
Copy Line Up / Down
Move Line Up / Down
─────────────────
Apply to File
Create New File from Selection
```
37.5 AI Code Lens (Desktop Only)
Small inline actions appear above functions/classes (like GitHub Copilot):
```
  ✨ Refactor  |  📝 Add docs  |  🧪 Write tests  |  🔍 Explain
  function handleSubmit(data) {
```
Clicking any lens action sends pre-filled prompt to chat automatically.
---
38. DESKTOP CHAT PANEL ENHANCEMENTS
38.1 Chat Panel Layout (Right Panel — Chat Tab)
```
┌─────────────────────────────────────┐
│ [Chat] [Preview]    [New] [History] │  ← Tab bar + controls
├─────────────────────────────────────┤
│                                     │
│  Chat messages (scrollable)         │
│  AI action log inline               │
│  Streaming response                 │
│                                     │
├─────────────────────────────────────┤
│ Context: [src/App.jsx ×] [+add]     │  ← Active context pills
├─────────────────────────────────────┤
│ ┌─────────────────────────────────┐ │
│ │ Type a prompt...                │ │  ← Textarea (auto-expands)
│ └─────────────────────────────────┘ │
│ [📎] [🎤] [↑ History] [▶ Send]     │
└─────────────────────────────────────┘
```
38.2 Context Pills (Desktop-Specific Feature)
Above the input, show which files AI has in context:
```
Context: [ 📄 App.jsx × ] [ 📄 Header.jsx × ] [ + Add file ]
```
Click `×` to remove a file from context
Click `+ Add file` to manually pin a file to context
Badge shows token count: `App.jsx (2.1K tokens)`
Files auto-added based on open editor tab
Overflow: pills scroll horizontally
38.3 Chat Keyboard Shortcuts
```
Enter              → Send message
Shift+Enter        → New line in input
Ctrl+Enter         → Send + keep focus
↑ (when input empty) → Edit last message
Escape             → Cancel current stream
Ctrl+K             → Clear chat (with confirmation)
Ctrl+/             → Open prompt history picker
```
38.4 Message Actions (Desktop Hover)
Hovering a message reveals action buttons on the right:
```
[📋 Copy] [🔁 Retry] [✏️ Edit] [🗑 Delete]
```
For AI code blocks, additional buttons appear:
```
[📋 Copy Code] [Apply to File ▾] [Open in Editor] [Create New File]
```
"Apply to File ▾" opens a dropdown if multiple relevant files exist.
---
39. DESKTOP PREVIEW PANEL
39.1 Preview Tab (In Right Panel)
```
┌─────────────────────────────────────┐
│ [Chat] [Preview]                    │
├─────────────────────────────────────┤
│ [↺ Reload] [↗ Open] [📱 │ 💻] [⚙] │  ← Preview controls
├─────────────────────────────────────┤
│                                     │
│  <iframe preview>                   │
│                                     │
│                                     │
├─────────────────────────────────────┤
│ [▼ Build Logs]  (collapsible)       │
└─────────────────────────────────────┘
```
39.2 Desktop Preview Enhancements
Detach preview: Click ↗ icon to open preview in a separate browser window (full-screen dev experience)
Viewport size input: Type exact width: `[375] px` for precise responsive testing
Device presets dropdown: iPhone SE / Pixel 7 / iPad / Desktop
Auto-refresh toggle: When ON, re-deploys preview automatically after AI finishes modifying files (desktop can handle this; mobile couldn't)
Preview URL copy button: One click copies shareable Firebase Hosting URL
Console log viewer: Panel below preview shows `console.log` outputs from preview (via postMessage from iframe)
39.3 Build Log Panel (Desktop)
Collapsible panel below preview:
```
▼ Build Logs                              [Clear] [Copy all]
─────────────────────────────────────────────────────
✓ Detected: React + Vite project
▶ Running: npm run build
  > vite build
  vite v5.0.0 building for production...
  ✓ 234 modules transformed
  dist/index.html    0.46 kB
  dist/assets/...    142.3 kB
✓ Build complete (4.2s)
▶ Deploying to Firebase Hosting...
✓ Preview ready: https://app--preview-abc123.web.app
```
---
40. DESKTOP TERMINAL ENHANCEMENTS
40.1 Terminal Panel Layout
```
┌─────────────────────────────────────────────────────────────────┐
│ TERMINAL  [bash ×] [npm dev ×] [+]     [↑] [⊡] [×]           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  > npm run dev                                                  │
│  VITE v5.0.0  ready in 312 ms                                  │
│  ➜  Local:   http://localhost:5173/                            │
│  ➜  Network: use --host to expose                              │
│  |                                                              │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│ $ [command input...............................] [▶ Run]        │
└─────────────────────────────────────────────────────────────────┘
```
40.2 Multiple Terminal Sessions (Desktop Only)
Tab system for terminal (like VS Code integrated terminal)
Each tab runs one background command process
Tab label = command name (e.g., "npm dev", "python app")
`+` to open new terminal session
Right-click tab: Rename / Kill process / Clear output
Maximum sessions: Free: 2, Pro: 5
40.3 Terminal Desktop Features
Larger output area (taller panel, more lines visible)
Scrollback buffer: 5000 lines (vs 500 on mobile)
Text selection + copy: Click-drag to select, right-click to copy
ANSI color support: Full color terminal output (npm, git etc. with colors)
Command history: ↑/↓ arrows navigate command history (last 100 commands per session)
Clear button: Clears terminal output without killing process
Word-wrap toggle: Long lines wrap or scroll horizontally
Font size control: Ctrl+= / Ctrl+- to adjust terminal font size independently
---
41. DESKTOP KEYBOARD SHORTCUT SYSTEM
41.1 Global Shortcuts
```
Ctrl+K               → Command palette (search all actions)
Ctrl+P               → Quick file open (fuzzy search all project files)
Ctrl+Shift+P         → Toggle right panel
Ctrl+B               → Toggle left sidebar
Ctrl+`               → Toggle terminal panel
Ctrl+,               → Open settings
Ctrl+Shift+N         → New project
Ctrl+N               → New chat
Ctrl+S               → Save current file
Ctrl+Shift+S         → Save all files
Ctrl+Z               → Undo (in editor)
Ctrl+Shift+Z         → Redo (in editor)
Ctrl+/               → Toggle line comment (in editor)
F5                   → Run preview / refresh preview
F1                   → Open command palette
Escape               → Cancel AI stream / close modal / close search
```
41.2 Navigation Shortcuts
```
Ctrl+1 / Ctrl+2 / Ctrl+3   → Switch to tab 1/2/3 in editor
Ctrl+Tab                     → Cycle editor tabs forward
Ctrl+Shift+Tab               → Cycle editor tabs backward
Ctrl+W                       → Close active editor tab
Ctrl+Shift+T                 → Reopen last closed tab
Ctrl+Shift+E                 → Focus file explorer
Ctrl+Shift+F                 → Focus global search
Ctrl+Shift+G                 → Focus git panel
Ctrl+Shift+X                 → Focus chat input
Alt+Left / Alt+Right         → Navigate editor history (back/forward)
```
41.3 Editor Shortcuts
```
Ctrl+F               → Find in file
Ctrl+H               → Find & replace in file
Ctrl+Shift+H         → Replace all in file
Ctrl+G               → Go to line number
Ctrl+D               → Select next occurrence of word
Ctrl+Shift+L         → Select all occurrences of word
Alt+Click            → Add cursor at click position
Alt+↑ / Alt+↓        → Move line up/down
Shift+Alt+↑/↓        → Duplicate line up/down
Ctrl+Shift+K         → Delete line
Ctrl+Enter           → Insert line below
Ctrl+Shift+Enter     → Insert line above
Shift+Alt+F          → Format document
Ctrl+]               → Indent line
Ctrl+[               → Outdent line
Ctrl+Space           → Trigger autocomplete
F12                  → Go to definition
```
41.4 Command Palette (Ctrl+K)
Fuzzy-search interface for all app actions:
```
┌──────────────────────────────────────────────────────┐
│ > _                                                  │
├──────────────────────────────────────────────────────┤
│ Recently used                                        │
│   npm run build                                      │
│   New chat                                           │
│   Run preview                                        │
│──────────────────────────────────────────────────────│
│ All commands                                         │
│   Open file...                    Ctrl+P            │
│   New project                     Ctrl+Shift+N      │
│   Run preview                     F5                │
│   Toggle terminal                 Ctrl+`            │
│   Export as ZIP                                      │
│   Push to GitHub                                     │
│   Open settings                   Ctrl+,            │
│   Switch model...                                    │
│   Clear chat                                         │
│   View snapshots                                     │
└──────────────────────────────────────────────────────┘
```
---
42. DESKTOP RIGHT-CLICK CONTEXT MENUS
42.1 File in Explorer (Right-Click)
```
New File
New Folder
─────────────────
Open
Open to the Side
─────────────────
Rename           F2
Duplicate
Copy Path        Ctrl+Shift+C
Copy Relative Path
─────────────────
Cut
Copy
Paste
─────────────────
Delete           Delete key
Move to Trash
─────────────────
Reveal in File Manager
─────────────────
View Snapshot history
```
42.2 Code in Editor (Right-Click)
```
Cut / Copy / Paste
─────────────────
Apply AI Fix
Explain with AI
Refactor with AI
Add Documentation
Write Tests for this
─────────────────
Go to Definition    F12
Peek Definition
Find All References
Rename Symbol       F2
─────────────────
Format Document     Shift+Alt+F
Format Selection
─────────────────
Copy Line Up
Copy Line Down
Move Line Up
Move Line Down
```
42.3 Chat Message (Right-Click)
```
Copy message
Copy code blocks only
─────────────────
Edit message
Delete message
─────────────────
Apply all code changes
Open code in editor
```
---
43. DESKTOP DRAG AND DROP
43.1 File Operations
Drag file in explorer: Move file to different folder
Drag file to editor tab bar: Open file in new tab
Drag file to split editor: Open in split view
Drag file to chat input: Attach file to message (sends file contents to AI)
Drag image to chat input: Attach image for visual context
Drag external file from OS file manager to explorer: Upload file to project
Visual feedback: drag ghost image + drop zone highlight
43.2 Tab Operations
Drag editor tabs to reorder
Drag tab to split pane area: open in split view
Drag tab to separate window (optional, if browser supports)
---
44. DESKTOP SEARCH SYSTEM
44.1 Global Project Search
Accessed via `Ctrl+Shift+F` or left sidebar Search icon:
```
┌─── SEARCH ─────────────────────────────────────────┐
│ 🔍 [Search project files...               ]        │
│    [Replace...                            ]        │
│    [Aa] [.*] [ab] [📁] ← case/regex/word/folder   │
├────────────────────────────────────────────────────┤
│ 24 results in 8 files                              │
│                                                    │
│ ▶ src/App.jsx (3 matches)                         │
│   Line 14:  const [user, setUser] = useState()    │
│   Line 28:  if (user && user.isLoggedIn) {        │
│   Line 45:  return <UserCard user={user} />        │
│                                                    │
│ ▶ src/auth/Login.jsx (2 matches)                  │
│   ...                                              │
└────────────────────────────────────────────────────┘
```
Click result line → jump to that line in editor
Replace: replace individual or all occurrences inline
---
End of VibeCoder Unified Product Specification v3.1
