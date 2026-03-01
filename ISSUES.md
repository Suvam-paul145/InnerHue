# InnerHue – Project Issues

This document outlines twenty carefully considered issues for the InnerHue emotional reflection web app — ten hard-level and ten medium-level. Each entry describes what needs to be done, why it matters, and how it can be approached. I want to work on all of these issues.

---

## Hard-Level Issues

---

### Hard Issue 1 — Backend Integration with Express.js and MongoDB

**Problem Description**

InnerHue currently stores all mood data exclusively in the browser's localStorage through the Zustand store defined in `lib/useMoodStore.ts`. This means every piece of user data — mood history, streaks, custom moods, and personalization settings — lives only in that single browser instance. If a user clears their browser storage, uses a different browser, or switches devices, all their emotional history is permanently gone. There is no server, no database, and no way to recover lost data. The app is entirely stateless from a backend perspective.

**Why It Is Required**

A mood-tracking application only delivers real long-term value when users can trust that their data is safe and persistent. Emotional wellness journeys are deeply personal and built over weeks or months. Losing that history defeats the entire purpose of the platform. Beyond data safety, a backend is the foundation for nearly every future feature listed in the README — including social sharing, cross-device sync, AI personalization, and push notifications. Without a backend, InnerHue will always be capped at being a local, single-device experience.

**My Approach**

I will introduce a Node.js and Express.js API server alongside the existing Next.js frontend. MongoDB will serve as the database, with Mongoose as the ODM for schema definitions. I will create data models for users, mood entries, and custom moods. The Next.js API routes in the `app/api` directory will be extended or replaced with calls to the Express backend. The Zustand store in `lib/useMoodStore.ts` will be refactored so that mutations (add, delete, clear) call the API rather than just updating localStorage. A migration path for existing localStorage data will also be provided so current users do not lose their history during the transition.

I want to work on this issue.

---

### Hard Issue 2 — User Authentication and Account Management

**Problem Description**

InnerHue has no concept of a user account. The login and signup pages exist in `app/login` and `app/signup`, and there is a `LoginForm.tsx` and `signupform.tsx` component, but there is no actual authentication mechanism wired up. Anyone visiting the app sees the same experience and the pages appear non-functional from an auth standpoint. There is no session management, no token-based authentication, no password hashing, and no protected routes.

**Why It Is Required**

Without authentication, it is impossible to associate mood data with a specific person, which in turn makes any backend or cloud sync meaningless. Authentication is also a prerequisite for social features, personalized AI recommendations, and data privacy. Users need to feel confident that their emotional data belongs only to them. Proper auth — with secure session or token management — is the bedrock of any trustworthy wellness platform.

**My Approach**

I will implement JWT-based authentication connected to the Express.js backend described in Hard Issue 1. The signup flow will hash passwords using bcrypt before storing them in MongoDB. Login will return a signed JWT stored in an HTTP-only cookie. The Next.js middleware will validate this token on protected routes such as `/analytics`, `/mood/[id]`, `/insights`, and `/personalization`. The existing `LoginForm.tsx` and `signupform.tsx` components will be wired to the real API endpoints. A refresh token mechanism will handle session longevity without forcing users to log in repeatedly.

I want to work on this issue.

---

### Hard Issue 3 — Real AI and NLP Integration for Dynamic Suggestions

**Problem Description**

The AI-powered suggestions in InnerHue — journal prompts, affirmations, and keyword clouds — are currently sourced from static data files, most notably `lib/reflectionData.ts` and `lib/moodData.ts`. The `AITherapist.tsx` component and `SimpleLangFlowChatbot.tsx` exist, but they rely on pre-written content rather than generating anything dynamically based on what a user actually types or the patterns in their mood history. The chatbot is not truly conversational and cannot adapt to individual context.

**Why It Is Required**

The emotional state of a human being is nuanced and contextual. Static arrays of prompts cannot capture that nuance. A user who has been logging sadness every day for a week has very different needs than a user who selected sadness for the first time. Real NLP allows the application to detect emotional tone in text input, generate personalized prompts, and have a meaningful therapeutic conversation. This is what would distinguish InnerHue from a simple mood diary and make it a genuinely helpful wellness tool.

