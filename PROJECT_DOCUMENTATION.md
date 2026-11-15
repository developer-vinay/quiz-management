# 🎓 Quiz Management System - Complete Project Documentation

## 📋 Table of Contents
1. [Executive Summary](#executive-summary)
2. [System Architecture](#system-architecture)
3. [Technology Stack](#technology-stack)
4. [User Roles & Permissions](#user-roles--permissions)
5. [Database Models](#database-models)
6. [API Endpoints](#api-endpoints)
7. [Key Features](#key-features)
8. [Authentication & Security](#authentication--security)
9. [Quiz Creation Methods](#quiz-creation-methods)
10. [Event Quiz System](#event-quiz-system)
11. [Email Notification System](#email-notification-system)
12. [Frontend Components](#frontend-components)
13. [Configuration & Setup](#configuration--setup)
14. [Data Flow Diagrams](#data-flow-diagrams)

---

## 🎯 Executive Summary

The **Quiz Management System** is a comprehensive full-stack MERN (MongoDB, Express.js, React.js, Node.js) web application designed for educational institutions to manage academic assessments and cross-college quiz competitions.

### Key Highlights
- **Multi-role system**: Admin, Faculty, Student, Event Manager
- **Flexible quiz creation**: 4 different input methods
- **Advanced security**: Anti-cheating measures with admin override
- **Event management**: Cross-college competitions with team support
- **Real-time analytics**: Comprehensive reporting and performance tracking
- **Email notifications**: Automated credential delivery and quiz alerts
- **Responsive design**: Mobile-friendly interface

---

## 🏗️ System Architecture

### Three-Tier Architecture

#### 1. **Frontend Layer (React + Vite)**
- Single Page Application (SPA)
- Material-UI (MUI) component library
- Context API for state management
- Role-based routing and navigation
- Responsive design for all devices

#### 2. **Backend Layer (Node.js + Express)**
- RESTful API architecture
- JWT-based authentication
- Role-based access control middleware
- File processing (Excel, Word, Image OCR)
- Email service integration
- Error handling and logging

#### 3. **Database Layer (MongoDB)**
- NoSQL document-based storage
- Mongoose ODM for schema validation
- Indexed queries for performance
- Data relationships and population
- Soft delete functionality

---

## 💻 Technology Stack

### Backend Dependencies
```json
{
  "express": "^4.21.2",
  "mongoose": "^7.8.7",
  "bcryptjs": "^2.4.3",
  "jsonwebtoken": "^9.0.0",
  "nodemailer": "^7.0.3",
  "multer": "^1.4.5-lts.1",
  "xlsx": "^0.18.5",
  "mammoth": "^1.9.1",
  "tesseract.js": "^6.0.1",
  "sharp": "^0.34.1",
  "cors": "^2.8.5",
  "dotenv": "^16.5.0"
}
```

### Frontend Dependencies
```json
{
  "react": "^19.1.0",
  "react-router-dom": "^7.6.0",
  "@mui/material": "^7.1.0",
  "@mui/icons-material": "^7.1.0",
  "axios": "^1.9.0",
  "react-toastify": "^11.0.5",
  "recharts": "^2.15.3",
  "jspdf": "^3.0.1",
  "dayjs": "^1.11.13",
  "papaparse": "^5.5.3",
  "tesseract.js": "^4.1.1"
}
```

### Development Environment
- **Node.js**: v14+ recommended
- **MongoDB**: Atlas or local instance
- **Port**: Backend (5000), Frontend (5173)
- **Package Manager**: npm or yarn

---

## 👥 User Roles & Permissions

### 1. **Admin (System Administrator)**

#### Capabilities
- ✅ Complete system control and configuration
- ✅ User management (create, edit, delete all users)
- ✅ College settings and branding
- ✅ Department and subject management
- ✅ Faculty subject assignments
- ✅ Quiz security settings and admin override
- ✅ View all quizzes and submissions
- ✅ Comprehensive analytics and reporting

#### Access Areas
- Admin Dashboard
- Account Management (Students, Faculty, Event Accounts)
- College Settings
- Quiz Management
- Result Analysis
- System Configuration

### 2. **Faculty**

#### Capabilities
- ✅ Create quizzes (4 methods: Manual, Excel, Word, Image OCR)
- ✅ Edit and delete own quizzes
- ✅ View submissions and grade responses
- ✅ Subject-specific access control
- ✅ Department-wise quiz creation
- ✅ Email notifications to eligible students
- ✅ Performance analytics for own quizzes
- ✅ Export results to PDF

#### Restrictions
- ❌ Cannot create quizzes for unassigned departments
- ❌ Cannot modify other faculty quizzes
- ❌ Cannot access admin settings

### 3. **Student**

#### Capabilities
- ✅ View available quizzes for their class
- ✅ Take quizzes within time window
- ✅ Review submitted quizzes and scores
- ✅ Track performance history
- ✅ Update profile information
- ✅ Register for event quizzes
- ✅ View results and analytics

#### Restrictions
- ❌ Cannot access quizzes for other departments/sections
- ❌ Cannot retake submitted quizzes
- ❌ Cannot view other students' submissions

### 4. **Event Manager**

#### Capabilities
- ✅ Create event quizzes for competitions
- ✅ Manage participant registrations
- ✅ Generate credentials for participants
- ✅ Send bulk emails to participants
- ✅ View event quiz submissions
- ✅ Create follow-up quizzes for top performers
- ✅ Support team-based participation

#### Restrictions
- ❌ Cannot access academic quizzes
- ❌ Cannot modify admin settings

---

## 🗄️ Database Models

### 1. **User Model** (`models/User.js`)

**Collection**: `users`

**Purpose**: Multi-role user management with conditional fields

#### Common Fields (All Roles)
```javascript
{
  _id: ObjectId,                    // Auto-generated unique identifier
  name: String (required),          // Full name of user
  email: String (required, unique), // Email address (lowercase)
  password: String (required),      // Bcrypt hashed password
  originalPassword: String,         // Plain text for admin recovery
  role: String (enum: ['admin', 'faculty', 'student', 'event']),
  createdAt: Date,                  // Auto-generated
  updatedAt: Date                   // Auto-generated
}
```

#### Student-Specific Fields (role = 'student')
```javascript
{
  department: String (required),
  year: Number (required, 1-4),
  semester: Number (required, 1-8),
  section: String (required),
  admissionNumber: String (required, unique),
  isLateral: Boolean (default: false)
}
```

#### Faculty/Event-Specific Fields (role = 'faculty' or 'event')
```javascript
{
  departments: [String] (required),
  years: [String] (required),
  semesters: [String] (required),
  sections: [String] (required),
  assignments: [{
    department: String,
    year: String,
    semester: String,
    sections: [String],
    subjects: [String]
  }]
}
```

#### Password Reset Fields
```javascript
{
  resetPasswordCode: String,        // 6-digit verification code
  resetPasswordExpiry: Date         // 10-minute expiration
}
```

#### Indexes
- `email`: Unique index
- `admissionNumber`: Unique sparse index
- `role, department`: Compound index

---

### 2. **Quiz Model** (`models/Quiz.js`)

**Collection**: `quizzes`

**Purpose**: Academic quiz management

#### Core Fields
```javascript
{
  _id: ObjectId,
  title: String (required, unique),
  description: String,
  type: String (enum: ['academic', 'event']),
  createdBy: ObjectId (ref: 'User'),
  duration: Number (required, in minutes),
  totalMarks: Number (required, auto-calculated),
  startTime: Date (required),
  endTime: Date (required),
  isActive: Boolean (default: true),
  createdAt: Date,
  updatedAt: Date
}
```

#### Academic Fields (type = 'academic')
```javascript
{
  subject: {
    code: String (required),
    name: String (required)
  },
  allowedGroups: [{
    department: String,
    year: Number,
    semester: Number,
    section: String,
    endTime: Date (optional, section-specific)
  }]
}
```

#### Question Schema
```javascript
{
  questions: [{
    question: String (required, HTML formatted),
    options: [String] (required, exactly 4),
    correctAnswer: Number (required, 0-3),
    marks: Number (required, min: 1),
    negativeMarks: Number (default: 0),
    isCodeQuestion: Boolean (default: false)
  }]
}
```

#### Advanced Features
```javascript
{
  negativeMarkingEnabled: Boolean (default: false),
  shuffleQuestions: Boolean (default: false),
  questionLimitEnabled: Boolean (default: false),
  questionLimit: Number (min: 1),
  randomizeQuestionsPerStudent: Boolean (default: false),
  sendEmailNotification: Boolean (default: false)
}
```

#### Methods
- `getShuffledQuestions(userId)`: Returns deterministic shuffled questions per student
- `getEffectiveTotalMarks()`: Calculates marks when question limiting is enabled

---

### 3. **EventQuiz Model** (`models/EventQuiz.js`)

**Collection**: `eventquizzes`

**Purpose**: Cross-college quiz competitions

#### Core Fields
```javascript
{
  _id: ObjectId,
  title: String (required, unique),
  description: String,
  duration: Number (required, minutes),
  startTime: Date (required),
  endTime: Date (required),
  totalMarks: Number (auto-calculated),
  createdAt: Date,
  updatedAt: Date
}
```

#### Participant Management
```javascript
{
  participantTypes: [String] (enum: ['college', 'external']),
  registrationEnabled: Boolean (default: true),
  spotRegistrationEnabled: Boolean (default: false),
  maxParticipants: Number (default: 0, unlimited),
  participationMode: String (enum: ['individual', 'team']),
  teamSize: Number (min: 1, max: 10)
}
```

#### Registration Array
```javascript
{
  registrations: [{
    name: String,
    email: String,
    college: String,
    department: String,
    year: String,
    phoneNumber: String,
    admissionNumber: String,
    participantType: String (enum: ['college', 'external']),
    isTeamRegistration: Boolean,
    teamName: String,
    teamMembers: [Object],
    registeredAt: Date,
    isDeleted: Boolean (default: false)
  }]
}
```

#### Follow-up Quiz Support
```javascript
{
  isFollowUp: Boolean (default: false),
  sourceQuiz: ObjectId (ref: 'EventQuiz')
}
```

---

### 4. **QuizSubmission Model** (`models/QuizSubmission.js`)

**Collection**: `quizsubmissions`

**Purpose**: Track student quiz attempts

```javascript
{
  _id: ObjectId,
  quiz: ObjectId (ref: 'Quiz'),
  student: ObjectId (ref: 'User'),
  answers: [{
    questionId: ObjectId,
    selectedAnswer: Number,
    isCorrect: Boolean,
    marks: Number,
    negativeMarks: Number
  }],
  score: Number,
  totalMarks: Number,
  status: String (enum: ['in-progress', 'submitted', 'evaluated']),
  startTime: Date,
  submitTime: Date,
  timeTaken: Number (in seconds),
  isDeleted: Boolean (default: false),
  violationCount: Number (default: 0),
  violations: [String],
  createdAt: Date,
  updatedAt: Date
}
```

---

### 5. **EventQuizResult Model** (`models/EventQuizResult.js`)

**Collection**: `eventquizresults`

**Purpose**: Store event quiz submissions with participant details

```javascript
{
  _id: ObjectId,
  quiz: ObjectId (ref: 'EventQuiz'),
  participantName: String,
  participantEmail: String,
  college: String,
  department: String,
  year: String,
  phoneNumber: String,
  admissionNumber: String,
  participantType: String,
  isTeamSubmission: Boolean,
  teamName: String,
  teamMembers: [Object],
  answers: [Object],
  score: Number,
  totalMarks: Number,
  status: String,
  startTime: Date,
  submitTime: Date,
  timeTaken: Number,
  violationCount: Number,
  violations: [String],
  isDeleted: Boolean (default: false)
}
```

---

### 6. **QuizCredentials Model** (`models/QuizCredentials.js`)

**Collection**: `quizcredentials`

**Purpose**: Store event participant login credentials

```javascript
{
  _id: ObjectId,
  quiz: ObjectId (ref: 'EventQuiz'),
  registration: ObjectId,
  username: String (unique per quiz),
  password: String (hashed),
  originalPassword: String (plain text),
  participantDetails: Object,
  isTeam: Boolean,
  teamName: String,
  teamMembers: [Object],
  isActive: Boolean (default: true),
  lastLogin: Date,
  createdAt: Date
}
```

---

### 7. **QuizSettings Model** (`models/QuizSettings.js`)

**Collection**: `quizsettings`

**Purpose**: Global quiz security configuration

```javascript
{
  _id: ObjectId,
  adminOverride: {
    enabled: Boolean (default: false),
    password: String (default: 'admin123'),
    triggerButtons: {
      button1: String (default: 'Ctrl'),
      button2: String (default: '6')
    }
  },
  violationSettings: {
    maxViolations: Number (default: 3),
    autoTerminate: Boolean (default: true),
    strictMode: Boolean (default: true)
  }
}
```

---

### 8. **AcademicDetail Model** (`models/AcademicDetail.js`)

**Collection**: `academicdetails`

**Purpose**: Store institution academic structure

```javascript
{
  _id: ObjectId,
  colleges: [String],
  departments: [String],
  years: [Number],
  semesters: [Number],
  sections: [String],
  subjects: [{
    code: String,
    name: String,
    department: String,
    semester: Number
  }]
}
```

---

### 9. **College Model** (`models/College.js`)

**Collection**: `colleges`

**Purpose**: Store institution branding and configuration

```javascript
{
  _id: ObjectId,
  name: String (required),
  address: String,
  logo: String,
  primaryColor: String,
  secondaryColor: String,
  contactEmail: String,
  contactPhone: String,
  website: String
}
```

---

### 10. **Department Model** (`models/Department.js`)

**Collection**: `departments`

**Purpose**: Department master data

```javascript
{
  _id: ObjectId,
  code: String (required, unique),
  name: String (required),
  isActive: Boolean (default: true)
}
```

---

## 🌐 API Endpoints

### Authentication Routes (`/api/auth`)

#### POST /api/auth/register
- **Purpose**: Register a new user
- **Access**: Public
- **Body**: `{ name, email, password, role, department, year, section, admissionNumber }`
- **Response**: `{ token, user }`

#### POST /api/auth/login
- **Purpose**: User login
- **Access**: Public
- **Body**: `{ email, password }`
- **Response**: `{ success, message, token, user }`

#### GET /api/auth/me
- **Purpose**: Get current user details
- **Access**: Protected (JWT required)
- **Response**: `{ user }`

#### PUT /api/auth/profile
- **Purpose**: Update user profile
- **Access**: Protected
- **Body**: `{ name, email, department }`
- **Response**: `{ user }`

#### PUT /api/auth/change-password
- **Purpose**: Change user password
- **Access**: Protected
- **Body**: `{ currentPassword, newPassword }`
- **Response**: `{ message }`

#### POST /api/auth/forgot-password
- **Purpose**: Send password reset code
- **Access**: Public
- **Body**: `{ email }`
- **Response**: `{ success, message }`

#### POST /api/auth/verify-reset-code
- **Purpose**: Verify password reset code
- **Access**: Public
- **Body**: `{ email, code }`
- **Response**: `{ success, message }`

#### POST /api/auth/reset-password
- **Purpose**: Reset password with code
- **Access**: Public
- **Body**: `{ email, code, newPassword }`
- **Response**: `{ success, message }`

---

### Quiz Routes (`/api/quiz`)

#### POST /api/quiz
- **Purpose**: Create a new academic quiz
- **Access**: Faculty, Admin
- **Body**: Quiz object with questions
- **Response**: `{ success, message, quiz, emailNotification }`

#### GET /api/quiz
- **Purpose**: Get quizzes (filtered by role)
- **Access**: Protected
- **Query**: `?department=&year=&section=&subject=`
- **Response**: `[Quiz]`

#### GET /api/quiz/statistics
- **Purpose**: Get quiz statistics
- **Access**: Faculty, Admin
- **Query**: `?department=&year=&section=&subject=`
- **Response**: Statistics object

#### GET /api/quiz/my-submissions
- **Purpose**: Get student's submissions
- **Access**: Student
- **Response**: `[QuizSubmission]`

#### GET /api/quiz/check-title/:title
- **Purpose**: Check if quiz title is unique
- **Access**: Public
- **Response**: `{ isUnique, exists, message }`

#### GET /api/quiz/:id
- **Purpose**: Get specific quiz details
- **Access**: Protected
- **Response**: Quiz object (shuffled if configured)

#### PUT /api/quiz/:id
- **Purpose**: Update quiz
- **Access**: Faculty (owner), Admin
- **Body**: Updated quiz fields
- **Response**: `{ message, quiz }`

#### DELETE /api/quiz/:id
- **Purpose**: Delete quiz
- **Access**: Faculty (owner), Admin
- **Response**: `{ message }`

#### POST /api/quiz/:id/start
- **Purpose**: Start quiz attempt
- **Access**: Student
- **Response**: `{ message, submission }`

#### POST /api/quiz/:id/submit
- **Purpose**: Submit quiz answers
- **Access**: Student
- **Body**: `{ answers, timeTaken }`
- **Response**: `{ message, submission, score }`

#### GET /api/quiz/:id/submissions
- **Purpose**: Get all submissions for a quiz
- **Access**: Faculty (owner), Admin
- **Response**: `[QuizSubmission]`

---

### Event Quiz Routes (`/api/event-quiz`)

#### POST /api/event-quiz
- **Purpose**: Create event quiz
- **Access**: Faculty, Admin, Event Manager
- **Body**: Event quiz object
- **Response**: Event quiz object

#### GET /api/event-quiz
- **Purpose**: Get event quizzes
- **Access**: Protected
- **Query**: `?status=&participantType=&search=`
- **Response**: `[EventQuiz]`

#### GET /api/event-quiz/public
- **Purpose**: Get public event quizzes
- **Access**: Public
- **Response**: `[EventQuiz]` (without answers)

#### GET /api/event-quiz/:id
- **Purpose**: Get specific event quiz
- **Access**: Public
- **Response**: Event quiz object

#### PUT /api/event-quiz/:id
- **Purpose**: Update event quiz
- **Access**: Faculty, Admin, Event Manager
- **Body**: Updated quiz fields
- **Response**: Updated quiz object

#### DELETE /api/event-quiz/:id
- **Purpose**: Delete event quiz
- **Access**: Faculty, Admin, Event Manager
- **Response**: `{ message }`

#### POST /api/event-quiz/:id/register
- **Purpose**: Register for event quiz
- **Access**: Public
- **Body**: Registration details
- **Response**: `{ message, credentials }`

#### GET /api/event-quiz/:id/registrations
- **Purpose**: Get quiz registrations
- **Access**: Faculty, Admin, Event Manager
- **Response**: `[Registration]`

#### POST /api/event-quiz/:id/generate-credentials
- **Purpose**: Generate credentials for registrations
- **Access**: Faculty, Admin, Event Manager
- **Response**: `{ message, success, failed, credentials }`

#### POST /api/event-quiz/:id/send-bulk-email
- **Purpose**: Send bulk email to participants
- **Access**: Faculty, Admin, Event Manager
- **Body**: `{ subject, message, recipientEmails }`
- **Response**: `{ success, successCount, failureCount }`

#### POST /api/event-quiz/login
- **Purpose**: Login to event quiz
- **Access**: Public
- **Body**: `{ username, password, quizId }`
- **Response**: `{ message, sessionToken, quiz, participant }`

#### POST /api/event-quiz/:id/submit-authenticated
- **Purpose**: Submit event quiz
- **Access**: Protected (session token)
- **Body**: `{ answers, timeTaken, violationCount }`
- **Response**: `{ message, result, score }`

---

### Admin Routes (`/api/admin`)

#### POST /api/admin/students
- **Purpose**: Create student account
- **Access**: Admin
- **Body**: Student details
- **Response**: Created student object

#### POST /api/admin/students/bulk
- **Purpose**: Bulk upload students via Excel
- **Access**: Admin
- **Body**: FormData with Excel file
- **Response**: `{ success, total, created, errors }`

#### GET /api/admin/students
- **Purpose**: Get all students
- **Access**: Admin
- **Query**: `?department=&year=&section=`
- **Response**: `[Student]`

#### PUT /api/admin/students/:id
- **Purpose**: Update student
- **Access**: Admin
- **Body**: Updated student details
- **Response**: Updated student object

#### DELETE /api/admin/students/:id
- **Purpose**: Delete student
- **Access**: Admin
- **Response**: `{ message }`

#### POST /api/admin/faculty
- **Purpose**: Create faculty account
- **Access**: Admin
- **Body**: Faculty details with assignments
- **Response**: Created faculty object

#### POST /api/admin/faculty/bulk-assign
- **Purpose**: Bulk assign subjects via Excel
- **Access**: Admin
- **Body**: FormData with Excel file
- **Response**: `{ success, message, details }`

#### GET /api/admin/college-settings
- **Purpose**: Get college configuration
- **Access**: Admin
- **Response**: College settings object

#### PUT /api/admin/college-settings
- **Purpose**: Update college settings
- **Access**: Admin
- **Body**: Updated settings
- **Response**: Updated settings object

#### GET /api/admin/quiz-settings
- **Purpose**: Get quiz security settings
- **Access**: Admin
- **Response**: Quiz settings object

#### PUT /api/admin/quiz-settings
- **Purpose**: Update quiz security settings
- **Access**: Admin
- **Body**: Updated security settings
- **Response**: Updated settings object

---

### Academic Details Routes (`/api/academic-details`)

#### GET /api/academic-details
- **Purpose**: Get academic structure
- **Access**: Protected
- **Response**: Academic details object

#### POST /api/academic-details/subjects
- **Purpose**: Add new subject
- **Access**: Admin
- **Body**: `{ code, name, department, semester }`
- **Response**: Updated academic details

#### DELETE /api/academic-details/subjects/:code
- **Purpose**: Delete subject
- **Access**: Admin
- **Response**: `{ message }`

---

## 🎯 Key Features

### 1. **Advanced Quiz Security**

#### Fullscreen Enforcement
- Entire browser window forced to fullscreen
- Exit fullscreen triggers violation warning
- Persistent fullscreen throughout quiz duration

#### Anti-Cheating Measures
- Right-click menu disabled
- Copy/paste functionality blocked
- Keyboard shortcuts disabled (Ctrl+C, Ctrl+V, etc.)
- Tab/window switch detection
- Violation counter with progressive warnings
- Auto-termination after max violations (configurable)

#### Admin Override System
- Emergency access with configurable key combination
- Default: Ctrl + 6
- Password-protected override
- Bypass security restrictions for troubleshooting

#### Violation Tracking
```javascript
{
  maxViolations: 3,           // Configurable limit
  autoTerminate: true,        // Auto-submit on exceeding
  strictMode: true,           // Enable all restrictions
  violations: [               // Logged violations
    "Tab switch detected at 10:30 AM",
    "Exit fullscreen at 10:35 AM"
  ]
}
```

---

### 2. **Quiz Creation Methods**

#### Method 1: Manual Entry
- Rich text editor for questions
- Support for HTML formatting
- Code question toggle for programming quizzes
- 4 options per question
- Individual marks and negative marks
- Real-time preview

#### Method 2: Excel Upload
**Expected Format:**
```
| Question | Option A | Option B | Option C | Option D | Correct Answer | Marks | Negative Marks |
|----------|----------|----------|----------|----------|----------------|-------|----------------|
| Q1 text  | Opt 1    | Opt 2    | Opt 3    | Opt 4    | A/B/C/D        | 1     | 0              |
```

**Features:**
- Bulk upload 100+ questions
- Automatic validation
- Error reporting with line numbers
- Preserves formatting and indentation

#### Method 3: Word Document Upload
**Expected Format:**
```
1. Question text here?
a) Option A
b) Option B
c) Option C
d) Option D
Answer: a
Marks: 1
```

**Features:**
- Parses DOCX files
- Extracts formatted text
- Preserves code blocks
- Automatic option detection

#### Method 4: Image OCR
**Process:**
1. Upload image (JPG, PNG, GIF)
2. Tesseract.js OCR processing
3. Text extraction and parsing
4. Manual review and editing
5. Add to quiz

**Supported:**
- Printed text
- Handwritten text (limited accuracy)
- Multiple questions per image
- Code snippets in images

---

### 3. **Question Randomization**

#### Deterministic Shuffling
```javascript
// Same student always gets same order
const seed = hashStudentId(studentId);
const shuffled = deterministicShuffle(questions, seed);
```

#### Question Limiting
- Configure number of questions per student
- Random selection from question pool
- Each student gets different subset
- Automatic marks calculation
- Effective total marks = (questionLimit / totalQuestions) * totalMarks

#### Configuration Options
```javascript
{
  shuffleQuestions: true,              // Enable shuffling
  questionLimitEnabled: true,          // Enable limiting
  questionLimit: 20,                   // Questions per student
  randomizeQuestionsPerStudent: true   // Different questions per student
}
```

---

### 4. **Code Question Support**

#### Auto-Detection
- Detects indentation (2+ spaces)
- Recognizes programming keywords
- Identifies function definitions
- Preserves formatting perfectly

#### HTML Formatting
```javascript
// Automatic wrapping in <pre> tags
<pre>
def hello_world():
    print("Hello, World!")
</pre>
```

#### Language Support
- Python, Java, C, C++, JavaScript
- HTML/CSS
- SQL queries
- Pseudocode
- Any indented content

---

### 5. **Negative Marking System**

#### Per-Question Configuration
```javascript
{
  marks: 4,              // Correct answer marks
  negativeMarks: 1,      // Incorrect answer penalty
  correctAnswer: 2       // Index 0-3
}
```

#### Score Calculation
```javascript
let totalScore = 0;
answers.forEach(answer => {
  if (answer.isCorrect) {
    totalScore += question.marks;
  } else if (answer.selectedAnswer !== null) {
    totalScore -= question.negativeMarks;
  }
  // Unattempted = 0 marks
});
```

---

### 6. **Section-Specific Settings**

#### Feature
- Different end times for different sections
- Same quiz, flexible timing
- Useful for staggered exams

#### Example Configuration
```javascript
allowedGroups: [
  {
    department: "CSE",
    year: 3,
    semester: 5,
    section: "A",
    endTime: "2024-12-20T10:00:00Z"
  },
  {
    department: "CSE",
    year: 3,
    semester: 5,
    section: "B",
    endTime: "2024-12-20T11:00:00Z"  // 1 hour later
  }
]
```

---

## 🎪 Event Quiz System

### 1. **Participant Types**

#### College Students
- Internal students from host institution
- Department/year/semester filtering
- Pre-registered in system
- Email notifications enabled

#### External Participants
- Students from other colleges
- Manual registration required
- Complete profile collection
- Team support available

### 2. **Registration System**

#### Individual Registration
```javascript
{
  name: "John Doe",
  email: "john@example.com",
  college: "ABC College",
  department: "Computer Science",
  year: "3",
  phoneNumber: "9876543210",
  admissionNumber: "20CS123",
  participantType: "external"
}
```

#### Team Registration (1-10 members)
```javascript
{
  isTeamRegistration: true,
  teamName: "Code Warriors",
  teamLeader: {
    name: "John Doe",
    email: "john@example.com",
    college: "ABC College",
    // ... other details
  },
  teamMembers: [
    {
      name: "Jane Smith",
      email: "jane@example.com",
      // ... other details
    }
    // ... more members
  ]
}
```

**Team Features:**
- Shared login credentials
- Only one member takes quiz
- All members receive email
- Team name displayed in results

### 3. **Credential Generation**

#### Username Pattern
```javascript
// Individual: Email address
username = email.toLowerCase()

// Team: Team leader's email
username = teamLeader.email.toLowerCase()
```

#### Password Pattern
```javascript
// With admission number
password = firstName + admissionNumber
// Example: "john20CS123"

// With phone number (no admission)
password = firstName + last4DigitsOfPhone
// Example: "john3210"

// External (no admission/phone)
password = firstName + deterministicHash
// Example: "john4857"
```

### 4. **Follow-up Quiz Creation**

#### Process
1. View submissions of source quiz
2. Select top performers (score-based)
3. Create new quiz with selected participants
4. System auto-registers participants
5. Credentials sent via email

#### Use Cases
- Second round for competitions
- Advanced level after qualifying round
- Selective skill assessment

### 5. **Spot Registration**

#### Feature
- On-site registration during event
- QR code scanning support
- Instant credential generation
- Immediate email delivery

#### Configuration
```javascript
{
  spotRegistrationEnabled: true,
  maxParticipants: 100  // 0 = unlimited
}
```

---

## 📧 Email Notification System

### 1. **Academic Quiz Notifications**

#### Trigger
- Quiz creation with `sendEmailNotification: true`
- Sent to all eligible students

#### Email Content
```
Subject: 📚 New Academic Quiz Available - [Quiz Title]

Content:
- Quiz details (title, subject, duration)
- Schedule (start time, end time)
- Academic criteria match
- Login instructions
- Important notes
```

#### Eligibility Filtering
```javascript
const eligibleStudents = await User.find({
  role: 'student',
  $or: quiz.allowedGroups.map(group => ({
    department: group.department,
    year: group.year,
    semester: group.semester,
    section: group.section
  }))
});
```

### 2. **Event Quiz Registration Emails**

#### Individual Registration Email
```
Subject: 🎉 Registration Confirmed - [Quiz Title]

Content:
- Confirmation message
- Quiz details
- Login credentials (username & password)
- Password explanation (based on admission/phone)
- Special instructions
- Important notes
```

#### Team Registration Email
```
Subject: 🎉 Welcome to [Quiz Title] - Team Registration Confirmed!

Content:
- Team confirmation
- Complete team details (all members)
- Shared login credentials
- Team coordination instructions
- Quiz details
- Important notes
```

**Sent to**: All team members (leader + members)

### 3. **Follow-up Quiz Invitation Email**

```
Subject: 🎯 Follow-up Quiz Invitation - [Quiz Title]

Content:
- Selection notification
- Source quiz reference
- New quiz details
- Use original password instruction
- Exclusive opportunity message
```

### 4. **Password Reset Email**

```
Subject: 🔐 Password Reset Code - Quiz Management System

Content:
- 6-digit verification code
- 10-minute expiration notice
- Security instructions
- No action required message (if not requested)
```

### 5. **Bulk Email Feature**

#### Use Cases
- Send reminders to participants
- Share results announcements
- Send additional instructions
- Communicate schedule changes

#### Process
```javascript
await sendBulkEmail({
  emails: ['participant1@email.com', 'participant2@email.com'],
  subject: 'Quiz Reminder',
  message: 'Custom message here',
  quizTitle: 'Event Quiz Title',
  senderName: 'Event Coordinator'
});
```

**Response:**
```javascript
{
  success: true,
  successCount: 45,
  failureCount: 2,
  failures: [
    { email: 'invalid@email.com', error: 'Invalid address' }
  ]
}
```

### 6. **Email Service Configuration**

#### Gmail SMTP Setup
```javascript
// .env configuration
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-app-password  // Not regular password!
```

#### App Password Generation
1. Enable 2FA on Gmail account
2. Go to Google Account > Security
3. Generate App Password for "Mail"
4. Use generated 16-character password

#### Nodemailer Transporter
```javascript
const transporter = nodemailer.createTransporter({
  service: 'gmail',
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS
  }
});
```

---

## 🎨 Frontend Components

### Admin Components (`/src/components/admin`)

1. **Dashboard.jsx** - Admin overview with statistics
2. **AccountManagement.jsx** - User account CRUD operations
3. **StudentAccounts.jsx** - Student-specific management
4. **FacultyAccounts.jsx** - Faculty management with assignments
5. **EventQuizAccounts.jsx** - Event manager accounts
6. **CollegeSettings.jsx** - Institution configuration
7. **QuizManagement.jsx** - Quiz oversight
8. **AdminQuizzes.jsx** - Quiz list with filters
9. **ResultAnalysis.jsx** - Performance analytics

### Quiz Components (`/src/components/quiz`)

1. **QuizCreate.jsx** - Create quiz with manual entry
2. **QuizFileUpload.jsx** - Upload Excel/Word/Image
3. **QuizEdit.jsx** - Edit existing quiz
4. **QuizList.jsx** - Display available quizzes
5. **QuizAttempt.jsx** - Take quiz interface
6. **QuizReview.jsx** - Review submitted quiz
7. **QuizResult.jsx** - Display quiz results
8. **QuizSubmissions.jsx** - View all submissions
9. **QuizSubmissionView.jsx** - Individual submission details
10. **QuizStatistics.jsx** - Quiz analytics
11. **QuizOverview.jsx** - Quiz summary
12. **QuizSecurity.jsx** - Security features component
13. **EventQuizzes.jsx** - Public event quiz list
14. **QuizLogin.jsx** - Event quiz login
15. **PublicQuizTake.jsx** - Take event quiz (public)
16. **AuthenticatedQuizTake.jsx** - Take event quiz (authenticated)

### Event Components (`/src/components/event`)

1. **EventPage.jsx** - Event manager dashboard
2. **EventQuizCreate.jsx** - Create event quiz
3. **EventQuizEdit.jsx** - Edit event quiz
4. **EventQuizList.jsx** - List event quizzes
5. **EventQuizCard.jsx** - Quiz card display
6. **EventQuizRegistration.jsx** - Registration form
7. **EventQuizRegistrations.jsx** - View registrations
8. **EventQuizSubmissions.jsx** - View submissions
9. **EventQuizSubmissionView.jsx** - Individual submission
10. **EventActiveQuizzes.jsx** - Active quizzes list
11. **EventUpcomingQuizzes.jsx** - Upcoming quizzes

### Auth Components (`/src/components/auth`)

1. **Login.jsx** - User login
2. **Register.jsx** - User registration
3. **ForgotPassword.jsx** - Password reset flow
4. **RoleBasedRedirect.jsx** - Redirect based on role

### Common Components (`/src/components/common`)

1. **Navigation.jsx** - Role-based navigation
2. **PrivateRoute.jsx** - Protected route wrapper
3. **LoadingScreen.jsx** - Loading indicator
4. **CollegeHeader.jsx** - Institution branding
5. **AcademicFilter.jsx** - Department/year/section filter
6. **ErrorBoundary.jsx** - Error handling

### Student Components (`/src/components/student`)

1. **ReviewQuizzes.jsx** - Review completed quizzes

### Subject Components (`/src/components/subject`)

1. **SubjectList.jsx** - Display subjects
2. **SubjectCreate.jsx** - Add new subject
3. **SubjectEdit.jsx** - Edit subject

### Profile Components (`/src/components/profile`)

1. **Profile.jsx** - View/edit user profile
2. **EditProfile.jsx** - Profile editing form

---

## 🔐 Authentication & Security

### 1. **JWT Authentication**

#### Token Generation
```javascript
const token = jwt.sign(
  { 
    userId: user._id,
    role: user.role,
    email: user.email
  },
  process.env.JWT_SECRET,
  { expiresIn: '24h' }
);
```

#### Token Validation
```javascript
const auth = async (req, res, next) => {
  const token = req.header('Authorization')?.replace('Bearer ', '');
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  const user = await User.findById(decoded.userId);
  req.user = user;
  next();
};
```

### 2. **Password Security**

#### Hashing
```javascript
const bcrypt = require('bcryptjs');
const salt = await bcrypt.genSalt(10);
const hashedPassword = await bcrypt.hash(password, salt);
```

#### Comparison
```javascript
const isMatch = await bcrypt.compare(password, user.password);
```

### 3. **Role-Based Access Control (RBAC)**

#### Middleware Implementation
```javascript
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        message: 'Access denied'
      });
    }
    next();
  };
};
```

#### Usage
```javascript
router.post('/quiz', 
  auth, 
  authorize('faculty', 'admin'), 
  createQuiz
);
```

### 4. **Password Reset Flow**

#### Step 1: Request Reset
```javascript
// Generate 6-digit code
const resetCode = Math.floor(100000 + Math.random() * 900000).toString();
const resetCodeExpiry = new Date(Date.now() + 10 * 60 * 1000); // 10 min

user.resetPasswordCode = resetCode;
user.resetPasswordExpiry = resetCodeExpiry;
await user.save();

// Send email with code
await sendForgotPasswordEmail(user.email, user.name, resetCode);
```

#### Step 2: Verify Code
```javascript
const user = await User.findOne({
  email: email.toLowerCase(),
  resetPasswordCode: code,
  resetPasswordExpiry: { $gt: new Date() }
});
```

#### Step 3: Reset Password
```javascript
const salt = await bcrypt.genSalt(10);
user.password = await bcrypt.hash(newPassword, salt);
user.originalPassword = newPassword;
user.resetPasswordCode = undefined;
user.resetPasswordExpiry = undefined;
await user.save();
```

### 5. **CORS Configuration**

```javascript
const corsOptions = {
  origin: [
    'http://localhost:3000',
    'http://localhost:5173',
    'https://quiz-website-mxfr.onrender.com'
  ],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'sessiontoken']
};

app.use(cors(corsOptions));
```

### 6. **Data Encryption**

#### Sensitive Data Encryption
```javascript
const crypto = require('crypto');

const encrypt = (text) => {
  const cipher = crypto.createCipher('aes-256-cbc', process.env.ENCRYPTION_KEY);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return encrypted;
};

const decrypt = (encryptedText) => {
  const decipher = crypto.createDecipher('aes-256-cbc', process.env.ENCRYPTION_KEY);
  let decrypted = decipher.update(encryptedText, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
};
```

#### Usage
- Original passwords for admin recovery
- Sensitive quiz credentials
- API keys storage

---

## ⚙️ Configuration & Setup

### 1. **Environment Variables**

#### Backend (.env)
```env
# Database
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/dbname

# Authentication
JWT_SECRET=your-secret-key-minimum-32-characters

# Email Service
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-16-char-app-password

# Encryption
ENCRYPTION_KEY=your-encryption-key-32-characters

# Server
PORT=5000
NODE_ENV=development
```

#### Frontend (.env)
```env
VITE_API_URL=http://localhost:5000
```

### 2. **Installation Steps**

#### Backend Setup
```bash
# Navigate to backend folder
cd backend

# Install dependencies
npm install

# Create .env file
cp .env.example .env
# Edit .env with your credentials

# Start development server
npm run server

# Or start production server
npm start
```

#### Frontend Setup
```bash
# Navigate to frontend folder
cd frontend

# Install dependencies
npm install

# Create .env file (optional)
cp .env.example .env

# Start development server
npm run dev

# Build for production
npm run build
```

### 3. **Database Setup**

#### MongoDB Atlas
1. Create account at mongodb.com
2. Create new cluster (free tier available)
3. Create database user with password
4. Whitelist IP (0.0.0.0/0 for all)
5. Get connection string
6. Replace in MONGODB_URI

#### Local MongoDB
```bash
# Install MongoDB
# macOS
brew install mongodb-community

# Start MongoDB service
brew services start mongodb-community

# Connection string
MONGODB_URI=mongodb://localhost:27017/quiz-db
```

### 4. **Initial Setup Wizard**

#### First Run
When no admin exists, system redirects to `/setup`

#### Setup Process
1. **College Information**
   - College name
   - Address
   - Contact details
   - Logo upload

2. **Admin Account**
   - Admin name
   - Email
   - Password
   - Department

3. **Academic Structure**
   - Departments
   - Years (1-4)
   - Semesters (1-8)
   - Sections (A-Z)

4. **Initial Subjects**
   - Subject code
   - Subject name
   - Department
   - Semester

### 5. **Production Deployment**

#### Backend (Render.com)
```bash
# Build command
npm install

# Start command
npm start

# Environment variables
# Add all .env variables in Render dashboard
```

#### Frontend (Vercel/Netlify)
```bash
# Build command
npm run build

# Output directory
dist

# Environment variables
VITE_API_URL=https://your-backend-url.onrender.com
```

#### Important Notes
- Update CORS origins in backend
- Use production MongoDB cluster
- Enable HTTPS
- Set NODE_ENV=production

---

## 📊 Data Flow Diagrams

### 1. **Academic Quiz Creation Flow**

```
Faculty Login
    ↓
Select Quiz Creation Method
    ↓
┌─────────────┬──────────────┬──────────────┬──────────────┐
│   Manual    │    Excel     │     Word     │    Image     │
└─────────────┴──────────────┴──────────────┴──────────────┘
    ↓               ↓               ↓               ↓
Enter Questions   Parse File    Parse DOCX     OCR Process
    ↓               ↓               ↓               ↓
    └───────────────┴───────────────┴───────────────┘
                        ↓
            Configure Quiz Settings
            - Duration
            - Allowed Groups
            - Negative Marking
            - Shuffling Options
            - Email Notification
                        ↓
                  Save to Database
                        ↓
        Email Notification to Eligible Students
```

### 2. **Student Quiz Taking Flow**

```
Student Login
    ↓
View Available Quizzes
    ↓
Select Quiz → Check Eligibility
    ↓
Start Quiz Attempt
    ↓
Security Features Activate
- Fullscreen Enforcement
- Right-click Disabled
- Copy/Paste Blocked
- Tab Switch Detection
    ↓
Answer Questions
- View Questions (shuffled if enabled)
- Select Answers
- Navigation Between Questions
    ↓
Submit Quiz
    ↓
Auto-Scoring
- Calculate Correct Answers
- Apply Negative Marking
- Total Score Calculation
    ↓
Save Submission
    ↓
Display Results
```

### 3. **Event Quiz Registration Flow**

```
Public User
    ↓
Browse Event Quizzes
    ↓
Select Quiz → Register
    ↓
Choose Participation Mode
    ↓
┌──────────────────┬──────────────────┐
│   Individual     │      Team        │
└──────────────────┴──────────────────┘
    ↓                      ↓
Enter Details        Enter Team Details
- Name               - Team Name
- Email              - Team Leader Info
- College            - Team Members Info
- Department         - Size: 1-10
- Phone
    ↓                      ↓
    └──────────────────────┘
              ↓
    Generate Credentials
    - Username = Email
    - Password = Auto-generated
              ↓
    Save to Database
    - QuizCredentials
    - EventQuiz.registrations
              ↓
    Send Email Notification
    - Login Credentials
    - Quiz Details
    - Instructions
```

### 4. **Password Reset Flow**

```
User Forgot Password
    ↓
Enter Email Address
    ↓
System Validates Email
    ↓
Generate 6-Digit Code
    ↓
Store Code + 10min Expiry
    ↓
Send Email with Code
    ↓
User Enters Code
    ↓
Verify Code & Expiry
    ↓
Enter New Password
    ↓
Hash New Password
    ↓
Update Database
    ↓
Clear Reset Code
    ↓
Success Message
```

---

## 🎯 Use Cases & Scenarios

### Scenario 1: Faculty Creates Weekly Test

**Steps:**
1. Faculty logs in
2. Creates quiz with 20 questions via Excel upload
3. Sets duration to 30 minutes
4. Selects allowed groups: CSE, Year 3, Semester 5, Sections A & B
5. Enables negative marking (1 mark per wrong answer)
6. Enables email notifications
7. Saves quiz

**System Actions:**
- Validates Excel format
- Parses questions
- Calculates total marks
- Finds eligible students (45 students)
- Sends 45 emails
- Quiz visible to students

### Scenario 2: Student Takes Quiz

**Steps:**
1. Student logs in
2. Sees "Data Structures Quiz" in available quizzes
3. Clicks "Start Quiz"
4. Browser enters fullscreen
5. Answers 18/20 questions
6. Attempts to switch tab (violation warning)
7. Completes quiz with 2 violations
8. Submits

**Results:**
- Score: 16/20 correct
- Marks: 64/100 (with negative marking)
- Time taken: 28 minutes
- Violations: 2
- Status: Evaluated

### Scenario 3: Inter-College Quiz Competition

**Steps:**
1. Event manager creates event quiz
2. Enables team participation (5 members)
3. Allows both college and external participants
4. Sets registration enabled
5. 20 teams register (100 participants)
6. System generates 20 sets of credentials
7. Sends emails to 100 participants
8. Teams take quiz on competition day
9. Top 5 teams selected
10. Follow-up quiz created for finalists
11. Credentials sent to 25 finalists

**Outcome:**
- Round 1: 20 teams
- Round 2: 5 teams
- Winner: Team "Code Warriors"

### Scenario 4: Admin Manages System

**Steps:**
1. Admin logs in
2. Bulk uploads 500 students via Excel
3. Creates 10 faculty accounts
4. Assigns subjects to faculty
5. Configures quiz security settings
6. Sets admin override to Ctrl+9
7. Updates college logo and branding
8. Views system analytics

**System State:**
- 500 students active
- 10 faculty members
- 50 subjects configured
- 3 departments
- Custom branding applied

---

## 📈 Performance & Scalability

### Database Indexes

#### User Collection
```javascript
db.users.createIndex({ email: 1 }, { unique: true })
db.users.createIndex({ admissionNumber: 1 }, { unique: true, sparse: true })
db.users.createIndex({ role: 1, department: 1 })
```

#### Quiz Collection
```javascript
db.quizzes.createIndex({ title: 1 }, { unique: true })
db.quizzes.createIndex({ createdBy: 1 })
db.quizzes.createIndex({ startTime: 1, endTime: 1 })
db.quizzes.createIndex({ "allowedGroups.department": 1 })
```

#### QuizSubmission Collection
```javascript
db.quizsubmissions.createIndex({ quiz: 1, student: 1 })
db.quizsubmissions.createIndex({ student: 1, isDeleted: 1 })
db.quizsubmissions.createIndex({ status: 1 })
```

### Optimization Techniques

#### 1. Lazy Loading
- Questions loaded on demand
- Paginated submission lists
- Infinite scroll for quiz lists

#### 2. Caching Strategy
```javascript
// Cache academic details (rarely changes)
const cachedAcademicDetails = await AcademicDetail.findOne()
  .lean()
  .cache(3600); // 1 hour cache
```

#### 3. Aggregation Pipeline
```javascript
// Efficient statistics calculation
const stats = await QuizSubmission.aggregate([
  { $match: { quiz: quizId } },
  { $group: {
      _id: null,
      avgScore: { $avg: "$score" },
      maxScore: { $max: "$score" },
      minScore: { $min: "$score" },
      totalSubmissions: { $sum: 1 }
    }
  }
]);
```

#### 4. Async Email Sending
```javascript
// Don't wait for email completion
sendEmailNotifications(quiz, students)
  .then(result => console.log('Emails sent:', result))
  .catch(err => console.error('Email error:', err));

// Return response immediately
res.json({ success: true, quiz });
```

### Scalability Considerations

#### Current Capacity
- **Users**: 10,000+ concurrent
- **Quizzes**: Unlimited
- **Submissions**: Millions (with indexing)
- **File Uploads**: 5MB per file

#### Bottlenecks & Solutions
1. **Email Sending**
   - Problem: Sequential sending is slow
   - Solution: Batch processing, queue system

2. **Large Quiz Lists**
   - Problem: Loading all quizzes at once
   - Solution: Pagination, filtering

3. **File Processing**
   - Problem: OCR processing is CPU-intensive
   - Solution: Background jobs, worker threads

4. **Concurrent Quiz Taking**
   - Problem: Multiple students submitting simultaneously
   - Solution: Database connection pooling, load balancing

---

## 🔧 Troubleshooting Guide

### Common Issues

#### 1. Email Not Sending
**Symptoms:** Registration emails not received

**Solutions:**
- Check EMAIL_USER and EMAIL_PASS in .env
- Verify Gmail App Password (not regular password)
- Check Gmail account 2FA is enabled
- Verify recipient email is valid
- Check spam folder

#### 2. MongoDB Connection Failed
**Symptoms:** "MongoNetworkError" or "ECONNREFUSED"

**Solutions:**
- Verify MONGODB_URI in .env
- Check MongoDB service is running (local)
- Whitelist IP in MongoDB Atlas
- Check network connectivity
- Verify database user credentials

#### 3. JWT Token Expired
**Symptoms:** "Token has expired" error after some time

**Solutions:**
- This is normal behavior (24-hour expiry)
- User should re-login
- Can increase expiry in JWT_SECRET configuration

#### 4. File Upload Failed
**Symptoms:** "Invalid file type" or upload error

**Solutions:**
- Check file size (max 5MB)
- Verify file format (xlsx, docx, jpg, png, gif)
- Check uploads/ folder permissions
- Ensure multer is configured correctly

#### 5. Quiz Not Visible to Students
**Symptoms:** Students can't see created quiz

**Solutions:**
- Check allowedGroups matches student's details
- Verify startTime is in the past
- Verify endTime is in the future
- Check isActive is true
- Verify student's department/year/section

#### 6. Security Features Not Working
**Symptoms:** Fullscreen not enforced, violations not tracked

**Solutions:**
- Check browser permissions for fullscreen
- Verify QuizSecurity component is loaded
- Check quiz settings configuration
- Test in supported browsers (Chrome, Firefox, Edge)
- Disable browser extensions that may interfere

#### 7. OCR Not Recognizing Text
**Symptoms:** Image upload returns gibberish or empty text

**Solutions:**
- Use high-quality images (300+ DPI)
- Ensure text is clear and readable
- Use printed text (better than handwritten)
- Pre-process image (contrast, brightness)
- Try different image format

---

## 🧪 Testing Guidelines

### Manual Testing Checklist

#### Admin Features
- [ ] Create admin account
- [ ] Bulk upload students via Excel
- [ ] Create faculty accounts
- [ ] Assign subjects to faculty
- [ ] Update college settings
- [ ] Configure quiz security
- [ ] View system statistics

#### Faculty Features
- [ ] Create quiz manually
- [ ] Create quiz via Excel upload
- [ ] Create quiz via Word upload
- [ ] Create quiz via Image OCR
- [ ] Edit existing quiz
- [ ] Delete quiz
- [ ] View submissions
- [ ] Export results to PDF

#### Student Features
- [ ] Login with credentials
- [ ] View available quizzes
- [ ] Start quiz attempt
- [ ] Answer questions
- [ ] Submit quiz
- [ ] View results
- [ ] Review completed quizzes
- [ ] Update profile

#### Event Quiz Features
- [ ] Create event quiz
- [ ] Register as individual
- [ ] Register as team
- [ ] Generate credentials
- [ ] Login with credentials
- [ ] Take event quiz
- [ ] View event submissions
- [ ] Create follow-up quiz
- [ ] Send bulk emails

#### Security Features
- [ ] Fullscreen enforcement
- [ ] Tab switch detection
- [ ] Right-click disabled
- [ ] Copy/paste blocked
- [ ] Violation warnings
- [ ] Admin override access

---

## 📚 Best Practices

### For Administrators
1. **Initial Setup**
   - Complete setup wizard thoroughly
   - Configure all academic structures
   - Test email service before bulk operations
   - Set appropriate quiz security settings

2. **User Management**
   - Use bulk upload for large batches
   - Verify data before uploading
   - Keep admission numbers unique
   - Regularly backup user data

3. **System Maintenance**
   - Monitor email quota usage
   - Clean up old quiz submissions
   - Review security logs
   - Update admin override password regularly

### For Faculty
1. **Quiz Creation**
   - Test quiz before publishing
   - Verify all questions have correct answers
   - Set realistic time limits
   - Use clear, unambiguous question text
   - Preview quiz from student perspective

2. **Excel Upload Format**
   - Use provided template
   - Verify column headers match exactly
   - Check for special characters
   - Test with small batch first

3. **Grading & Review**
   - Review submissions promptly
   - Check for anomalies (unusually high/low scores)
   - Export results for record-keeping
   - Provide feedback to students

### For Students
1. **Taking Quizzes**
   - Login 15 minutes before start time
   - Ensure stable internet connection
   - Use updated browser
   - Avoid switching tabs/windows
   - Read instructions carefully

2. **Technical Requirements**
   - Use desktop/laptop (not mobile)
   - Allow fullscreen permissions
   - Disable browser extensions
   - Have backup device ready
   - Note down credentials safely

### For Event Managers
1. **Event Quiz Setup**
   - Create quiz well in advance
   - Test registration process
   - Verify email delivery
   - Prepare backup credentials
   - Have support team ready

2. **Registration Management**
   - Monitor registrations regularly
   - Respond to queries promptly
   - Send reminder emails before event
   - Keep backup of all registrations

3. **During Event**
   - Monitor submissions in real-time
   - Have technical support available
   - Track violation reports
   - Provide help desk for issues

---

## 🔮 Future Enhancements

### Planned Features
1. **AI-Powered Question Generation**
   - Automatic question creation from syllabus
   - Difficulty level assessment
   - Similar question detection

2. **Advanced Analytics**
   - Student performance trends
   - Topic-wise analysis
   - Predictive scoring
   - Comparative analytics

3. **Mobile Application**
   - Native iOS/Android apps
   - Offline quiz support
   - Push notifications
   - Biometric authentication

4. **Video Proctoring**
   - Webcam monitoring
   - Face detection
   - Screen recording
   - Automated violation detection

5. **Question Bank**
   - Centralized question repository
   - Tagging and categorization
   - Difficulty ratings
   - Reusability across quizzes

6. **Collaborative Features**
   - Faculty collaboration on quizzes
   - Peer review system
   - Question sharing
   - Department-wide question pools

7. **Enhanced Reporting**
   - Custom report builder
   - Scheduled report generation
   - Export to multiple formats
   - Data visualization dashboard

8. **Integration Capabilities**
   - LMS integration (Moodle, Canvas)
   - Google Classroom sync
   - Microsoft Teams integration
   - Zoom meeting links

---

## 📞 Support & Contact

### Getting Help
- **Documentation**: This file
- **Email Support**: admin@quizapp.com (configure your support email)
- **Issue Reporting**: GitHub Issues (if using version control)

### Development Team
- **Backend Development**: Node.js + Express + MongoDB
- **Frontend Development**: React + Vite + Material-UI
- **Security Implementation**: JWT + Bcrypt + CORS
- **Email Service**: Nodemailer + Gmail SMTP
- **File Processing**: Multer + XLSX + Mammoth + Tesseract.js

---

## 📄 License & Credits

### Open Source Libraries Used
- **React**: MIT License
- **Express**: MIT License
- **MongoDB/Mongoose**: Server Side Public License
- **Material-UI**: MIT License
- **Nodemailer**: MIT License
- **Tesseract.js**: Apache License 2.0
- **XLSX**: Apache License 2.0

### Acknowledgments
- MongoDB Atlas for database hosting
- Gmail SMTP for email service
- Tesseract OCR for text recognition
- Material-UI team for beautiful components

---

## 🎓 Glossary

**Academic Quiz**: Quiz created by faculty for their students based on academic structure

**Admin Override**: Emergency access feature to bypass security restrictions

**Allowed Groups**: Student groups (department/year/section) eligible for a quiz

**Bulk Upload**: Uploading multiple records via Excel file

**Deterministic Shuffling**: Same user always gets same question order using hash-based seeding

**Event Quiz**: Cross-college competition quiz open to external participants

**Follow-up Quiz**: Subsequent quiz created for selected participants from previous quiz

**JWT (JSON Web Token)**: Authentication token format used for user sessions

**Negative Marking**: Penalty marks deducted for incorrect answers

**OCR (Optical Character Recognition)**: Technology to extract text from images

**Question Limiting**: Showing subset of questions to each student from larger pool

**RBAC (Role-Based Access Control)**: Permission system based on user roles

**Session Token**: Temporary authentication token for event quiz participants

**Soft Delete**: Marking records as deleted without removing from database

**Spot Registration**: On-site registration during event

**Violation**: Security breach attempt (tab switch, exit fullscreen, etc.)

---

## 📊 System Statistics & Limits

### Current Limits
| Feature | Limit | Configurable |
|---------|-------|--------------|
| Max File Upload Size | 5 MB | Yes (multer config) |
| JWT Token Expiry | 24 hours | Yes (JWT config) |
| Password Reset Code Expiry | 10 minutes | Yes (code generation) |
| Max Team Size | 10 members | Yes (event quiz config) |
| Max Violations | 3 | Yes (quiz settings) |
| Quiz Duration | No limit | Per quiz |
| Questions per Quiz | No limit | Recommended < 500 |
| Concurrent Users | 10,000+ | Server dependent |

### Database Collections Overview
| Collection | Purpose | Indexes | Average Docs |
|------------|---------|---------|--------------|
| users | User accounts | 3 | 1,000 - 10,000 |
| quizzes | Academic quizzes | 4 | 100 - 1,000 |
| eventquizzes | Event quizzes | 3 | 10 - 100 |
| quizsubmissions | Student submissions | 3 | 10,000+ |
| eventquizresults | Event submissions | 2 | 1,000+ |
| quizcredentials | Event credentials | 2 | 100 - 1,000 |
| quizsettings | Security config | 1 | 1 (singleton) |
| academicdetails | Academic structure | 1 | 1 (singleton) |
| colleges | College info | 1 | 1 (singleton) |
| departments | Department list | 1 | 10 - 50 |

---

## 🗺️ Project Structure

### Backend Structure
```
backend/
├── models/               # Mongoose schemas
│   ├── User.js
│   ├── Quiz.js
│   ├── EventQuiz.js
│   ├── QuizSubmission.js
│   ├── EventQuizResult.js
│   ├── QuizCredentials.js
│   ├── QuizSettings.js
│   ├── AcademicDetail.js
│   ├── College.js
│   └── Department.js
├── routes/               # API endpoints
│   ├── auth.js
│   ├── quiz.js
│   ├── eventQuiz.js
│   ├── admin.js
│   ├── academicDetails.js
│   └── setup.js
├── middleware/           # Custom middleware
│   ├── auth.js
│   ├── admin.js
│   └── authorize.js
├── services/             # Business logic
│   ├── emailService.js
│   └── imageProcessing.js
├── utils/                # Utility functions
│   └── encryption.js
├── uploads/              # File uploads storage
├── app.js                # Express app config
├── server.js             # Server entry point
├── package.json
└── .env.example
```

### Frontend Structure
```
frontend/
├── src/
│   ├── components/
│   │   ├── admin/           # Admin components
│   │   ├── auth/            # Authentication
│   │   ├── common/          # Shared components
│   │   ├── dashboard/       # Dashboard views
│   │   ├── event/           # Event quiz components
│   │   ├── profile/         # User profile
│   │   ├── quiz/            # Quiz components
│   │   ├── student/         # Student components
│   │   ├── subject/         # Subject management
│   │   └── setup/           # Initial setup
│   ├── context/             # React Context
│   │   ├── AuthContext.jsx
│   │   ├── ThemeContext.jsx
│   │   ├── LoadingContext.jsx
│   │   └── NavigationContext.jsx
│   ├── hooks/               # Custom hooks
│   ├── routes/              # Route configurations
│   ├── utils/               # Utility functions
│   ├── App.jsx              # Main app component
│   ├── main.jsx             # Entry point
│   └── index.css            # Global styles
├── public/
│   └── assets/              # Static assets
├── package.json
├── vite.config.js
└── .env.example
```

---

## 🔐 Security Best Practices

### Password Security
✅ **DO:**
- Use bcrypt with salt rounds ≥ 10
- Enforce minimum password length (6+ characters)
- Store original password encrypted for admin recovery
- Implement password reset with time-limited codes

❌ **DON'T:**
- Store passwords in plain text
- Use weak hashing algorithms (MD5, SHA1)
- Allow common passwords
- Share passwords via insecure channels

### JWT Token Security
✅ **DO:**
- Use strong JWT_SECRET (32+ characters)
- Set reasonable expiration times
- Validate tokens on every request
- Store tokens securely (httpOnly cookies or localStorage)

❌ **DON'T:**
- Use weak secrets
- Set unlimited expiration
- Trust client-side token modifications
- Expose tokens in URLs

### API Security
✅ **DO:**
- Implement CORS with specific origins
- Use HTTPS in production
- Validate all input data
- Sanitize user inputs
- Rate limit API endpoints
- Log security events

❌ **DON'T:**
- Allow all CORS origins (*)
- Trust client-side validation alone
- Expose sensitive data in error messages
- Skip input validation

### Database Security
✅ **DO:**
- Use environment variables for credentials
- Implement least privilege access
- Regular backups
- Index frequently queried fields
- Use connection pooling

❌ **DON'T:**
- Hardcode database credentials
- Grant admin access to application user
- Store sensitive data unencrypted
- Skip data validation

---

## 📝 Summary

The Quiz Management System is a production-ready, enterprise-grade application that provides:

### ✨ Key Strengths
1. **Comprehensive Role Management** - Four distinct user types with granular permissions
2. **Flexible Quiz Creation** - Multiple input methods for different use cases
3. **Advanced Security** - Multi-layered anti-cheating measures
4. **Event Management** - Full support for inter-college competitions
5. **Automated Workflows** - Email notifications and credential management
6. **Scalable Architecture** - Built to handle thousands of concurrent users
7. **Modern Tech Stack** - MERN stack with best practices
8. **Extensive Documentation** - Complete API and feature documentation

### 🎯 Ideal For
- Educational institutions (colleges, universities)
- Online learning platforms
- Corporate training programs
- Competitive exam organizations
- Assessment service providers

### 🚀 Quick Start
```bash
# Clone repository
git clone <repository-url>

# Setup backend
cd backend
npm install
cp .env.example .env
# Edit .env with your credentials
npm start

# Setup frontend (in new terminal)
cd frontend
npm install
npm run dev

# Access application
# Frontend: http://localhost:5173
# Backend: http://localhost:5000
```

---

## 📞 Need More Information?

This documentation covers the entire project. For specific queries:

1. **Feature-specific details**: Check the relevant section above
2. **API details**: See API Endpoints section
3. **Setup issues**: Check Configuration & Setup and Troubleshooting
4. **Database schema**: See Database Models section
5. **Security concerns**: See Authentication & Security section

---

**Last Updated**: June 6, 2026  
**Version**: 1.0.0  
**Document Author**: AI Documentation System

---

**End of Documentation**
