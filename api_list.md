# PurpleTech Telemedicine API Reference

This document outlines the Backend API endpoints and their corresponding Frontend `ApiService` methods for connecting the Flutter app to the FastAPI backend, including **all required/optional parameters** and **expected responses**.

---

## 1. Authentication (`/auth`)

These endpoints manage user authentication, registration, and OTP verification.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Expected Response |
| :--- | :--- | :--- | :--- | :--- |
| `/auth/login` | `POST` | `login()` | **Body:**<br>- `mobile` (str)<br>- `password` (str) | `{ "access_token": "...", "refresh_token": "...", "token_type": "bearer", "user_id": "...", "role": "...", "name": "..." }` |
| `/auth/register` | `POST` | `register()` | **Body:**<br>- `name` (str)<br>- `mobile` (str)<br>- `password` (str)<br>- `role` (str: "doctor"\|"patient")<br>- `specialization` (str, opt)<br>- `age` (int, opt)<br>- `gender` (str, opt) | `{ "access_token": "...", "refresh_token": "...", "token_type": "bearer", "user_id": "...", "role": "...", "name": "..." }` |
| `/auth/verify-otp` | `POST` | `verifyOtp()` | **Body:**<br>- `mobile` (str)<br>- `otp` (str)<br>- `new_password` (str) | `{ "message": "Password reset successful" }` |
| `/auth/forgot-password` | `POST` | `forgotPassword()` | **Body:**<br>- `mobile` (str) | `{ "message": "OTP sent to mobile number" }` |
| `/auth/refresh-token` | `POST` | `refreshToken()` | **Body:**<br>- `refresh_token` (str) | `{ "access_token": "...", "refresh_token": "...", "token_type": "bearer", ... }` |
| `/auth/logout` | `POST` | *Handled locally* | *None* | `{ "message": "Logged out successfully" }` |

---

## 2. Bot & AI (`/bot` & `/ai`)

Handles the AI therapy chatbot interactions, emotion analysis, and summarization.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path / Query) | Expected Response |
| :--- | :--- | :--- | :--- | :--- |
| `/bot/start-session` | `POST` | `startBotSession()` | *Internal Params* | `{ "session_id": "..." }` |
| `/bot/send-message` | `POST` | `sendBotMessage()` | *Internal Params* | `{ "response": "AI text response..." }` |
| `/bot/session-history` | `GET` | `getBotSessionHistory()` | **Query:**<br>- `session_id` (str) | `[ { "role": "user", "message": "..." }, ... ]` |
| `/ai/analyze-emotion` | `POST` | *Internal / Socket* | **Body:**<br>- `session_id` (str)<br>- `text` (str) | `{ "emotion": "anxious", "confidence": 0.85 }` |
| `/ai/session-summary` | `POST` | `getAiSessionSummary()` | **Body:**<br>- `session_id` (str) | `{ "summary": "...", "key_points": ["...", "..."] }` |
| `/ai/generate-questions` | `POST` | *Internal* | **Body:**<br>- `session_id` (str)<br>- `context` (str, opt) | `{ "questions": ["Question 1?", "Question 2?"] }` |

---

## 3. Appointments & Calendar (`/appointments` & `/calendar`)

Handles booking, slot management, and appointment lifecycles.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Expected Response |
| :--- | :--- | :--- | :--- | :--- |
| `/calendar/my-slots` | `GET` | `getDoctorSlots()` | **Auth:** Uses JWT `user_id` | `[ { "id": "...", "date": "...", "start_time": "...", "end_time": "..." } ]` |
| `/calendar/slots` | `POST` | `createDoctorSlot()` | *Depends on router schema* | `{ "id": "...", "date": "...", "start_time": "...", "end_time": "..." }` |
| `/calendar/doctor/{id}/available-slots` | `GET` | `getAvailableSlots()` | **Path:**<br>- `doctor_id` (str) | `[ { "id": "...", "start_time": "...", "end_time": "..." } ]` |
| `/appointments/` | `POST` | `bookAppointment()` | **Body:**<br>- `doctor_id` (str)<br>- `patient_id` (str)<br>- `appointment_date` (datetime)<br>- `session_type` (str)<br>- `payment_id` (str, opt) | `{ "id": "...", "status": "scheduled", "created_at": "...", "doctor": {...}, "patient": {...} }` |
| `/appointments/{id}/status` | `PUT` | `updateAppointment()` | **Path:**<br>- `appointment_id` (str)<br>**Body:**<br>- `status` (str) | `{ "id": "...", "status": "updated_status", ... }` |
| `/appointments/patient/{id}` | `GET` | `getPatientAppointments()` | **Path:**<br>- `patient_id` (str) | `[ { "id": "...", "status": "...", "doctor": {...} } ]` |
| `/appointments/doctor/{id}` | `GET` | `getDoctorAppointments()` | **Path:**<br>- `doctor_id` (str) | `[ { "id": "...", "status": "...", "patient": {...} } ]` |

