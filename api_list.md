# PurpleTech Telemedicine API Reference

This document outlines the Backend API endpoints and their corresponding Frontend `ApiService` methods for connecting the Flutter app to the FastAPI backend.

## 1. Authentication (`/auth`)

These endpoints manage user authentication, registration, and OTP verification.

| Backend Endpoint | Method | Frontend `ApiService` Method | Description |
| :--- | :--- | :--- | :--- |
| `/auth/login` | `POST` | `login(mobile, password)` | Authenticates a user and returns JWT tokens. |
| `/auth/register` | `POST` | `register(name, mobile, password, role, ...)` | Registers a new Doctor or Patient. |
| `/auth/verify-otp` | `POST` | `verifyOtp(mobile, otp, newPassword)` | Validates the OTP and resets the password. |
| `/auth/forgot-password` | `POST` | `forgotPassword(mobile)` | Sends an OTP to the registered mobile number. |
| `/auth/refresh-token` | `POST` | `refreshToken(token)` | Refreshes the JWT session token. |
| `/auth/logout` | `POST` | *Handled locally* | Logs out the user (clears tokens). |

## 2. Bot & AI (`/bot` & `/ai`)

Handles the AI therapy chatbot interactions, emotion analysis, and summarization.

| Backend Endpoint | Method | Frontend `ApiService` Method | Description |
| :--- | :--- | :--- | :--- |
| `/bot/start-session` | `POST` | `startBotSession(patientId)` | Initializes a new AI chat session. |
| `/bot/send-message` | `POST` | `sendBotMessage(sessionId, message)` | Sends a user message to the AI and gets a response. |
| `/bot/session-history` | `GET` | `getBotSessionHistory(sessionId)` | Retrieves the chat transcript of a session. |
| `/ai/analyze-emotion` | `POST` | *Internal / Socket* | Analyzes text/audio to detect patient emotions. |
| `/ai/session-summary` | `POST` | `getAiSessionSummary(meetingId)` | Generates a clinical summary after an appointment. |

## 3. Appointments & Calendar (`/appointments` & `/calendar`)

Handles booking, slot management, and appointment lifecycles.

| Backend Endpoint | Method | Frontend `ApiService` Method | Description |
| :--- | :--- | :--- | :--- |
| `/calendar/my-slots` | `GET` | `getDoctorSlots(doctorId)` | Gets a doctor's configured availability slots. |
| `/calendar/slots` | `POST` | `createDoctorSlot(...)` | Creates new availability slots for a doctor. |
| `/calendar/doctor/{id}/available-slots` | `GET` | `getAvailableSlots(doctorId, date)` | Fetches available slots for patient booking. |
| `/appointments/` | `POST` | `bookAppointment(...)` | Books a new appointment from an available slot. |
| `/appointments/patient/{id}` | `GET` | `getPatientAppointments(patientId)` | Retrieves all appointments for a patient. |
| `/appointments/doctor/{id}` | `GET` | `getDoctorAppointments(doctorId)` | Retrieves all appointments for a doctor. |

## 4. Meetings & WebRTC (`/meetings` & `/chat`)

Manages the live video/audio calls and real-time transcripts.

| Backend Endpoint | Method | Frontend `ApiService` Method | Description |
| :--- | :--- | :--- | :--- |
| `/meetings/start` | `POST` | `startMeeting(appointmentId)` | Initiates a WebRTC meeting room. |
| `/meetings/join` | `POST` | `joinMeeting(appointmentId)` | Connects a user to an active meeting room. |
| `/chat/ws/{meeting_id}` | `WS` | `SignalingService.connect()` | WebSocket endpoint for WebRTC signaling. |
| `/chat/{meeting_id}/history` | `GET` | `getMeetingTranscript(meetingId)` | Fetches the live transcript history of a call. |

## 5. Analytics (`/analytics`)

Provides data and statistics for the dashboard views.

| Backend Endpoint | Method | Frontend `ApiService` Method | Description |
| :--- | :--- | :--- | :--- |
| `/analytics/doctor/overview` | `GET` | `getDoctorAnalytics(doctorId)` | Fetches total patients, sessions, and ratings. |
| `/analytics/doctor/monthly` | `GET` | `getDoctorMonthlyStats(doctorId)` | Retrieves data for dashboard charts. |
| `/analytics/rate` | `POST` | `submitRating(meetingId, rating)` | Submits a post-session rating and feedback. |

## 6. Payments (`/payments`)

Handles transaction processing for bookings.

| Backend Endpoint | Method | Frontend `ApiService` Method | Description |
| :--- | :--- | :--- | :--- |
| `/payments/process` | `POST` | `processPayment(appointmentId, method)` | Processes a transaction (mocked via Razorpay/Stripe). |
| `/payments/verify` | `POST` | `verifyPayment(transactionId)` | Validates the payment signature/status. |