**My Approach**

I will integrate the OpenAI API (or a locally-hosted alternative like Ollama for privacy) into the `app/api/chat` route. The `AITherapist.tsx` component will send the user's typed messages along with their recent mood history to the API endpoint. The backend will construct a system prompt that includes the user's emotional context — frequency of specific moods, streak data from `lib/streakService.ts`, and the current selected emotion — before calling the language model. The response will be streamed back to the UI for a real-time conversational feel. A fallback using the existing static data in `data/fallbackQuotes.ts` will remain in place for when the API is unavailable.

I want to work on this issue.

---

### Hard Issue 4 — Cross-Device Data Synchronization

**Problem Description**

All user data in InnerHue is stored in the browser's localStorage through the Zustand persistence layer in `lib/useMoodStore.ts`. This creates a hard boundary at the device level. A user who logs their morning mood on a phone and their evening mood on a laptop will have two completely separate, incompatible histories. Custom moods created via `lib/customMoods.ts` on one device will not appear on another. Personalization settings stored through `hooks/usePersonalization.ts` are also locked to a single browser.

**Why It Is Required**

Modern users expect their data to follow them seamlessly across their devices. A wellness platform that fails to deliver this feels unreliable and incomplete. If a user gets a new phone or reinstalls their browser, they should not lose months of emotional history. Cross-device sync is also a fundamental requirement for any kind of backup or data recovery capability, and it enables future features like shared insights with a therapist or trusted contact.

**My Approach**

This issue depends on the backend introduced in Hard Issue 1 and the authentication from Hard Issue 2. Once a user is authenticated, every write to the Zustand store will be mirrored to the server via an API call. On app load, if a user is logged in, the client will fetch the server state and merge it with any local state using a timestamp-based conflict resolution strategy — the most recent entry wins. The custom moods from `lib/customMoods.ts` and personalization settings from `lib/personalizationTypes.ts` will be included in the sync scope.

I want to work on this issue.

---

### Hard Issue 5 — End-to-End and Unit Test Suite

**Problem Description**

The InnerHue repository has a `app/test` directory but no actual test files, test configuration, or testing framework setup in `package.json`. There is no evidence of unit tests for the utility functions in `lib/`, no component tests for the React components in `components/`, and no end-to-end tests for user flows like selecting a mood, viewing analytics, or creating a custom emotion. The application has no automated safety net against regressions.

**Why It Is Required**

As InnerHue grows and contributors add features, the absence of tests creates compounding risk. Refactoring the Zustand store in `lib/useMoodStore.ts`, changing the mood data in `lib/moodData.ts`, or updating the streak logic in `lib/streakService.ts` could silently break key functionality with no warning. A test suite is not just a quality tool — it is documentation of expected behavior and a collaboration enabler. Open-source contributors in particular need tests to validate that their changes do not break existing features.

**My Approach**

I will set up Jest and React Testing Library for unit and component tests, and Playwright for end-to-end tests. I will begin with the most critical logic: the `computeStats` function in `lib/useMoodStore.ts`, the streak calculation in `lib/streakService.ts`, the validation logic in `lib/validation.ts`, and the quote retrieval in `lib/getQuote.ts`. Component tests will cover `MoodCard.tsx`, `AddMoodModal.tsx`, and `MoodChart.tsx`. End-to-end tests will cover the primary user journey: landing on the home page, selecting a mood, navigating to the detail view, and verifying that the analytics page reflects the new entry.

I want to work on this issue.

---

### Hard Issue 6 — Voice Journaling with Sentiment Analysis

**Problem Description**

InnerHue has no audio input capability. All emotional reflection happens through clicking predefined mood cards in `app/emotions/page.tsx`. There is no way for a user to speak about how they are feeling, record a voice journal entry, or have the application analyze the emotional content of their words. The README lists voice journaling as a future enhancement but there is no groundwork for it in the current codebase.

**Why It Is Required**

Voice is the most natural form of human expression. Many people find it easier — and more emotionally honest — to speak their feelings than to tap an emoji on a screen. Voice journaling dramatically lowers the barrier to self-reflection, especially for users who are not comfortable with writing. Sentiment analysis on voice recordings would allow InnerHue to detect emotional themes automatically and suggest mood categories without requiring the user to explicitly choose one, making the experience far more accessible.