---

## 4. Meetings & WebRTC (`/meetings` & `/chat`)

Manages the live video/audio calls and real-time transcripts.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Expected Response |
| :--- | :--- | :--- | :--- | :--- |
| `/meetings/start` | `POST` | `startMeeting()` | **Body:**<br>- `doctor_id` (str)<br>- `password` (str)<br>- `patient_id` (str, opt) | `{ "id": "...", "meeting_id": "...", "doctor_id": "...", "status": "active" }` |
| `/meetings/join` | `POST` | `joinMeeting()` | **Body:**<br>- `meeting_id` (str)<br>- `password` (str)<br>- `patient_id` (str, opt) | `{ "id": "...", "meeting_id": "...", "status": "active" }` |
| `/chat/ws/{meeting_id}` | `WS` | `SignalingService.connect()` | **Path:**<br>- `meeting_id` (str) | *WebSocket Connection Stream* |
| `/chat/{meeting_id}/history` | `GET` | `getMeetingTranscript()` | **Path:**<br>- `meeting_id` (str) | `[ { "id": "...", "speaker": "...", "message": "...", "is_final": true, "timestamp": "..." } ]` |

---

## 5. Analytics (`/analytics`)

Provides data and statistics for the dashboard views.

| Backend Endpoint | Method | Frontend Method | Parameters | Expected Response |
| :--- | :--- | :--- | :--- | :--- |
| `/analytics/doctor/overview` | `GET` | `getDoctorAnalytics()` | **Auth:** Requires Doctor JWT token | `{ "total_patients": 10, "total_sessions": 25, "rating": 4.8 }` |
| `/analytics/doctor/monthly` | `GET` | `getDoctorMonthlyStats()` | **Auth:** Requires Doctor JWT token | `[ { "month": "Jan", "sessions": 12 }, ... ]` |
| `/analytics/rate` | `POST` | `submitRating()` | *Depends on router schema* | `{ "message": "Rating submitted successfully" }` |

---

## 6. Payments (`/payments`)

Handles transaction processing for bookings.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Expected Response |
| :--- | :--- | :--- | :--- | :--- |
| `/payments/process` | `POST` | `processPayment()` | **Body:**<br>- `appointment_id` (str)<br>- `patient_id` (str)<br>- `doctor_id` (str)<br>- `amount` (int)<br>- `currency` (str) | `{ "payment_id": "...", "razorpay_order_id": "...", "amount": 1000, "currency": "INR", "status": "created" }` |
| `/payments/verify` | `POST` | `verifyPayment()` | **Body:**<br>- `payment_id` (str)<br>- `razorpay_payment_id` (str)<br>- `razorpay_signature` (str) | `{ "success": true, "message": "Payment verified", "status": "paid" }` |

---

## 7. Therapist Verification (`/verification`)

Handles credential submissions for doctors.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Expected Response |
| :--- | :--- | :--- | :--- | :--- |
| *Verification endpoints* | `POST` | *Internal* | **Body:**<br>- `doctor_id` (str)<br>- `licence_number` (str)<br>- `registration_number` (str)<br>- `specialization` (str)<br>- `experience_years` (int)<br>- `documents` (List[str]) | `{ "id": "...", "doctor_id": "...", "status": "pending", "documents": ["..."] }` |

---

## 8. Doctors & Directory (`/doctors`)

Manages the public directory and information for therapists.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Expected Response |
| :--- | :--- | :--- | :--- | :--- |
| `/doctors/` | `GET` | `getDoctorsList()` | *None* | `[ { "id": "...", "name": "...", "specialization": "..." }, ... ]` |
| `/doctors/` | `POST` | *Internal* | **Body:** Doctor details | `{ "id": "...", "name": "..." }` |
| `/doctors/{id}` | `PUT` | *Internal* | **Path:** `doctor_id`<br>**Body:** Updates | `{ "id": "...", "name": "..." }` |

---

## 9. Notifications (`/notifications`)

In-app notification delivery and read-state management.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Expected Response |
| :--- | :--- | :--- | :--- | :--- |
| `/notifications/{user_id}` | `GET` | `getNotifications()` | **Path:** `user_id` (str) | `[ { "id": "...", "title": "...", "message": "...", "is_read": 0 } ]` |
| `/notifications/{id}/read` | `PUT` | `markNotificationRead()` | **Path:** `notification_id` (str) | `{ "id": "...", "is_read": 1, ... }` |

---

