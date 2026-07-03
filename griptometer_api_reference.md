# Griptometer — Complete Backend API Reference

**Base URL:** `https://griptometer.invisiblefiction.com/v1`  
**Auth Header:** `Authorization: Bearer <token>`  
**Content-Type:** `application/json`  
**App Version:** 1.0.3+4

> 🟢 = No auth required &nbsp;&nbsp; 🔐 = Bearer token required  
> 🆕 = New field added in latest release

---

## GROUP 1 — Authentication

---

### 1.1 Register Clinician

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `POST /auth/register` |
| **Auth** | 🟢 Public |
| **Frontend Method** | `LaravelApiService.register()` → called from `AuthProvider.register()` → triggered by `RegisterScreen` on submit |

**Request Body (JSON):**
```json
{
  "name":                  "Dr. Raj Patel",
  "email":                 "raj@clinic.com",
  "password":              "secret123",
  "password_confirmation": "secret123",
  "phone_number":          "+919876543210",
  "clinic":                "City Rehabilitation Center",
  "discipline":            "Physiotherapy"
}
```

| Parameter | Type | Required | Notes |
|-----------|------|:--------:|-------|
| `name` | string | ✅ | Full name of clinician |
| `email` | string | ✅ | Must be unique |
| `password` | string | ✅ | Min 6 chars |
| `password_confirmation` | string | ✅ | Must match `password` |
| `phone_number` | string | ✅ | Include country code |
| `clinic` 🆕 | string | ❌ | Hospital / clinic name |
| `discipline` 🆕 | string | ❌ | e.g. Physiotherapy, OT, Orthopaedics |

**Expected Response `201`:**
```json
{
  "user_profile": {
    "uid": "uuid-xxxx",
    "name": "Dr. Raj Patel",
    "email": "raj@clinic.com",
    "phone_number": "+919876543210",
    "clinic": "City Rehabilitation Center",
    "discipline": "Physiotherapy"
  },
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Error Response `422`:**
```json
{ "error": { "message": "The email has already been taken." } }
```

---

### 1.2 Login

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `POST /auth/login` |
| **Auth** | 🟢 Public |
| **Frontend Method** | `LaravelApiService.login()` → called from `AuthProvider.login()` → triggered by `LoginScreen` on submit |

**Request Body (JSON):**
```json
{
  "email":    "raj@clinic.com",
  "password": "secret123"
}
```

| Parameter | Type | Required | Notes |
|-----------|------|:--------:|-------|
| `email` | string | ✅ | |
| `password` | string | ✅ | |

**Expected Response `200`:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user_profile": {
    "uid": "uuid-xxxx",
    "name": "Dr. Raj Patel",
    "email": "raj@clinic.com",
    "phone_number": "+919876543210",
    "clinic": "City Rehabilitation Center",
    "discipline": "Physiotherapy"
  }
}
```

**Error Response `401`:**
```json
{ "error": { "message": "Invalid credentials." } }
```

---

### 1.3 Logout

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `POST /auth/logout` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.logout()` → called from `AuthProvider.logout()` → triggered by logout button |

**Request Body:** _(empty)_

**Expected Response `200`:**
```json
{ "success": true }
```

---

## GROUP 2 — User / Clinician Profile

---

### 2.1 Get My Profile

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `GET /users/me` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.getMe()` → called from `AuthProvider.refreshUserProfile()` → triggered on app start, network restore |

**Request Body:** _(none)_

**Expected Response `200`:**
```json
{
  "user_profile": {
    "uid": "uuid-xxxx",
    "name": "Dr. Raj Patel",
    "email": "raj@clinic.com",
    "phone_number": "+919876543210",
    "clinic": "City Rehabilitation Center",
    "discipline": "Physiotherapy",
    "created_at": "2026-01-01T00:00:00Z"
  }
}
```

> Frontend reads: `res['user_profile'] ?? res['user'] ?? res`  
> Maps: `uid` or `id` → stored as `laravel_id` locally  
> Maps: `phone_number` or `phone` → stored as `userPhone`

---

