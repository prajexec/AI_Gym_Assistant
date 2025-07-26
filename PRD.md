# Product Requirements Document (PRD)

## Project Name: CodeFlex AI — Intelligent Voice-Enabled Fitness & Nutrition Assistant

---

### 1. Document Control & Metadata
* **Document Version:** 1.0.0
* **Status:** Complete / Production-Ready MVP
* **Product Name:** CodeFlex AI (AI Gym Assistant)
* **Author:** Engineering & Product Team
* **Repository:** [https://github.com/prajexec/AI_Gym_Assistant](https://github.com/prajexec/AI_Gym_Assistant)
* **Primary Tech Stack:** Next.js 15 (App Router), React 19, TypeScript, Convex, Clerk, Google Gemini 2.0 Flash, Vapi AI, Tailwind CSS, shadcn/ui.

---

### 2. Executive Summary & Vision

#### 2.1 Overview
**CodeFlex AI** is a next-generation, AI-powered health and fitness coaching application that eliminates the complexity of onboarding and personal workout programming. Rather than forcing users through cumbersome multi-page forms or generic static questionnaires, CodeFlex AI leverages a **real-time conversational voice AI coach** (powered by Vapi AI).

Users speak directly to their AI fitness coach in natural language, discussing their fitness goals, schedule availability, previous injuries, target intensity, and dietary preferences. In the background, intelligent voice agents extract structured health metrics and orchestrate generation pipelines via **Google Gemini 2.0 Flash** and **Convex**, producing fully customized, medically-aware, and periodized workout routines alongside macro-balanced nutrition plans in under 3 minutes.

#### 2.2 Core Value Propositions
* **Zero-Friction Voice Onboarding:** Speak conversationally with an intelligent voice coach instead of filling out long, tedious forms.
* **Intelligent Personalization:** Workout regimens account for injury constraints, equipment access, frequency, and fitness level to prevent injury and overtraining.
* **Precision Nutrition Coaching:** Calculated daily caloric targets accompanied by structured, dietary-compliant daily meal plans.
* **Instant Reactive Synchronization:** Real-time persistence and updates across devices powered by Convex's reactive document engine.
* **Futuristic Cyberpunk Aesthetic:** Terminal-inspired HUD interfaces, real-time waveform voice visualizations, and responsive modern components.

---

### 3. Problem Statement & Target Audience

#### 3.1 The Problem
1. **Generic Templates Fail:** Most online fitness plans are one-size-fits-all spreadsheets that do not adapt to individual limitations (e.g., lower back pain, knee issues, home gym limitations).
2. **High Barrier to Entry for Personal Trainers:** One-on-one personal coaching costs between $50–$150/hour, making continuous professional fitness guidance inaccessible for millions.
3. **Questionnaire Fatigue:** Users abandon fitness apps during onboarding when faced with 20+ form inputs, dropdowns, and sliders.
4. **Disconnection Between Fitness and Nutrition:** Most applications isolate workout routines from dietary guidance, reducing the effectiveness of both.

#### 3.2 Target Personas
* **Persona A: The Busy Professional (e.g., Sarah, 34)**
  * *Needs:* High-efficiency home workouts, limited time (3-4 days/week), modifications for sedentary-induced lower back pain, lactose-free nutrition.
  * *Goal:* Sustainable weight loss without risking injury.
* **Persona B: The Muscle-Building Enthusiast (e.g., Michael, 28)**
  * *Needs:* Hypertrophy-focused gym splits, progressive overload volume, high caloric surplus with macro distribution.
  * *Goal:* Clean muscle gain and progressive strength improvements.
* **Persona C: The Functional Fitness Seeker (e.g., Elena, 45)**
  * *Needs:* Joint-friendly, bodyweight-only routines, mobility and cardiovascular health, vegetarian diet.
  * *Goal:* Longevity, joint protection, and daily energy.

---

### 4. System Architecture & Tech Stack

```
+--------------------------------------------------------------------------------+
|                                CLIENT APPLICATION                              |
|   Next.js 15 (App Router) + React 19 + TypeScript + Tailwind CSS + shadcn/ui   |
|   - Landing Page (HUD / Cyberpunk Theme)                                       |
|   - Voice Consultation Room (Vapi AI Web SDK + Real-time Audio Waves)          |
|   - Dynamic Profile & Plan Manager (Convex Reactive Hooks)                     |
+----------------------+---------------------------------+-----------------------+
                       |                                 |
              (Authentication)                    (Voice Stream)
                       v                                 v
          +------------------------+          +--------------------+
          |      Clerk Auth        |          |      Vapi AI       |
          |  (JWT / Session State) |          | (Speech-to-Text /  |
          +-----------+------------+          |  Conversational AI)|
                      |                       +----------+---------+
              (Svix Webhooks)                            | (JSON Extraction)
                      |                                  |
                      +----------------+   +-------------+
                                       |   |
                                       v   v
+--------------------------------------------------------------------------------+
|                               CONVEX CLOUD BACKEND                             |
|  - HTTP Router Actions:                                                        |
|      * /clerk-webhook: Svix verification -> User synchronization               |
|      * /vapi/generate-program: Payload processing -> Gemini LLM Orchestration  |
|  - Database Engine:                                                            |
|      * `users` table: Clerk sync, profile photos, email index                  |
|      * `plans` table: Structured workout & diet routines, active flags         |
|  - Real-time Queries & Mutations (`createPlan`, `getUserPlans`, `syncUser`)    |
+--------------------------------------+-----------------------------------------+
                                       |
                           (Generative AI Synthesis)
                                       v
                     +-----------------------------------+
                     |      Google Gemini 2.0 Flash      |
                     |  - Prompt-guided JSON generation  |
                     |  - Exercise split & volume rules  |
                     |  - Calorie & macro calculation    |
                     +-----------------------------------+
```

#### 4.1 Tech Stack Justification
| Layer | Technology | Decision Rationale |
| :--- | :--- | :--- |
| **Framework** | Next.js 15 (App Router) & React 19 | Server-side rendering, streaming, optimized bundle sizes, and robust routing. |
| **Styling** | Tailwind CSS & shadcn/ui | Radix UI primitives with custom cyberpunk neon styling and accessibility. |
| **Authentication** | Clerk (`@clerk/nextjs`) | Enterprise-grade auth handling OAuth, sessions, JWTs, and secure Svix webhooks. |
| **Voice AI Agent** | Vapi AI Web SDK (`@vapi-ai/web`) | Low-latency bi-directional voice streaming, natural conversational turns, tool calling. |
| **Backend & DB** | Convex (`convex/server`) | Type-safe end-to-end reactivity, eliminating boilerplate REST/GraphQL layers. |
| **LLM Model** | Google Gemini 2.0 Flash | Ultra-fast token generation speed, high reasoning capacity, and strict JSON output mode. |

---

### 5. Detailed Functional Requirements

#### 5.1 Authentication & User Lifecycle
* **Registration & Sign-In:** Seamless sign-up and sign-in powered by Clerk with social and email credentials.
* **Webhook User Sync:** When a user is created or updated in Clerk, a Svix-signed webhook is sent to `/clerk-webhook` in Convex. Convex validates the signature and atomically inserts or updates records in the `users` table.
* **Route Protection:** Protected middleware (`src/middleware.ts`) secures `/generate-program` and `/profile`, redirecting unauthorized requests to `/sign-in`.

#### 5.2 Real-time Voice Consultation Room (`/generate-program`)
* **Voice Agent Initialization:** The client connects to Vapi AI using client tokens and an configured Assistant Workflow ID.
* **Context Injection:** The user's name and unique Convex/Clerk ID are injected as runtime variables (`variableValues: { full_name, user_id }`).
* **Visual Voice Feedback:**
  * Animated audio waveform bars dynamically reacting to `speech-start` and `speech-end` events.
  * Real-time scrolling transcript showing user utterances and AI responses.
  * Audio pulse indicator highlighting active agent status.
* **Call Completion & Navigation:** When the voice agent wraps up the consultation, Vapi triggers end-of-call webhooks and automatically redirects the user to `/profile`.

#### 5.3 Automated Plan Generation Pipeline
* **Trigger Endpoint:** `/vapi/generate-program` receives structured parameters extracted during the voice conversation:
  * `age`, `height`, `weight`, `injuries`, `workout_days`, `fitness_goal`, `fitness_level`, `dietary_restrictions`.
* **Gemini LLM Prompting:**
  * Dedicated prompts for both **Workout Plan** and **Diet Plan**.
  * Strict schema enforcement requiring pure JSON output without markdown wrapper noise.
  * Validation filters (`validateWorkoutPlan`, `validateDietPlan`) that normalize numeric sets/reps and calorie values.
* **Atomic Plan Activation:** Convex automatically archives any existing active plan (`isActive: false`) and inserts the new plan as `isActive: true`.

#### 5.4 Profile & Plan Dashboard (`/profile`)
* **Multi-Plan History:** Displays all current and historical plans created by the user.
* **Interactive Plan Switching:** Allows switching between generated plans with instantaneous reactive rendering.
* **Workout Schedule Explorer:**
  * Day-by-day split view with exercise names, targets (sets x reps), and rest guidance.
* **Nutrition Breakdown:**
  * Daily calorie target badge.
  * Meal-by-meal card layout (Breakfast, Lunch, Dinner, Snacks) with food items and dietary considerations.

---

### 6. Data Models & Database Schema

#### 6.1 `users` Table
```typescript
users: defineTable({
  name: v.string(),
  email: v.string(),
  image: v.optional(v.string()),
  clerkId: v.string(),
}).index("by_clerk_id", ["clerkId"])
```

#### 6.2 `plans` Table
```typescript
plans: defineTable({
  userId: v.string(),
  name: v.string(),
  workoutPlan: v.object({
    schedule: v.array(v.string()),
    exercises: v.array(
      v.object({
        day: v.string(),
        routines: v.array(
          v.object({
            name: v.string(),
            sets: v.number(),
            reps: v.number(),
          })
        ),
      })
    ),
  }),
  dietPlan: v.object({
    dailyCalories: v.number(),
    meals: v.array(
      v.object({
        name: v.string(),
        foods: v.array(v.string()),
      })
    ),
  }),
  isActive: v.boolean(),
})
  .index("by_user_id", ["userId"])
  .index("by_active", ["isActive"])
```

---

### 7. Non-Functional Requirements

#### 7.1 Performance & Latency
* **Voice Agent Latency:** Audio response latency must remain under 800ms through WebRTC/Vapi streaming.
* **LLM Generation Speed:** Gemini 2.0 Flash combined workout + diet generation must complete in under 5 seconds.
* **UI Load Speed:** Core Web Vitals score > 90 on desktop and mobile.

#### 7.2 Security & Data Privacy
* **Secret Protection:** Webhook secrets, Clerk credentials, Gemini API keys, and Vapi private tokens are isolated in environment configurations (`.env.local` / Convex environment variables) and excluded from version control.
* **Webhook Verification:** Clerk webhooks reject unverified requests lacking valid Svix headers (`svix-id`, `svix-signature`, `svix-timestamp`).
* **Row-Level Security:** User plans are indexed and queried strictly scoped by authenticated `userId`.

---

### 8. Future Roadmap & Iterations

* [ ] **Phase 2: Live Workout Companion Mode**
  * Real-time voice guidance during workouts with audio countdown timers and rep tracking.
* [ ] **Phase 3: Camera-Based Pose & Form Feedback**
  * Computer vision integration (MediaPipe) to track posture and exercise form via device webcam.
* [ ] **Phase 4: Wearables Integration**
  * Synchronization with Apple HealthKit, Google Fit, and Whoop to calibrate daily calorie targets based on real burn data.
* [ ] **Phase 5: Grocery List Generation & Delivery Integration**
  * One-click export of weekly meal plans into Instacart or Amazon Fresh grocery orders.
