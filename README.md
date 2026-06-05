# PurpleTech — Patient Side Workflow

> This document maps the **exact** navigation flow as implemented in the Flutter code.

---

## 🟣 Patient Complete Workflow

```mermaid
graph TD
    A([🚀 App Launch]) --> B[SplashScreen\nscreens/auth/splash_screen.dart]
    B --> C[RoleSelectionScreen\nscreens/auth/role_selection_screen.dart]
    C --> D[LoginSignupScreen\nscreens/auth/login_signup_screen.dart]
    D -->|OTP Verification| E[OtpScreen\nscreens/auth/otp_screen.dart]
    D -->|Forgot Password| F[ForgotPasswordScreen\nscreens/auth/forgot_password_screen.dart]
    E --> G

    D --> G{PatientDashboardScreen\nscreens/patient/patient_dashboard_screen.dart}

    %% Bottom Nav / Sidebar Tabs
    G -->|Tab: Home| H[🏠 Home Tab\n_HomeTab widget]
    G -->|Tab: Calendar| I[PatientCalendarScreen\nscreens/patient/patient_calendar_screen.dart]
    G -->|Tab: AI| J[AiBotSelectionScreen\nscreens/bot/ai_bot_selection_screen.dart]
    G -->|Tab: Therapists| K[TherapistFilterScreen\nscreens/patient/therapist_filter_screen.dart]
    G -->|Tab: Profile| L[ProfileScreen\nscreens/profile/profile_screen.dart]

    %% Home Tab Quick Actions
    H -->|🔔 Bell Icon| M[NotificationsScreen\nscreens/common/notifications_screen.dart]
    H -->|Stat: Upcoming| N1[MeetingsListScreen\ninitialTab: Upcoming]
    H -->|Stat: Requests| N2[MeetingsListScreen\ninitialTab: Requests]
    H -->|Stat: History| N3[MeetingsListScreen\ninitialTab: Completed]
    H -->|Quick Action: AI Bot| J
    H -->|Quick Action: Join| O[PatientJoinScreen\nscreens/meeting/patient_join_screen.dart]
    H -->|Quick Action: Calendar| I
    H -->|Quick Action: Sessions| N1

    %% Therapist Booking Flow
    K -->|Browse & Select Therapist| P[SessionTypeScreen\nscreens/patient/session_type_screen.dart\nChoose: Text / Audio / Video]
    P -->|Continue to Payment| Q[PaymentScreen\nscreens/common/payment_screen.dart]
    Q -->|Payment Success| R[🔔 Notification sent to Doctor\nStatus: PENDING]

    %% Sessions List → Detail
    N1 --> S[MeetingDetailScreen\nscreens/meeting/meeting_detail_screen.dart]
    N2 --> S
    N3 --> S

    %% Confirmed Session → Join Call
    S -->|Status: Confirmed\nJoin Meeting| T[WebRTCMeetingScreen\nscreens/meeting/webrtc_meeting_screen.dart]
    O --> T

    %% In-Meeting
    T -->|Live Transcript| U[📝 Real-time Transcript Overlay\nWeb Speech API / STT]
    T -->|Chat Signaling| V[SignalingService WebSocket\nservices/signaling_service.dart]
    T -->|End Call| W[🛑 Camera + Mic OFF\nHardware Released]
    W --> X[Session Saved to Backend\nStatus: COMPLETED]
    X --> S

    %% Completed Session
    S -->|Status: Completed\nView Summary| Y[📊 AI Clinical Summary\nDiagnosis + Recommendations\nfrom backend API]
    S -->|View Transcript| Z[📜 Full Session Transcript\nfrom backend API]

    %% AI Bot Path
    J -->|Chat Bot| AA[BotChatScreen\nscreens/bot/bot_chat_screen.dart]
    J -->|Voice AI| AB[GeminiChatScreen\nscreens/bot/gemini_chat_screen.dart]

    %% Styling
    classDef screen fill:#f3e5f5,stroke:#7b1fa2,stroke-width:1px,color:#000
    classDef action fill:#e3f2fd,stroke:#1565c0,stroke-width:1px,color:#000
    classDef event fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px,color:#000
    classDef entry fill:#7b1fa2,stroke:#4a148c,stroke-width:2px,color:#fff

    class A entry
    class B,C,D,E,F,G,H,I,J,K,L,M,N1,N2,N3,O,P,Q,S,T,AA,AB screen
    class R,U,V,W,X,Y,Z event
```

---

## 📋 Step-by-Step Patient Journey

| Step | Screen | File | Action |
|------|--------|------|--------|
| 1 | Splash Screen | `auth/splash_screen.dart` | App loads, checks auth token |
| 2 | Role Selection | `auth/role_selection_screen.dart` | User selects **Patient** |
| 3 | Login / Signup | `auth/login_signup_screen.dart` | Enter credentials |
| 4 | OTP Verify | `auth/otp_screen.dart` | Confirm identity |
| 5 | Patient Dashboard | `patient/patient_dashboard_screen.dart` | Home screen with 5 tabs |
| 6a | Browse Therapists | `patient/therapist_filter_screen.dart` | Search/filter by specialty |
| 6b | AI Bot | `bot/ai_bot_selection_screen.dart` | Chat or Voice therapy |
| 7 | Session Type | `patient/session_type_screen.dart` | Pick: Text / Audio / Video |
| 8 | Payment | `common/payment_screen.dart` | Pay & book appointment |
| 9 | Await Approval | `meeting/meetings_list_screen.dart` (Requests tab) | Status = **Pending** |
| 10 | Confirmed | `meeting/meetings_list_screen.dart` (Upcoming tab) | Status = **Confirmed** |
| 11 | Join Meeting | `meeting/webrtc_meeting_screen.dart` | P2P Video + Live Transcript |
| 12 | End Call | Hardware released | Camera + Mic OFF automatically |
| 13 | View Summary | `meeting/meeting_detail_screen.dart` | AI Diagnosis + Transcript |

---

## 🤖 AI Bot Sub-Flow

```mermaid
graph LR
    A[AiBotSelectionScreen] -->|1. Chat Bot| B[BotChatScreen\nText-based therapy\nvia Groq LLM]
    A -->|2. Voice AI| C[GeminiChatScreen\nVoice STT → Gemini → TTS]
```

---

## 📡 Backend Connections (Patient Side)

| Feature | API Call | Backend Endpoint |
|---------|----------|-----------------|
| Login | `ApiService().login()` | `POST /api/v1/auth/login` |
| Browse Therapists | `ApiService().getDoctors()` | `GET /api/v1/doctors` |
| Book Appointment | `ApiService().createAppointment()` | `POST /api/v1/appointments` |
| My Sessions | `ApiService().getPatientAppointments()` | `GET /api/v1/appointments/patient/{id}` |
| Session Summary | `ApiService().getSessionSummary()` | `GET /api/v1/sessions/{id}/summary` |
| Session Transcript | `ApiService().getSessionTranscript()` | `GET /api/v1/sessions/{id}/transcript` |
| AI Bot Chat | `ApiService().sendBotMessage()` | `POST /api/v1/bot/chat` |
| WebRTC Signaling | `SignalingService.connect()` | `WS /api/v1/webrtc/ws/{id}/{role}` |