### 2.2 Update My Profile

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `PUT /users/me` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.updateMe()` → called from `AuthProvider.updateAccountDetails()` → triggered by `AccountManagementScreen` save button |

**Request Body (JSON):**
```json
{
  "name":         "Dr. Raj Patel",
  "phone_number": "+919876543210",
  "clinic":       "City Rehabilitation Center",
  "discipline":   "Physiotherapy"
}
```

| Parameter | Type | Required | Notes |
|-----------|------|:--------:|-------|
| `name` | string | ❌ | |
| `phone_number` | string | ❌ | |
| `clinic` 🆕 | string | ❌ | Hospital / clinic name |
| `discipline` 🆕 | string | ❌ | e.g. Physiotherapy, OT |

**Expected Response `200`:** Updated `user_profile` object (same as GET /users/me).

---

### 2.3 Delete My Account

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `DELETE /users/me` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `AuthProvider.deleteAccount()` → triggered by `AccountManagementScreen` delete account button |

> ⚠️ Note: Currently the app deletes locally only. A `DELETE /users/me` call should also be wired into `LaravelApiService` for full cloud deletion.

**Expected Response `200`:**
```json
{ "success": true, "message": "Account deleted successfully." }
```

---

## GROUP 3 — Patients

---

### 3.1 List All Patients

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `GET /patients` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.getPatients()` → called from `SyncService.pullRemoteData()` → triggered on sync |

**Query Parameters:** _(none currently used)_

**Expected Response `200`:**
```json
[
  {
    "uid": "uuid-xxxx",
    "name": "John Doe",
    "gender": "Male",
    "dob": "2010-05-15",
    "age": 16,
    "handedness": "Right",
    "weight_kg": 35.5,
    "height_cm": 130.0,
    "bmi": 21.0,
    "created_at": "2026-06-01T00:00:00Z",
    "is_child_clinical": true,
    "medical_diagnosis": "Cerebral Palsy",
    "affected_side": "Left"
  }
]
```

> Frontend also accepts: `{ "data": [...] }` or `{ "patients": [...] }`

---

### 3.2 Create Patient

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `POST /patients` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.addPatient()` → called from `SyncService.syncAll()` (auto-sync) and `PatientProvider.addPatient()` |

**Request Body (JSON):**
```json
{
  "name":       "John Doe",
  "dob":        "2010-05-15",
  "gender":     "Male",
  "handedness": "Right",
  "weight_kg":  35.5,
  "height_cm":  130.0,

  "is_child_clinical":    true,
  "medical_diagnosis":    "Cerebral Palsy",
  "date_of_diagnosis":    "2012-03-10",
  "duration_of_condition":"4 years",
  "affected_side":        "Left",

  "has_previous_surgery":  true,
  "surgery_details":       "Tendon transfer 2018",
  "has_previous_fracture": false,
  "medication_history":    "Baclofen 10mg daily"
}
```

| Parameter | Type | Required | Notes |
|-----------|------|:--------:|-------|
| `name` | string | ✅ | |
| `dob` | date string | ✅ | Format: `YYYY-MM-DD` |
| `gender` | string | ✅ | `Male` / `Female` / `Other` |
| `handedness` | string | ✅ | `Right` / `Left` / `Ambidextrous` |
| `weight_kg` | float | ✅ | |
| `height_cm` | float | ✅ | |
| `is_child_clinical` 🆕 | boolean | ❌ | Default `false` |
| `medical_diagnosis` 🆕 | string | ❌ | Required if `is_child_clinical = true` |
| `date_of_diagnosis` 🆕 | date string | ❌ | Format: `YYYY-MM-DD` |
| `duration_of_condition` 🆕 | string | ❌ | e.g. "6 months", "2 years" |
| `affected_side` 🆕 | string | ❌ | `Right` / `Left` / `Bilateral` |
| `has_previous_surgery` 🆕 | boolean | ❌ | |
| `surgery_details` 🆕 | string | ❌ | Required if `has_previous_surgery = true` |
| `has_previous_fracture` 🆕 | boolean | ❌ | |
| `medication_history` 🆕 | string | ❌ | |

**Expected Response `201`:**
```json
{
  "uid": "uuid-yyyy",
  "name": "John Doe",
  "created_at": "2026-07-03T08:00:00Z"
}
```

> Frontend reads `uid` or `id` from response and stores it as `laravel_id`.

---

### 3.3 Get Single Patient

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `GET /patients/{uid}` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.getPatient(uid)` |

