# 🏥 WhatsApp Appointment Booking Agent

An AI-powered WhatsApp appointment booking system built with **n8n**, **OpenRouter**, **Google Sheets**, and **Google Calendar**.

Patients can book, view, reschedule, and cancel appointments entirely through WhatsApp without requiring a receptionist for routine scheduling. The workflow combines conversational AI with deterministic automation to ensure reliable bookings, prevent double bookings, and keep appointment data synchronized.


---

## Features

- AI-powered conversational booking assistant
- Book new appointments through WhatsApp
- View upcoming appointments
- Reschedule existing appointments
- Cancel appointments
- Automatic patient lookup and registration
- Real-time slot availability
- Session-based conversation memory
- Google Sheets as the backend database
- Automatic Google Calendar synchronization
- Daily appointment reminders
- Daily doctor's schedule summary
- Double-booking prevention

---

## Production Design Decisions

This project was designed to make appointment scheduling reliable rather than simply conversational.

### AI never writes to the database

The AI agent is responsible only for understanding user requests and guiding the conversation.

Once the patient confirms an action, the AI returns a structured handoff response.

The n8n workflow parses that response and performs all database operations.

This approach prevents:

- Hallucinated bookings
- Invalid database writes
- Inconsistent appointment records

---

### Slot availability is verified twice

Available slots are shown during the conversation.

Before creating or updating an appointment, the workflow checks the selected slot again to ensure another patient hasn't booked it in the meantime.

This prevents race conditions and double bookings.

---

### Appointment data is always revalidated

Reschedule, cancel, and appointment lookup requests always retrieve fresh appointment data before any action is performed.

No appointment information is trusted from previous conversation memory.

---

### Session memory expires automatically

Each WhatsApp conversation has its own session.

If the patient is inactive for more than 30 minutes, a new session is created so every new conversation starts with a clean context.

---

### Timezone-safe scheduling

All dates and times are generated using the **Asia/Karachi** timezone to avoid incorrect dates caused by UTC conversions.

---

### Safe Google Sheets handling

Google Sheets may return empty placeholder rows when no matching records exist.

The workflow explicitly filters these rows before making decisions to avoid false matches.

---

## Workflow Architecture

```text
                   WhatsApp Message
                          │
                          ▼
                 WhatsApp Trigger
                          │
                          ▼
          Message & Date Preprocessing
                          │
                          ▼
               Session Management
                          │
                          ▼
             AI Agent (OpenRouter)
                          │
      ┌───────────────────┼───────────────────┐
      │                   │                   │
      ▼                   ▼                   ▼
 Get Patient      Check Available Slots   Get Appointments
      │                   │                   │
      └───────────────────┼───────────────────┘
                          ▼
          Parse Structured AI Response
                          │
                          ▼
         Deterministic Workflow Logic
                          │
      ┌───────────────────┼───────────────────┐
      ▼                   ▼                   ▼
 Google Sheets     Google Calendar     WhatsApp Reply
```

---

## Background Automations

### Daily Patient Reminder

Every morning the workflow:

- Finds tomorrow's confirmed appointments
- Sends reminder messages through WhatsApp
- Marks reminders as sent

### Daily Doctor Schedule

Every morning the workflow:

- Collects today's confirmed appointments
- Creates a schedule summary
- Sends it to the clinic's WhatsApp

---

## Technologies Used

- **n8n** – Workflow automation
- **OpenRouter** – AI model provider
- **GPT-OSS-120B** – Large Language Model
- **WhatsApp Business API** – Messaging platform
- **Google Sheets** – Database
- **Google Calendar** – Calendar synchronization
- **JavaScript** – Custom logic using n8n Code nodes

---

## Google Sheets Structure

### Patients

- `patient_id`
- `whatsapp_number`
- `Name`
- `Age`
- `Gender`
- `Booker_Name`

### Sessions

- `whatsapp_number`
- `session_id`
- `last_active_at`

### Appointments

- `appointment_id`
- `patient_id`
- `whatsapp_number`
- `date`
- `Time`
- `Status`
- `calendar_event_id`
- `reminder_sent`

---

## Repository Structure

```text
.
├── Doctor Clinic WhatsApp Booking System.json
├── README.md
└── docs/
    ├── workflow.png
    ├── architecture.png
    ├── whatsapp-demo.gif
    └── chatbot-demo.png
```

---

## Setup

1. Import the workflow into **n8n**.
2. Configure the required credentials:
   - WhatsApp Business API
   - OpenRouter
   - Google Sheets
   - Google Calendar
3. Create a Google Spreadsheet containing the following sheets:
   - Patients
   - Sessions
   - Appointments
4. Update the spreadsheet IDs, calendar ID, and WhatsApp configuration inside the workflow.
5. Activate the workflow.

> **Note:** The exported workflow contains only credential references. Replace them with your own credentials before deploying.

---

## Future Improvements

- Multi-doctor support
- Multi-clinic support
- Online payment integration
- SMS and email notifications
- Holiday and leave management
- Analytics dashboard
- Patient portal
- Multi-language support

---