**My Approach**

I will add a voice recording interface accessible from the mood detail pages within `app/mood/[id]`. The Web Speech API will handle recording in the browser. The audio transcript will be sent to the backend, which will call a sentiment analysis API — either OpenAI Whisper for transcription combined with a sentiment model, or a purpose-built service. The detected emotional keywords will be matched against the mood taxonomy in `lib/moodData.ts` and surfaced as suggested mood tags. The raw transcript will be stored as an optional journal note associated with the mood entry in the database.

I want to work on this issue.

---

### Hard Issue 7 — Progressive Web App with Full Offline Support and Push Notifications

**Problem Description**

The `package.json` includes `next-pwa` as a dependency and `app/manifest.ts` defines a web app manifest, but the offline experience is incomplete. If a user loses internet connectivity, they cannot add a new mood entry, view their analytics reliably, or interact with the AI chatbot. There is no background sync for entries created offline, and no push notification system to send mood check-in reminders even when the app is not open.

**Why It Is Required**

A wellness application must be available when a person needs it most — which is often in moments of emotional distress or reflection that do not wait for a reliable internet connection. Full offline support means users can always log a mood, always read their history, and always access their personalization settings. Push notifications for mood reminders respect user attention by delivering timely check-in prompts even when the app is not in the foreground, which is proven to significantly improve streak maintenance and daily engagement.

**My Approach**

I will configure the service worker through the existing `next-pwa` setup to cache all critical routes including `/`, `/emotions`, `/analytics`, and `/mood/[id]`. Offline mood entries will be queued using the Background Sync API and flushed to the server when connectivity is restored. I will integrate the Web Push API for sending scheduled reminder notifications. The notification schedule will be configurable through the personalization settings page at `app/personalization/page.tsx`. The nudge logic in `lib/nudgeService.ts` will be extended to trigger push notifications rather than just in-app toasts.

I want to work on this issue.

---

### Hard Issue 8 — Mood Pattern Recognition Using Machine Learning

**Problem Description**

The analytics dashboard in `app/analytics/page.tsx` and the components `BentoDashboard.tsx` and `MoodChart.tsx` display historical mood data in a visual format, but all analysis is purely retrospective and statistical. The app can show which mood is most common or how many entries exist this week, but it cannot predict future emotional states, identify triggers from behavioral patterns, detect concerning trends like a prolonged streak of negative emotions, or provide proactive recommendations.

**Why It Is Required**

The true value of longitudinal emotional data lies in the insights it can generate. A machine learning layer that can identify patterns — for example, that a user consistently feels anxious on Monday mornings or calm after logging certain activities — transforms InnerHue from a passive diary into an active wellness partner. Early detection of negative emotional trends could prompt gentle interventions and recommendations before a user reaches a crisis point. This is the kind of differentiated value that wellness platforms aspire to provide.

**My Approach**

I will build a lightweight pattern recognition service on the backend. Using the user's mood history from the database (introduced in Hard Issue 1), the service will run periodic analysis to detect recurring temporal patterns, emotional clusters, and trajectory trends. For the initial version, rule-based heuristics will be used — for instance, flagging when a user has logged more than five consecutive negative moods, or identifying a recurring pattern on specific days of the week. A dedicated insights panel will be added to the analytics page, and the insights data will flow into the existing `app/insights/page.tsx` to enrich it with machine-generated observations. The foundation will be built to allow a proper ML model to replace the heuristics in a later iteration.

I want to work on this issue.

---

### Hard Issue 9 — Social Features and Shared Insight Boards

**Problem Description**

InnerHue is entirely a single-player experience. There is no way for a user to share a mood summary, export an insight card, invite a friend to check in together, or create a shared emotional wellness space with a trusted person such as a partner, family member, or therapist. The README mentions social features as a planned enhancement but there is no design or implementation for this.

**Why It Is Required**

Emotional wellness is often strengthened through connection. Sharing a "how I've been feeling this week" summary with a therapist or a trusted friend creates accountability and opens conversations that might otherwise not happen. For couples or close friends, a shared check-in board creates mutual empathy and understanding. These social features do not require exposing raw data — even a summarized or anonymized view adds significant value. Social proof and connection are also among the strongest drivers of engagement and retention for consumer applications.