## 10. App Config & Exports (`/bottom_bar` & `/stats`)

Handles global settings, AI models, language selection, and data exports.

| Backend Endpoint | Method | Frontend Method | Parameters (Body / Path) | Expected Response |
| :--- | :--- | :--- | :--- | :--- |
| `/bottom_bar/languages` | `GET` | *Internal* | *None* | `[ "en", "hi", "gu", ... ]` |
| `/bottom_bar/models` | `GET` | *Internal* | *None* | `[ "gemini-3", "gpt-4", ... ]` |
| `/bottom_bar/export/txt` | `POST` | `exportTxt()` | **Body:** `meeting_id` | `{ "download_url": "..." }` |
| `/bottom_bar/export/audio` | `POST` | `exportAudio()` | **Body:** `meeting_id` | `{ "download_url": "..." }` |

---

## 11. Frontend Navigation Structure

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

## 12. Pending / Need to Create APIs (To-Do List)

The following APIs and corresponding frontend features are identified as necessary additions for a complete, production-ready Telemedicine platform but are **currently not implemented**.

### 1. User Profile & File Management
* `PUT /profile/upload-avatar`
  * **Parameters (Body):** `file` (Multipart Image)
  * **Expected Response:** `{ "avatar_url": "https://...", "message": "Avatar updated" }`
* `PUT /profile/update`
  * **Parameters (Body):** `name` (str), `email` (str), `age` (int), `gender` (str)
  * **Expected Response:** `{ "id": "...", "name": "...", "email": "...", "updated_at": "..." }`
* `POST /medical-records/upload`
  * **Parameters (Body):** `patient_id` (str), `file` (Multipart PDF/Image), `document_type` (str)
  * **Expected Response:** `{ "document_id": "...", "url": "...", "uploaded_at": "..." }`
* `GET /medical-records/patient/{patient_id}`
  * **Parameters (Path):** `patient_id` (str)
  * **Expected Response:** `[ { "id": "...", "document_type": "...", "url": "...", "date": "..." }, ... ]`

### 2. Search & Discovery
* `GET /doctors/search`
  * **Parameters (Query):** `specialization` (str), `max_price` (int), `min_rating` (float)
  * **Expected Response:** `[ { "doctor_id": "...", "name": "...", "specialization": "...", "rating": 4.8 }, ... ]`
* `GET /doctors/{doctor_id}/reviews`
  * **Parameters (Path):** `doctor_id` (str)
  * **Expected Response:** `[ { "reviewer_name": "...", "rating": 5, "comment": "...", "date": "..." }, ... ]`

### 3. Asynchronous Messaging
* `POST /messages/send`
  * **Parameters (Body):** `receiver_id` (str), `content` (str)
  * **Expected Response:** `{ "message_id": "...", "status": "sent", "timestamp": "..." }`
* `GET /messages/{conversation_id}`
  * **Parameters (Path):** `conversation_id` (str)
  * **Expected Response:** `[ { "sender_id": "...", "content": "...", "timestamp": "...", "is_read": bool }, ... ]`
* `PUT /messages/{message_id}/read`
  * **Parameters (Path):** `message_id` (str)
  * **Expected Response:** `{ "success": true, "status": "read" }`

### 4. Prescriptions & Clinical Notes
* `POST /prescriptions/create`
  * **Parameters (Body):** `appointment_id` (str), `medications` (List[Dict]), `notes` (str)
  * **Expected Response:** `{ "prescription_id": "...", "download_url": "...", "created_at": "..." }`
* `GET /prescriptions/{appointment_id}`
  * **Parameters (Path):** `appointment_id` (str)
  * **Expected Response:** `{ "prescription_id": "...", "medications": [...], "notes": "...", "doctor_signature": "..." }`

### 5. Advanced Video & WebRTC Features
* `GET /meetings/{meeting_id}/recording`
  * **Parameters (Path):** `meeting_id` (str)
  * **Expected Response:** `{ "recording_url": "https://...", "duration_seconds": 1800, "expires_at": "..." }`
* `POST /meetings/{meeting_id}/mute-participant`
  * **Parameters (Path):** `meeting_id` (str) <br>**Parameters (Body):** `participant_id` (str)
  * **Expected Response:** `{ "status": "muted", "participant_id": "..." }`

### 6. Subscriptions & Packages
* `GET /packages/list`
  * **Parameters:** *None*
  * **Expected Response:** `[ { "package_id": "...", "name": "4 Sessions/Month", "price": 5000, "credits": 4 }, ... ]`
* `POST /packages/purchase`
  * **Parameters (Body):** `package_id` (str), `payment_method` (str)
  * **Expected Response:** `{ "transaction_id": "...", "status": "success", "credits_added": 4 }`
