# PurpleTech Telemedicine API Reference

This document outlines the Backend API endpoints and their corresponding Frontend `ApiService` methods for connecting the Flutter app to the FastAPI backend, including **all required and optional parameters**.

---

## 1. Authentication (`/auth`)

These endpoints manage user authentication, registration, and OTP verification.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Description |
| :--- | :--- | :--- | :--- | :--- |
| `/auth/login` | `POST` | `login()` | **Body:**<br>- `mobile` (str)<br>- `password` (str) | Authenticates a user and returns JWT access & refresh tokens. |
| `/auth/register` | `POST` | `register()` | **Body:**<br>- `name` (str)<br>- `mobile` (str)<br>- `password` (str)<br>- `role` (str: "doctor" or "patient")<br>- `specialization` (str, optional)<br>- `age` (int, optional)<br>- `gender` (str, optional) | Registers a new Doctor or Patient. |
| `/auth/verify-otp` | `POST` | `verifyOtp()` | **Body:**<br>- `mobile` (str)<br>- `otp` (str)<br>- `new_password` (str) | Validates the OTP and resets the password. |
| `/auth/forgot-password` | `POST` | `forgotPassword()` | **Body:**<br>- `mobile` (str) | Sends an OTP to the registered mobile number. |
| `/auth/refresh-token` | `POST` | `refreshToken()` | **Body:**<br>- `refresh_token` (str) | Refreshes the JWT session token. |
| `/auth/logout` | `POST` | *Handled locally* | *None* | Logs out the user (clears tokens client-side). |

---

## 2. Bot & AI (`/bot` & `/ai`)

Handles the AI therapy chatbot interactions, emotion analysis, and summarization.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Description |
| :--- | :--- | :--- | :--- | :--- |
| `/bot/start-session` | `POST` | `startBotSession()` | *Internal Params* | Initializes a new AI chat session. |
| `/bot/send-message` | `POST` | `sendBotMessage()` | *Internal Params* | Sends a user message to the AI and gets a response. |
| `/bot/session-history` | `GET` | `getBotSessionHistory()` | **Query:**<br>- `session_id` (str) | Retrieves the chat transcript of a session. |
| `/ai/analyze-emotion` | `POST` | *Internal / Socket* | **Body:**<br>- `session_id` (str)<br>- `text` (str) | Analyzes text/audio to detect patient emotions. |
| `/ai/session-summary` | `POST` | `getAiSessionSummary()` | **Body:**<br>- `session_id` (str) | Generates a clinical summary after an appointment. |
| `/ai/generate-questions` | `POST` | *Internal* | **Body:**<br>- `session_id` (str)<br>- `context` (str, optional) | AI generates dynamic follow-up questions for the doctor. |

---

## 3. Appointments & Calendar (`/appointments` & `/calendar`)

Handles booking, slot management, and appointment lifecycles.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Description |
| :--- | :--- | :--- | :--- | :--- |
| `/calendar/my-slots` | `GET` | `getDoctorSlots()` | **Path/Auth:** Uses JWT `user_id` | Gets a doctor's configured availability slots. |
| `/calendar/slots` | `POST` | `createDoctorSlot()` | *Depends on router schema* | Creates new availability slots for a doctor. |
| `/calendar/doctor/{doctor_id}/available-slots` | `GET` | `getAvailableSlots()` | **Path:**<br>- `doctor_id` (str) | Fetches available slots for patient booking. |
| `/appointments/` | `POST` | `bookAppointment()` | **Body:**<br>- `doctor_id` (str)<br>- `patient_id` (str)<br>- `appointment_date` (datetime)<br>- `session_type` (str, default "video")<br>- `payment_id` (str, optional) | Books a new appointment from an available slot. |
| `/appointments/{appointment_id}/status` | `PUT` | `updateAppointment()` | **Path:**<br>- `appointment_id` (str)<br>**Body:**<br>- `status` (str) | Updates the status of an appointment. |
| `/appointments/patient/{patient_id}` | `GET` | `getPatientAppointments()` | **Path:**<br>- `patient_id` (str) | Retrieves all appointments for a patient. |
| `/appointments/doctor/{doctor_id}` | `GET` | `getDoctorAppointments()` | **Path:**<br>- `doctor_id` (str) | Retrieves all appointments for a doctor. |

---

## 4. Meetings & WebRTC (`/meetings` & `/chat`)

Manages the live video/audio calls and real-time transcripts.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Description |
| :--- | :--- | :--- | :--- | :--- |
| `/meetings/start` | `POST` | `startMeeting()` | **Body:**<br>- `doctor_id` (str)<br>- `password` (str)<br>- `patient_id` (str, optional) | Initiates a WebRTC meeting room. |
| `/meetings/join` | `POST` | `joinMeeting()` | **Body:**<br>- `meeting_id` (str)<br>- `password` (str)<br>- `patient_id` (str, optional) | Connects a user to an active meeting room. |
| `/chat/ws/{meeting_id}` | `WS` | `SignalingService.connect()` | **Path:**<br>- `meeting_id` (str) | WebSocket endpoint for WebRTC signaling. |
| `/chat/{meeting_id}/history` | `GET` | `getMeetingTranscript()` | **Path:**<br>- `meeting_id` (str) | Fetches the live transcript history of a call. |

---

## 5. Analytics (`/analytics`)

