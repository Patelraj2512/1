# Purple Tech - Role and Feature Workflows

Below are the visual workflows for the Purple Tech platform, broken down by **User Roles** (Patient vs. Therapist) and by **Core Features**.

## 1. Role-Wise Workflows

### A. Patient Workflow
This diagram represents the typical journey of a Patient from onboarding to completing a therapy session.

```mermaid
graph TD
    %% Patient Nodes
    Patient([Patient]) --> Auth[Login / Registration]
    Auth --> Dashboard{Patient Dashboard}
    
    %% Dashboard Main Options
    Dashboard -->|Path 1| Browse[Browse Therapists]
    Dashboard -->|Path 2| AIBot[AI Bot]
    
    %% Browse Therapists Path
    Browse --> ViewProfile[View Therapist Profile]
    ViewProfile --> Book[Select Slot & Book Appointment]
    Book --> Payment[Process Payment]
    Payment --> Wait[Wait for Therapist Approval]
    Wait --> WebRTC[Join WebRTC Video Call]
    WebRTC --> Transcript[View Live AI Transcript / Chat]
    
    %% AI Bot Path
    AIBot -->|Option 1| ChatBot[1. Chat Bot]
    AIBot -->|Option 2| VoiceAssist[2. Voice Assistant]
    AIBot -->|Option 3| AnimationBot[3. 3D Animations AI Bot]
    
    ChatBot --> BotInteract[Text-based therapy session]
    VoiceAssist --> VoiceInteract[Voice-based therapy session via STT/TTS]
    AnimationBot --> AnimInteract[Immersive 3D AI companion session]
    
    %% Styling
    classDef main fill:#e1bee7,stroke:#8e24aa,stroke-width:2px,color:#000000;
    class Patient main;
```

### B. Doctor / Therapist Workflow
This diagram represents the journey of a Therapist managing their schedule and conducting sessions.

```mermaid
graph TD
    %% Doctor Nodes
    Doctor([Therapist / Doctor]) --> Auth[Login / Registration]
    Auth --> Verify[Profile & Document Verification]
    Verify --> Dashboard{Therapist Dashboard}
    
    %% Request Management Path
    Dashboard -->|Request Management| ManageReq[View Pending Patient Requests]
    ManageReq --> ReviewReq[Review Patient Details]
    ReviewReq --> ApproveReq[Approve / Reject / Reschedule]
    ApproveReq --> Notify[Push Notification to Patient]
    
    %% Schedule Path
    Dashboard -->|Manage Schedule| Calendar[Update Calendar Availability]
    
    %% Patient History & Session Path
    Dashboard -->|Upcoming Session| History[Review Patient History & Records]
    History --> WebRTC[Start WebRTC Video Call]
    WebRTC --> LiveTools[Use Live Tools: Transcript & Chat]
    
    %% Post Session
    WebRTC --> Analytics[View Earnings & Post-Session Analytics]
    
    %% Styling
    classDef main fill:#bbdefb,stroke:#1976d2,stroke-width:2px,color:#000000;
    class Doctor main;
```

---

## 2. Feature-Wise Workflows

### A. Telemedicine Video Session (WebRTC Signaling)
This sequence diagram shows how the FastAPI backend facilitates the connection between a Patient and a Doctor so they can stream video directly to each other.

```mermaid
sequenceDiagram
    autonumber
    participant Patient as Patient App (Flutter)
    participant Server as FastAPI Signaling Server
    participant Doctor as Doctor App (Flutter)
    
    Note over Patient, Doctor: Both users join the meeting room
    
    Patient->>Server: Connect to ws://.../webrtc/ws/{id}/patient
    Doctor->>Server: Connect to ws://.../webrtc/ws/{id}/doctor
    
    Note over Patient: Generates SDP Offer
    Patient->>Server: Send SDP Offer
    Server->>Doctor: Forward SDP Offer
    
    Note over Doctor: Generates SDP Answer
    Doctor->>Server: Send SDP Answer
    Server->>Patient: Forward SDP Answer
    
    Note over Patient, Doctor: ICE Candidate Exchange (Network Routing)
    Patient->>Server: Send ICE Candidates
    Server->>Doctor: Forward ICE Candidates
    Doctor->>Server: Send ICE Candidates
    Server->>Patient: Forward ICE Candidates
    
    Note over Patient, Doctor: Direct Connection Established
    Note over Patient, Doctor: 📹 Secure P2P Audio / Video Stream 🎤
```

### B. AI Bot Interaction Flow
This sequence shows the workflow when a patient decides to interact with the AI Bot instead of a human therapist.

```mermaid
sequenceDiagram
    autonumber
    participant App as Patient App
    participant API as FastAPI Backend
    participant Engine as LLM / STT Engines (Groq)
    
    App->>App: User speaks into mic
    App->>App: Speech-to-Text conversion (Local or API)
    App->>API: POST /api/v1/bot/chat (User's text)
    
    API->>Engine: Send prompt with Therapy Context
    Note over Engine: Generates empathetic response
    Engine-->>API: Stream or return text completion
    
    API-->>App: Return AI Response text
    App->>App: Text-to-Speech (TTS) readout to Patient
    App->>App: Update UI Chat Bubble
```

### C. Appointment Booking & Notification Flow
How a booking is made and the therapist is notified.

```mermaid
sequenceDiagram
    autonumber
    participant Patient
    participant API as Backend Database
    participant Firebase as FCM / Push Notifications
    participant Therapist
    
    Patient->>API: POST /api/v1/appointments (Select slot)
    API->>API: Mark slot as 'Pending' in DB
    API->>Firebase: Trigger "New Appointment Request"
    Firebase-->>Therapist: 📱 Push Notification received
    
    Therapist->>API: PUT /api/v1/appointments/{id}/approve
    API->>API: Mark slot as 'Booked' in DB
    API->>Firebase: Trigger "Appointment Approved"
    Firebase-->>Patient: 📱 Push Notification received
```