**Path Parameter:**
| Parameter | Type | Required |
|-----------|------|:--------:|
| `uid` | string (UUID) | ✅ |

**Expected Response `200`:** Full patient object (same fields as POST, including all child clinical fields).

---

### 3.4 Update Patient

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `PUT /patients/{uid}` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.updatePatient(uid, data)` → called from `PatientProvider.updatePatient()` |

**Path Parameter:** `uid` (UUID)  
**Request Body:** Same fields as `POST /patients` — all optional.

**Expected Response `200`:** Updated patient object.

---

### 3.5 Delete Patient

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `DELETE /patients/{uid}` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.deletePatient(uid)` → called from `PatientProvider.deletePatient()` |

**Path Parameter:** `uid` (UUID)

**Expected Response `204` or `200`:** _(empty or `{ "success": true }`)_

---

### 3.6 List Patient Sessions

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `GET /patients/{uid}/sessions` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.getPatientSessions(uid)` → called from `SyncService.pullRemoteData()` |

**Path Parameter:** `uid` (patient UUID)  
**Query Parameters:**
| Parameter | Type | Default | Notes |
|-----------|------|---------|-------|
| `per_page` | integer | `50` | Results per page |

**Expected Response `200`:**
```json
{
  "data": [
    {
      "id": 42,
      "session_id": 42,
      "game_name": "Rocket Ascent",
      "session_start": "2026-07-03T08:00:00Z",
      "session_end": "2026-07-03T08:01:00Z",
      "duration": 60,
      "duration_seconds": 60,
      "component": "Strength",
      "grip_type": "Cylindrical",
      "attempt1": 21.5,
      "attempt2": 22.0,
      "attempt3": 20.8,
      "attempts_average": 21.43,
      "latest_grip_strength": 22.0,
      "score": 1240,
      "sensor_windows": [
        {
          "window_index": 0,
          "s1_avg": 22.0,
          "s2_avg": 21.5,
          "s3_avg": 20.8,
          "s4_avg": 19.2,
          "sensor_avg": 20.87
        }
      ]
    }
  ],
  "total": 1,
  "per_page": 50,
  "current_page": 1
}
```

---

### 3.7 Get Single Session

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `GET /patients/{uid}/sessions/{sessionId}` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.getPatientSession(patientUid, sessionId)` |

**Path Parameters:**
| Parameter | Type | Required |
|-----------|------|:--------:|
| `uid` | string (UUID) | ✅ |
| `sessionId` | integer or string | ✅ |

**Expected Response `200`:** Single session object (same structure as one item in list above).

---

## GROUP 4 — Devices

---

### 4.1 List Registered Devices

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `GET /devices` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.getDevices()` → called from `AuthProvider.fetchUserDevices()` → triggered on login and app start |

**Expected Response `200`:**
```json
[
  {
    "id": 1,
    "device_name": "Griptometer Ball 1",
    "mac_address": "AA:BB:CC:DD:EE:FF",
    "serial_id": "GRP-001",
    "sensor_count": 4,
    "status": "active",
    "registered_at": "2026-06-01T00:00:00Z",
    "offsets": [0, 0, 0, 0],
    "gains": [1.0, 1.0, 1.0, 1.0]
  }
]
```

> Frontend also accepts: `{ "data": [...] }` or `{ "devices": [...] }`

---

### 4.2 Register a Device

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `POST /devices` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.addDevice()` → called from `SyncService.syncAll()` (auto-sync when unsynced device detected) |

**Request Body (JSON):**
```json
{
  "device_name": "Griptometer Ball 1",
  "mac_address": "AA:BB:CC:DD:EE:FF",
  "serial_id":   "GRP-001",
  "sensor_count": 4,
  "mfg_data":    "optional manufacturer hex data",
  "offsets":     [0, 0, 0, 0],
  "gains":       [1.0, 1.0, 1.0, 1.0]
}
```

| Parameter | Type | Required | Notes |
|-----------|------|:--------:|-------|
| `device_name` | string | ✅ | Friendly name |
| `mac_address` | string | ✅ | BLE MAC address |
| `serial_id` | string | ❌ | Manufacturer serial |
| `sensor_count` | integer | ❌ | Default 4 |
| `mfg_data` | string | ❌ | BLE advertisement manufacturer data |
| `offsets` | int array | ❌ | Calibration offsets per sensor |
| `gains` | float array | ❌ | Calibration gains per sensor |