**My Approach**

I will design a sharing system that allows users to generate shareable "insight cards" — visually designed summaries of their mood journey over a chosen time period — that can be exported as images or shared via a unique URL. These cards will be generated server-side using the mood data from the backend. A separate "companion mode" will allow two users to opt into a mutual check-in relationship, where each can see a simplified emotional summary of the other without revealing full journal entries. Privacy controls will be prominent and granular, ensuring users always decide exactly what to share and with whom.

I want to work on this issue.

---

### Hard Issue 10 — Wearable and Health Data Integration

**Problem Description**

InnerHue has no connection to any physiological or health data. Mood and emotional state are significantly influenced by sleep quality, heart rate, physical activity, and other biometric signals — all of which modern wearables and health platforms collect. The app currently relies entirely on the user's self-reported emotional state, which is subjective and sometimes delayed relative to when the emotion actually occurred.

**Why It Is Required**

Integrating health data would create a richer, more complete picture of a user's emotional and physical wellbeing. A user whose sleep data shows four consecutive nights of under six hours of sleep and who has also been logging "stressed" and "overwhelmed" can receive a far more contextually accurate recommendation than one based solely on mood taps. Wearable integration elevates InnerHue from a mood diary to a holistic wellness platform and positions it competitively alongside leading mental health apps.

**My Approach**

I will implement OAuth-based connections to Apple HealthKit (via a native shell for iOS) and Google Fit for Android users. For web-only users, I will integrate with the Fitbit API and Garmin Connect API as they offer web-based OAuth flows. Health data — specifically sleep duration, resting heart rate, and step count — will be fetched periodically and stored alongside mood entries in the database. The analytics dashboard will overlay this health data on the mood timeline, and the AI suggestion system from Hard Issue 3 will be updated to include this physiological context when generating recommendations.

I want to work on this issue.

---

## Medium-Level Issues

---

### Medium Issue 1 — Journal Entry System for Mood Reflections

**Problem Description**

When a user selects an emotion and navigates to the mood detail page at `app/mood/[id]`, the experience is visual and reactive but does not invite the user to write anything. The existing suggestion panel in `components/SuggestionPanel.tsx` offers journal prompts as questions, but there is no text input field, no way to record a written response, and no way to save or revisit what was written. The journaling capability is implied but entirely absent.

**Why It Is Required**

Written reflection is one of the most evidence-backed methods for processing emotions and building self-awareness. Simply naming an emotion is a starting point, but articulating why one feels a certain way deepens the insight significantly. Journal entries would also give the AI therapist meaningful data to respond to, and they would make the mood history in `app/analytics/page.tsx` far more meaningful — turning a simple list of emotion labels into a genuine personal timeline.

**My Approach**

I will add a text area component to the mood detail page at `app/mood/[id]` that appears after a mood is selected. The text area will be pre-seeded with one of the existing journal prompts from `lib/reflectionData.ts` as a starting point. Saving the entry will write the text to the Zustand store in `lib/useMoodStore.ts` by extending the `MoodEntry` interface to include an optional `journalText` field. The analytics history view in `app/analytics/page.tsx` will be updated to display a preview of the journal text alongside each entry, with the ability to expand and read the full note.

I want to work on this issue.

---

### Medium Issue 2 — Mood Reminders and Daily Check-In Notifications

**Problem Description**

The nudge service defined in `lib/nudgeService.ts` exists but operates only within the app — it can trigger a toast notification if the user is already on the page. There is no scheduled reminder system that prompts users to check in at a specific time each day, and there is no mechanism to reach the user when the application is not open in their browser. The streak tracking in `lib/streakService.ts` motivates consistency but there is nothing that actively reminds users to maintain it.

**Why It Is Required**

Habit formation is central to the value of any wellness application. Most users who abandon mood-tracking apps do so not because they stopped caring but because they simply forgot. A well-timed, gentle daily reminder dramatically increases check-in frequency and streak length. The reminder does not need to be intrusive — a single in-browser notification at a user-chosen time is enough to anchor the habit. This feature directly supports the core goal of building a consistent emotional self-awareness practice.