Provides data and statistics for the dashboard views.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Description |
| :--- | :--- | :--- | :--- | :--- |
| `/analytics/doctor/overview` | `GET` | `getDoctorAnalytics()` | **Auth:** Requires Doctor JWT token | Fetches total patients, sessions, and ratings. |
| `/analytics/doctor/monthly` | `GET` | `getDoctorMonthlyStats()` | **Auth:** Requires Doctor JWT token | Retrieves data for dashboard charts. |
| `/analytics/rate` | `POST` | `submitRating()` | *Depends on router schema* | Submits a post-session rating and feedback. |

---

## 6. Payments (`/payments`)

Handles transaction processing for bookings.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Description |
| :--- | :--- | :--- | :--- | :--- |
| `/payments/process` | `POST` | `processPayment()` | **Body:**<br>- `appointment_id` (str)<br>- `patient_id` (str)<br>- `doctor_id` (str)<br>- `amount` (int, in cents)<br>- `currency` (str, optional, default "INR") | Processes a transaction (mocked via Razorpay/Stripe). |
| `/payments/verify` | `POST` | `verifyPayment()` | **Body:**<br>- `payment_id` (str)<br>- `razorpay_payment_id` (str)<br>- `razorpay_signature` (str) | Validates the payment signature/status. |

---

## 7. Therapist Verification (`/verification`)

Handles credential submissions for doctors.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Description |
| :--- | :--- | :--- | :--- | :--- |
| *Various Verification endpoints* | `POST` | *Internal* | **Body:**<br>- `doctor_id` (str)<br>- `licence_number` (str)<br>- `registration_number` (str)<br>- `specialization` (str)<br>- `experience_years` (int)<br>- `linkedin_url` (str, optional)<br>- `documents` (List[str]) | Therapist credentials verification payload. |

---

## 8. Frontend Navigation Structure

While the frontend uses Flutter routing rather than traditional API endpoints, these are the primary "Screens" that act as the application's client-side endpoints.

### Authentication Flow
* `SplashScreen`: Initial app load and session verification.
* `RoleSelectionScreen`: User chooses between Doctor or Patient.
* `LoginSignupScreen`: The WhatsApp-style mobile number entry screen.
* `OtpScreen`: The 6-digit verification input that routes to the dashboards.
* `ProfileSetupScreen`: Mandatory details collection for first-time users.

### Patient Dashboards & Features
* `PatientDashboardScreen`: Main hub containing quick actions, tips, and active sessions.
* `TherapistFilterScreen`: Search and filter catalog for finding doctors.
* `PatientCalendarScreen`: Calendar view for checking appointment schedules.
* `AiBotSelectionScreen`: Hub for choosing between Gemini chat and Animation Bot.
* `PaymentScreen`: Handles transaction logic before finalizing an appointment.

### Doctor Dashboards & Features
* `DoctorDashboardScreen`: Main hub for doctors containing upcoming sessions and alerts.
* `CalendarScreen` (Doctor): Slot creation, booking management, and meeting triggers.
* `AnalyticsScreen`: Detailed insights, rating metrics, and patient volume charts.
* `PatientsListScreen`: CRM view of all assigned patients.

### Meetings & WebRTC
* `WebRTCMeetingScreen`: The core live video/audio conferencing interface with real-time transcription.
* `MeetingDetailScreen`: Post-session summary, diagnosis notes, and AI summary triggers.
* `ExportScreen`: Utility to export meeting data (audio/video/txt).

---

## 9. Pending / Need to Create APIs (To-Do List)

The following APIs and corresponding frontend features are identified as necessary additions for a complete, production-ready Telemedicine platform but are **currently not implemented**.

### 1. User Profile & File Management
* `PUT /profile/upload-avatar`: Upload and update the user's profile picture.
* `PUT /profile/update`: Edit personal details (name, email, age, gender) after registration.
* `POST /medical-records/upload`: Upload patient medical records, lab reports, or PDF prescriptions.
* `GET /medical-records/patient/{patient_id}`: Retrieve a list of a patient's historical medical documents.

### 2. Search & Discovery
* `GET /doctors/search`: Advanced search endpoint with query parameters for filtering therapists by `specialization`, `price_range`, `rating`, `availability`, and `languages`.
* `GET /doctors/{doctor_id}/reviews`: Fetch paginated patient reviews and ratings for a specific doctor.

### 3. Asynchronous Messaging
* `POST /messages/send`: Send a direct text message between a Doctor and Patient outside of an active video call.
* `GET /messages/{conversation_id}`: Retrieve chat history for asynchronous messaging.
* `PUT /messages/{message_id}/read`: Mark a direct message as read.

### 4. Prescriptions & Clinical Notes
* `POST /prescriptions/create`: Allow a doctor to generate and securely sign a digital prescription post-session.
* `GET /prescriptions/{appointment_id}`: Fetch the specific prescription generated for an appointment.

### 5. Advanced Video & WebRTC Features
* `GET /meetings/{meeting_id}/recording`: Fetch the securely stored video recording of a past consultation.
* `POST /meetings/{meeting_id}/mute-participant`: Host control to mute a participant.

### 6. Subscriptions & Packages
* `GET /packages/list`: Fetch available therapy session bundles (e.g., "4 Sessions / Month").
* `POST /packages/purchase`: Process payment for a subscription bundle and allocate credits to the user's account.