**Expected Response `201`:**
```json
{
  "id": 1,
  "device_name": "Griptometer Ball 1",
  "mac_address": "AA:BB:CC:DD:EE:FF"
}
```

---

## GROUP 5 — Session Lifecycle

> Sessions follow a 3-step lifecycle: **Start → Upload Windows (batch) → End**

---

### 5.1 Start Session

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `POST /sessions/start` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.startSession()` → called from `SyncService.syncAll()` (offline sync) and live game flow |

**Request Body (JSON):**
```json
{
  "patient_uid":   "uuid-xxxx",
  "game_name":     "Rocket Ascent",
  "session_start": "2026-07-03T08:00:00Z",

  "component":              "Strength",
  "grip_type":              "Cylindrical",
  "difficulty":             "Medium",
  "hand":                   "Right",
  "number_of_repetitions":  3,
  "number_of_grips":        3,
  "left_max_grip_strength":  18.5,
  "right_max_grip_strength": 22.0,
  "next_assessment_date":   "2026-07-10"
}
```

| Parameter | Type | Required | Notes |
|-----------|------|:--------:|-------|
| `patient_uid` | string (UUID) | ✅ | Backend patient ID |
| `game_name` | string | ✅ | Name of the game played |
| `session_start` | ISO 8601 datetime | ❌ | UTC timestamp of session start |
| `component` 🆕 | string | ❌ | `Strength` / `Speed` / `Endurance` / `Coordination` |
| `grip_type` 🆕 | string | ❌ | `Cylindrical` / `Lumbrical` / `Prehension` / `Precision` / `Hook` / `Lateral` / `Tip` |
| `difficulty` | string | ❌ | `Easy` / `Medium` / `Hard` |
| `hand` | string | ❌ | `Left` / `Right` |
| `number_of_repetitions` 🆕 | integer | ❌ | |
| `number_of_grips` 🆕 | integer | ❌ | |
| `left_max_grip_strength` 🆕 | float | ❌ | kg |
| `right_max_grip_strength` 🆕 | float | ❌ | kg |
| `next_assessment_date` 🆕 | date string | ❌ | Format: `YYYY-MM-DD` |

**Expected Response `201`:**
```json
{
  "session_id": 42,
  "session": {
    "id": 42,
    "status": "active",
    "patient_uid": "uuid-xxxx",
    "game_name": "Rocket Ascent",
    "session_start": "2026-07-03T08:00:00Z"
  }
}
```

> Frontend reads `session_id` OR `session.id` OR `id` from this response.

---

### 5.2 Upload Sensor Window Batch

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `POST /sessions/{sessionId}/windows` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.uploadWindowBatch(sessionId, windows)` → called from `SyncService.syncAll()` and live BLE data flush |

**Path Parameter:**
| Parameter | Type | Required |
|-----------|------|:--------:|
| `sessionId` | integer | ✅ |

**Request Body (JSON):**
```json
{
  "windows": [
    {
      "window_index": 0,
      "timestamp": "2026-07-03T08:00:01Z",
      "s1_avg": 22.0,
      "s2_avg": 21.5,
      "s3_avg": 20.8,
      "s4_avg": 19.2,
      "sensor_avg": 20.875
    },
    {
      "window_index": 1,
      "timestamp": "2026-07-03T08:00:02Z",
      "s1_avg": 18.0,
      "s2_avg": 17.5,
      "s3_avg": 16.8,
      "s4_avg": 15.2,
      "sensor_avg": 16.875
    }
  ]
}
```

| Parameter | Type | Required | Notes |
|-----------|------|:--------:|-------|
| `windows` | array | ✅ | Array of window objects |
| `windows[].window_index` | integer | ❌ | Sequential index |
| `windows[].timestamp` | ISO 8601 | ❌ | UTC timestamp |
| `windows[].s1_avg` | float | ✅ | Sensor 1 average reading |
| `windows[].s2_avg` | float | ✅ | Sensor 2 average reading |
| `windows[].s3_avg` | float | ✅ | Sensor 3 average reading |
| `windows[].s4_avg` | float | ✅ | Sensor 4 average reading |
| `windows[].sensor_avg` | float | ❌ | Average across all 4 sensors |