**My Approach**

I will extend the personalization settings page at `app/personalization/page.tsx` with a notification preference section where users can select a daily reminder time and toggle the feature on or off. The browser's Notification API will be used to schedule reminders for users who grant permission. The `lib/nudgeService.ts` file will be extended to register a notification via the Service Worker at the configured time. For users who have not opened the app that day, the nudge will include a message generated from the existing nudge logic, customized to reflect the user's recent mood patterns.

I want to work on this issue.

---

### Medium Issue 3 — Search and Filter Functionality in Analytics History

**Problem Description**

The analytics history view in `app/analytics/page.tsx` currently shows the fifteen most recent mood entries in reverse chronological order with no way to search, filter, or sort. A user who has been tracking their emotions for several months and wants to find all instances when they felt "hopeful" or view entries from a specific week must scroll through a truncated list with no tools to help them find what they are looking for.

**Why It Is Required**

As users build up a meaningful mood history over time, discoverability and navigation of that data become essential. The emotional patterns most worth reflecting on — recurring stress before certain events, periods of calm following specific activities — can only be found if the user has the tools to explore their history. A simple search and filter system transforms the history from a passive list into an active reflection tool, making the app substantially more valuable for long-term users.

**My Approach**

I will add a filter bar above the history list in `app/analytics/page.tsx`. The filters will include a text search field for emotion names, a date range picker using the `react-day-picker` library already present in `package.json`, and a dropdown to filter by emotion category (positive, negative, calm, etc.) drawing from the category data in `lib/moodData.ts`. Sorting options — newest first, oldest first, alphabetical by emotion — will also be provided. All filtering will happen client-side for performance since the data is already loaded from the Zustand store.

I want to work on this issue.

---

### Medium Issue 4 — Mood Goals and Milestone Tracking

**Problem Description**

InnerHue tracks mood history and streaks but sets no goals and celebrates no milestones. A user has no way to set a personal intention — such as checking in daily for thirty days, logging a positive emotion at least once per week, or completing a seven-day reflection challenge. The streak counter in `lib/streakService.ts` tracks the raw number, but there is no goal attached to it and no celebration when a meaningful threshold is reached.

**Why It Is Required**

Goals give users a sense of direction and purpose, which is especially important in a wellness context where the benefits of habit-building are gradual and easy to overlook. Milestones — reaching a ten-day streak, logging fifty total moods, or exploring twenty different emotional states — provide positive reinforcement at meaningful points in the journey. This kind of achievement system increases retention, creates emotional investment in the product, and encourages deeper exploration of the app's features.

**My Approach**

I will define a set of milestone thresholds within the existing data architecture in `lib/`. These will cover streak length, total entries logged, emotional diversity (number of distinct emotions logged), and category exploration (having logged at least one emotion from every category in `lib/moodData.ts`). When a user reaches a milestone, a celebratory animation using Framer Motion will appear and a toast notification from `react-hot-toast` will confirm the achievement. The `BentoDashboard.tsx` component will include a new milestone progress card, and a dedicated goals page will allow users to set custom personal targets.

I want to work on this issue.

---

### Medium Issue 5 — Accessibility Improvements

**Problem Description**

InnerHue's visual design is polished, but accessibility has not been systematically addressed. The mood cards in `components/MoodCard.tsx` use `div` elements with click handlers rather than semantic `button` elements with proper ARIA labels. The animated gradient backgrounds and aurora effects do not check for `prefers-reduced-motion`. The color contrasts between text and background in several components — particularly on the translucent glassmorphism cards — have not been verified against WCAG AA standards. Keyboard navigation between mood cards is not implemented.

**Why It Is Required**

Emotional wellness tools are especially important for people who may also be managing disabilities — physical, cognitive, or visual. An app that cannot be used by someone navigating with only a keyboard or a screen reader is excluding exactly the population that might benefit from it most. Beyond ethical obligation, accessibility compliance reduces legal risk and broadens the potential user base. It also tends to improve the overall UX for all users, as accessibility best practices align with general usability principles.

**My Approach**

I will audit the component library starting with the most-used interactive elements: `MoodCard.tsx`, `AddMoodModal.tsx`, `ThemeToggle.tsx`, and `SuggestionPanel.tsx`. Interactive `div` elements will be replaced with semantically correct `button` elements or given appropriate `role`, `tabIndex`, and `aria-label` attributes. All animations controlled by Framer Motion will be wrapped with a check against the `prefers-reduced-motion` media query. Color contrast ratios will be verified using the WCAG contrast checker and adjusted where needed, particularly in the analytics history section of `app/analytics/page.tsx`.

I want to work on this issue.

---

### Medium Issue 6 — Improved Mobile Navigation with a Bottom Navigation Bar

**Problem Description**

On mobile devices, the navigation in InnerHue is confined to a top header bar containing icon-only links to analytics, music, and a theme toggle. There is no persistent navigation that provides clear wayfinding across the main sections of the app. Users on small screens must scroll back to the top to navigate between pages, the icon-only buttons lack descriptive labels, and the overall mobile navigation experience does not meet the expectations of a modern consumer wellness application.

**Why It Is Required**

Mobile devices are likely the primary platform for a mood-tracking app given that emotional check-ins happen throughout the day, often in moments away from a desk. The current navigation creates friction on the very platform that matters most. A bottom navigation bar is the established mobile UX pattern for apps with four to five primary sections, as it keeps navigation within easy thumb reach and provides clear visual context about where the user currently is within the app.

**My Approach**

I will create a new `BottomNav.tsx` component in the `components/` directory that renders a fixed bottom navigation bar on screen widths below the tablet breakpoint defined in `tailwind.config.js`. The bar will include four tabs — Home, Emotions, Analytics, and Music — with icons from `lucide-react` and short text labels. The active tab will be highlighted using the current route from `next/navigation`'s `usePathname` hook. The component will be added to `app/layout.tsx` so it appears consistently across all pages. The existing top-header navigation on mobile will be simplified to only show the app logo and the theme toggle.

I want to work on this issue.

---

### Medium Issue 7 — Custom Theme Creation Beyond Light and Dark Mode

**Problem Description**

InnerHue supports light and dark modes through the `ThemeProvider.tsx` and `ThemeToggle.tsx` components using `next-themes`. The dark mode uses a deep purple-to-indigo gradient palette defined in `app/globals.css`. However, users have no control over the visual palette beyond this binary choice. The rich color system described in the README — with emotion-driven colors for happy, calm, sad, excited, and anxious — is used for mood visualization but not for the overall application theme.

**Why It Is Required**

Color is deeply personal and emotionally significant, especially in a wellness application. Allowing users to choose or build a theme that resonates with them creates a more intimate relationship with the product. A user who finds that a warm amber palette makes them feel calm and focused will use the app more consistently than one who is stuck with colors that feel clinical or mismatched to their personality. Custom themes also provide a meaningful way to incorporate InnerHue's color psychology into the overall visual experience.

**My Approach**

I will extend the personalization settings at `app/personalization/page.tsx` with a theme customization section. I will define five preset emotional themes — each based on a combination of the emotion colors from `lib/moodData.ts` — named after broad emotional tones such as "Serene," "Vibrant," "Grounded," "Dreamy," and "Bold." The selected theme will be stored in the personalization hook at `hooks/usePersonalization.ts` and applied as CSS custom property overrides on the document root, complementing the existing `next-themes` system. An advanced option will allow users to pick individual accent colors using a color picker.

I want to work on this issue.

---

### Medium Issue 8 — Explore Page with Emotion Education Content

**Problem Description**

The `app/explore/page.tsx` exists in the file structure but its contents have not been fully developed into an educational or exploratory experience. InnerHue offers 38 distinct emotion categories, yet the app provides no educational context about these emotions — their psychological definitions, how they manifest physically, what typically causes them, or how they relate to one another in the broader emotional landscape. The emotion detail pages at `app/mood/[id]` focus on reflection tools rather than education.

**Why It Is Required**

Emotional literacy — the ability to recognize, name, and understand a wide range of feelings — is a foundational skill in emotional intelligence. Many users may select a mood without fully understanding what distinguishes "anxious" from "overwhelmed," or "melancholy" from "sad." Educational content on the Explore page would help users develop a more precise emotional vocabulary, which in turn makes their mood logging more accurate and their self-reflection more meaningful. This content also positions InnerHue as a trusted educational resource, not just a tracker.