**Expected Response `200`:**
```json
{ "success": true, "windows_saved": 2 }
```

---

### 5.3 End Session

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `POST /sessions/{sessionId}/end` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.endSession(sessionId, ...)` → called from `SyncService.syncAll()` and live session completion |

**Path Parameter:**
| Parameter | Type | Required |
|-----------|------|:--------:|
| `sessionId` | integer | ✅ |

**Request Body (JSON):**
```json
{
  "session_end":      "2026-07-03T08:01:00Z",
  "duration":         60,
  "duration_seconds": 60,

  "attempt1":              21.5,
  "attempt2":              22.0,
  "attempt3":              20.8,
  "attempts_average":      21.43,
  "latest_grip_strength":  22.0,

  "score":    1240,
  "status":   "win",
  "peak_avg": 22.0,
  "peak_s1":  22.0,
  "peak_s2":  21.5,
  "peak_s3":  20.8,
  "peak_s4":  19.2,
  "avg_force": 20.1
}
```

| Parameter | Type | Required | Notes |
|-----------|------|:--------:|-------|
| `session_end` | ISO 8601 datetime | ❌ | UTC end timestamp |
| `duration` | integer | ❌ | Seconds (backend subtracts 5s warmup automatically) |
| `duration_seconds` | integer | ❌ | Alias of `duration` |
| `attempt1` 🆕 | float | ❌ | Grip strength reading attempt 1 (kg) |
| `attempt2` 🆕 | float | ❌ | Grip strength reading attempt 2 (kg) |
| `attempt3` 🆕 | float | ❌ | Grip strength reading attempt 3 (kg) |
| `attempts_average` 🆕 | float | ❌ | Average of non-zero attempts |
| `latest_grip_strength` 🆕 | float | ❌ | Maximum of the 3 attempts |
| `score` | integer | ❌ | Game score |
| `status` | string | ❌ | `win` / `defeat` / `complete` |
| `peak_avg` | float | ❌ | Peak average sensor reading |
| `peak_s1` | float | ❌ | Peak sensor 1 |
| `peak_s2` | float | ❌ | Peak sensor 2 |
| `peak_s3` | float | ❌ | Peak sensor 3 |
| `peak_s4` | float | ❌ | Peak sensor 4 |
| `avg_force` | float | ❌ | Overall average force across session |

**Expected Response `200`:**
```json
{
  "session": {
    "id": 42,
    "status": "complete",
    "duration": 55,
    "score": 1240,
    "peak_avg": 22.0
  }
}
```

---

## GROUP 6 — Clinical Tree

---

### 6.1 Get Doctor's Full Data Tree

| Field | Value |
|-------|-------|
| **Backend Endpoint** | `GET /doctor/tree` |
| **Auth** | 🔐 Required |
| **Frontend Method** | `LaravelApiService.getClinicalTree()` → called from `SyncService.pullRemoteData()` → triggered on sync |

**Expected Response `200`:**
```json
{
  "doctors": [
    {
      "id": "uuid-xxxx",
      "name": "Dr. Raj Patel",
      "email": "raj@clinic.com",
      "phone": "+919876543210",
      "clinic": "City Rehabilitation Center",
      "discipline": "Physiotherapy",
      "patients": [
        {
          "uid": "uuid-yyyy",
          "name": "John Doe",
          "gender": "Male",
          "dob": "2010-05-15",
          "sessions": []
        }
      ]
    }
  ]
}
```

---

## Master Summary Table — All 19 Endpoints

| # | Backend Endpoint | Method | Frontend Method | Auth |
|---|-----------------|--------|-----------------|:----:|
| 1.1 | `/auth/register` | POST | `LaravelApiService.register()` | 🟢 |
| 1.2 | `/auth/login` | POST | `LaravelApiService.login()` | 🟢 |
| 1.3 | `/auth/logout` | POST | `LaravelApiService.logout()` | 🔐 |
| 2.1 | `/users/me` | GET | `LaravelApiService.getMe()` | 🔐 |
| 2.2 | `/users/me` | PUT | `LaravelApiService.updateMe()` | 🔐 |
| 2.3 | `/users/me` | DELETE | `AuthProvider.deleteAccount()` | 🔐 |
| 3.1 | `/patients` | GET | `LaravelApiService.getPatients()` | 🔐 |
| 3.2 | `/patients` | POST | `LaravelApiService.addPatient()` | 🔐 |
| 3.3 | `/patients/{uid}` | GET | `LaravelApiService.getPatient()` | 🔐 |
| 3.4 | `/patients/{uid}` | PUT | `LaravelApiService.updatePatient()` | 🔐 |
| 3.5 | `/patients/{uid}` | DELETE | `LaravelApiService.deletePatient()` | 🔐 |
| 3.6 | `/patients/{uid}/sessions` | GET | `LaravelApiService.getPatientSessions()` | 🔐 |
| 3.7 | `/patients/{uid}/sessions/{id}` | GET | `LaravelApiService.getPatientSession()` | 🔐 |
| 4.1 | `/devices` | GET | `LaravelApiService.getDevices()` | 🔐 |
| 4.2 | `/devices` | POST | `LaravelApiService.addDevice()` | 🔐 |
| 5.1 | `/sessions/start` | POST | `LaravelApiService.startSession()` | 🔐 |
| 5.2 | `/sessions/{id}/windows` | POST | `LaravelApiService.uploadWindowBatch()` | 🔐 |
| 5.3 | `/sessions/{id}/end` | POST | `LaravelApiService.endSession()` | 🔐 |
| 6.1 | `/doctor/tree` | GET | `LaravelApiService.getClinicalTree()` | 🔐 |

---

## New Database Columns Required (This Release 🆕)

### `users` / `doctors` table
| Column | Type | Nullable |
|--------|------|:--------:|
| `clinic` | VARCHAR(255) | ✅ |
| `discipline` | VARCHAR(100) | ✅ |

### `patients` table
| Column | Type | Nullable |
|--------|------|:--------:|
| `is_child_clinical` | BOOLEAN | ❌ (default false) |
| `medical_diagnosis` | TEXT | ✅ |
| `date_of_diagnosis` | DATE | ✅ |
| `duration_of_condition` | VARCHAR(100) | ✅ |
| `affected_side` | ENUM('Right','Left','Bilateral') | ✅ |
| `has_previous_surgery` | BOOLEAN | ❌ (default false) |
| `surgery_details` | TEXT | ✅ |
| `has_previous_fracture` | BOOLEAN | ❌ (default false) |
| `medication_history` | TEXT | ✅ |

### `sessions` table
| Column | Type | Nullable |
|--------|------|:--------:|
| `component` | ENUM('Strength','Speed','Endurance','Coordination') | ✅ |
| `grip_type` | ENUM('Cylindrical','Lumbrical','Prehension','Precision','Hook','Lateral','Tip') | ✅ |
| `number_of_repetitions` | INTEGER | ✅ |
| `number_of_grips` | INTEGER | ✅ |
| `left_max_grip_strength` | DECIMAL(6,2) | ✅ |
| `right_max_grip_strength` | DECIMAL(6,2) | ✅ |
| `next_assessment_date` | DATE | ✅ |
| `attempt1` | DECIMAL(6,2) | ✅ |
| `attempt2` | DECIMAL(6,2) | ✅ |
| `attempt3` | DECIMAL(6,2) | ✅ |
| `attempts_average` | DECIMAL(6,2) | ✅ |
| `latest_grip_strength` | DECIMAL(6,2) | ✅ |

---

## HTTP Status Code Conventions

| Code | Meaning | When |
|------|---------|------|
| `200` | OK | Successful GET, PUT, POST (end/logout) |
| `201` | Created | Successful POST (register, patient, device, session start) |
| `204` | No Content | Successful DELETE |
| `401` | Unauthorized | Missing or invalid token |
| `422` | Unprocessable Entity | Validation failure (e.g. email taken) |
| `404` | Not Found | Resource doesn't exist |
| `500` | Server Error | Unexpected backend failure |

## Standard Error Response Format
```json
{
  "error": {
    "message": "The email has already been taken.",
    "code": 422
  }
}
```