**My Approach**

I will build out `app/explore/page.tsx` into a browsable library of emotions. Each emotion card will link to a dedicated content page that describes the emotion's psychological definition, common physical sensations, typical triggers, and healthy coping strategies. I will source this content from the existing rich data already partially defined in `lib/moodData.ts` and `lib/reflectionData.ts`, extending them with educational fields. A search bar and category filter will make the library easy to navigate. A "related emotions" section on each page will show nearby emotional states, helping users understand the nuanced differences within emotional families.

I want to work on this issue.

---

### Medium Issue 9 — Performance Optimization and Lazy Loading

**Problem Description**

The home page at `app/page.tsx` imports and renders all 38 mood cards simultaneously on initial load, along with the `AITherapist.tsx` chatbot, the `AuroraBackground.tsx` component with its canvas animations, and the `Hero.tsx` landing section. The `MoodChart.tsx` and `BentoDashboard.tsx` components in the analytics page are also rendered eagerly. Heavy dependencies like `three`, `@react-three/fiber`, `@react-three/drei`, and `framer-motion` may contribute to a large initial JavaScript bundle.

**Why It Is Required**

Page load performance is directly correlated with user retention. A slow initial load is particularly damaging for a wellness app where users often arrive in a moment of emotional need and expect immediate responsiveness. The 2000ms artificial loading delay currently in `app/page.tsx` exacerbates this. Code splitting and lazy loading are standard Next.js optimizations that can significantly reduce the initial bundle size and time-to-interactive, improving experience especially on low-power mobile devices.

**My Approach**

I will audit the bundle size using Next.js's built-in bundle analyzer. Components that are not needed for the initial paint — particularly `AITherapist.tsx`, `OrbVisualizer.tsx`, `ShaderOrb.tsx`, and the Three.js-powered visualizations — will be imported using `next/dynamic` with `ssr: false` and `loading` placeholders. The `setTimeout`-based simulated loading in `app/page.tsx` will be removed entirely and replaced with actual loading state driven by real data availability, as a simulated delay provides no user value and directly harms the time-to-interactive metric. Images in the `public/` directory will be audited for format and size. The large static data in `lib/moodData.ts` will be reviewed to determine if any of it can be loaded on demand rather than at build time.

I want to work on this issue.

---

### Medium Issue 10 — Improved Error Handling and User Feedback System

**Problem Description**

The `components/ErrorState.tsx` component exists and is used in `app/page.tsx` for handling loading failures, but error handling is inconsistent across the application. The analytics page at `app/analytics/page.tsx`, the mood detail pages at `app/mood/[id]`, and the music page at `app/music/page.tsx` do not have consistently implemented error boundaries or fallback states. The `app/_offline/` route exists but it is unclear when or whether it is actually served to users who lose connectivity. Console errors from dev-mode logging scattered through the codebase (as seen in `app/emotions/page.tsx`) are not systematically handled.

**Why It Is Required**

Error handling is the difference between an app that feels trustworthy and one that feels fragile. In a wellness context, encountering an unhandled error or a blank screen during a moment of emotional vulnerability can actively harm the user experience and erode trust in the platform. Consistent, friendly error states with clear calls to action — retry, go back, contact support — maintain user confidence even when something goes wrong. Proper error boundaries also prevent a single component failure from crashing the entire application.

**My Approach**

I will create a reusable React error boundary component in `components/` that wraps major page sections and displays the existing `ErrorState.tsx` UI with contextually appropriate messages. I will audit every page in the `app/` directory and add appropriate loading and error states where they are missing. The dev-mode console logs in files like `app/emotions/page.tsx` will be replaced with a structured logging utility that respects the production environment and never leaks internal information to users. The offline page at `app/_offline/` will be verified to function correctly with the service worker configuration.

I want to work on this issue.

---

*All twenty issues above are intended to meaningfully advance InnerHue toward a production-quality, full-featured emotional wellness platform. Each issue is self-contained enough to be worked on independently while contributing to a coherent overall product roadmap.*
