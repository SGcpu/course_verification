# FRCRCE Credit Substitution Portal Roadmap

## Phase 1: In-depth Planning, Infrastructure Setup & Core Design

### 1. Detailed Project Scope & FRCRCE Specific Requirements Definition:
*Problem Statement Elaboration*: Clearly define the core purpose: to streamline the process for FRCRCE’s CSE department's 3rd-year students (7th semester) to substitute certain electives with SWAYAM/NPTEL MOOC courses. The portal must reduce paperwork and errors in credit transfer by automating approvals and record-keeping.

*Key Objectives:*
- Managing NPTEL course approvals.
- Tracking earned credits against specific requirements (e.g., 20 total MOOC credits required for 7th-semester substitution).
- Generating/updating marksheets (transcripts) reflecting these substitute courses.
- User Roles & Permissions (RBAC Model): Establish two distinct roles: Student and Faculty.  
- Student: Can view profile, submit courses, track credits, view past semester data, view/upload certificates, and use the chat.  
- Faculty (or designated SPOC/mentor): Can view pending student requests, approve/reject courses, access student records, and use the chat.
    
### 2. Robust Technology Stack Selection & Initial Setup:
*Frontend:*
- React.js (JavaScript): Chosen for building a responsive user interface.
- React Router: Essential for client-side navigation between different portal sections (e.g., Dashboard, Upload Course, Credit Status).
- Component-based Design: Implement forms, tables, and the chat UI using React components for modularity and reusability.
- CSS Framework: Integrate a framework like Tailwind or Bootstrap for efficient styling and consistent visual appearance across pages.

*Backend:*
- Django (Python): Acts as the web server and API layer.
- Django REST Framework: Used to create secure API endpoints for user login, course requests, and data management.
- Django Channels (ASGI): Crucial for implementing real-time chat via WebSockets.
- Redis (Optional but Recommended): As a channel layer for Django Channels to manage WebSocket connections and message broadcasting efficiently.
  
*Database:*
- PostgreSQL: Selected for storing all structured data due to its reliability and data integrity.
- Initial Schema Design: Define tables such as User (with id, username/roll_number, hashed_password, role field, pointers to student/faculty profiles), Student, Faculty, CourseRequest (with id, student_id, course_name, duration, marks, status, certificate_file path), and Certificate.

*Version Control:*
- Set up a Git repository (e.g., on GitHub) to manage code changes, enable collaborative development, and track history.
    
### 3. Comprehensive Security & Best Practices Integration:
- HTTPS/TLS: Mandate secure communication over HTTPS for all data in transit. Implement SSL certificates (e.g., Let's Encrypt) in production.
- Password Hashing: Store user passwords with robust hashing algorithms like bcrypt or Argon2 to prevent plaintext storage.
- AES Encryption for Data at Rest: Implement AES encryption for sensitive data and uploaded files (e.g., certificate PDFs, personal information) stored in the database or file storage.
- Role-Based Access Control (RBAC): Ensure the backend rigorously enforces permissions based on user roles, limiting access to features and data.
    
### 4. Initial UI/UX Design & Wireframing (Conceptualization):
- Consistent Branding: Incorporate FRCRCE/CSE branding, logos, and departmental colors for a cohesive look.
- Navigation Structure: Design the Homepage with four main tabs: Home, About, Login, and Help.  
  Post-login, design distinct header/side menus for:  
  **Student:** Dashboard, Upload Course, Credit Status, View Past Semesters, Certificates, Chat, Logout.  
  **Faculty:** Dashboard, Pending Requests, Approved Courses, Chat, Logout.
- Wireframing: Create basic sketches or digital wireframes for all key pages (Homepage, Login, Student/Faculty Dashboards, Course Upload Form, Credit Status table, Chat widget) to define layout, content placement, and user flow before development.
- Simplicity & Clarity: Ensure a simple and uncluttered layout, especially for the login page and forms, with clearly labeled menus and buttons (e.g., "Submit", "Approve", "Reject").
    
## Phase 2: Core Feature Development & Integration (Iterative)

### 1. User Authentication & Dashboard Implementation:
- **Login Page Development:** Create a straightforward login form with input fields for Username/Roll No. and Password, plus a Role selector (Student or Faculty).  
  No sign-up or password reset functionality on the portal; users lacking credentials should contact an admin.
- **Backend Authentication API:** Develop Django REST Framework API endpoints for user login, validating credentials against hashed passwords, and issuing session tokens or JSON Web Tokens (JWTs) that include the user's role.
- **Role-Based Redirection:** After successful authentication, redirect users to their respective dashboards based on the "role" field in the user model.
- **Student Dashboard:**  
  * Default View: Display a welcome message (e.g., "Welcome, [Name]"), a summary of current status (total credits earned vs. required, number of pending course requests), and quick links to key actions.  
  * Past Semester Courses (View Only): Implement a table showing historical data: Semester, Course Code, Course Title, Credits, Grade, Year, and a calculated CGPA, which students can view but not edit.
- **Faculty Dashboard:**  
  * Default View: Display an overview such as "Pending Approvals: X courses" and "Approved this month: Y," with shortcuts to key actions.
    
### 2. Course Submission & Faculty Approval Workflow:
- **Student Course Upload Interface ("Upload Course" tab):**  
  * Develop a detailed form for students to submit NPTEL/SWAYAM course information: Course Name and Code, Instructor or Provider, Course Duration, and Obtained Marks/Grade.  
  * Include a file upload field for the course completion certificate (PDF/image).  
  * Upon submission, the backend saves a new CourseRequest record with status "Pending" and stores the uploaded file in secure (AES-encrypted) storage.
- **Student "Pending Approval" Tracking:** The student's dashboard displays submitted requests in a "Pending Approval" table, allowing corrections or cancellation before approval.
- **Faculty Review Interface ("Pending Requests" tab):**  
  * Display all pending course requests with student details, course info, and "Approve" / "Reject" buttons.  
  * Include a link to view the uploaded certificate for each request.  
  * Implement confirmation prompts for deliberate approval/rejection.  
  * Approval changes status to "Approved" and triggers credit calculation. Rejection sets status to "Rejected" and notifies the student.
- **Faculty "Approved Courses" View:** Shows all approved requests with student info, course title, credits, and approval date.
    
### 3. Credit Tracking & Certificate Management:
- **Credit Mapping Logic:**  
  * 4-week course = 1 credit  
  * 8-week course = 2 credits  
  * 12-week course = 3 credits  
  * 15/16-week course = 4 credits
- **Student Credit Summary ("Credit Status" tab):**  
  * Display the student’s credit tally, showing a table with Course Name, Duration, Credits Earned.  
  * Show Total Earned credits, Credits Required (20), and Remaining credits.  
  * List substitution-eligible subjects for the 7th semester with earned/pending indicators.
- **Dynamic Calculation:** Credits update in real-time on course approval or withdrawal.
- **Certificate Attachment Process ("Certificates" tab):**  
  * After approval, students can upload the official course certificate.  
  * Securely store files using AES encryption.  
  * Display certificate upload status with view/download options.

### 4. Grading, CGPA Calculation & Selective Course Submission:
- **Grading Scale Implementation:**  
  * ≥ 85: Grade O (Outstanding), Points 10  
  * 80–84.99: Grade A (Excellent), Points 9  
  * 70–79.99: Grade B (Very Good), Points 8  
  * 60–69.99: Grade C (Good), Points 7  
  * 50–59.99: Grade D (Fair), Points 6  
  * 45–49.99: Grade E (Average), Points 5  
  * 40–44.99: Grade P (Pass), Points 4  
  * < 40: Grade F (Fail), Points 0 (or AB for absent)
- **CGPA Calculation:** Weighted average CGPA = Σ(grade_points × credits) / Σcredits.
- **Selective Course Submission Interface:** Checkboxes to choose which approved courses appear on the final marksheet.
- **"Submit for Marksheet" Action:** Locks selected courses for PDF generation until deadline.

## Phase 3: Advanced Features, Deployment & Maintenance

### 1. Real-time Chat System ("Chat" tab/widget):
- WebSocket Integration using Django Channels.
- Floating chat widget visible post-login.
- Full-duplex communication between students and faculty.
- Support for text messages, timestamps, and sender names.
- Store all chat messages in the database.
- Organize conversations into rooms per student-faculty pair.
- Visual cues for unread messages; allow transcript saving.

### 2. Official Marksheets PDF Generation:
- Use a Python PDF library like ReportLab or WeasyPrint.
- Include student info, selected courses, credits, grades, grade points, total credits, and CGPA.
- Embed or reference certificate images for each course.
- Include authorized signatures/stamps.
- Output downloadable PDF that matches portal’s course table.

### 3. Thorough Testing and Quality Assurance:
- Unit testing for backend logic.
- Integration testing between frontend, backend, and database.
- User Acceptance Testing with actual FRCRCE students/faculty.
- Security audits for vulnerabilities.

### 4. Deployment, Monitoring & Continuous Maintenance:
- Deploy backend on Render/Railway or college server; frontend on Vercel/Netlify.
- Set up CI/CD with GitHub Actions.
- Ensure SSL certificates are active.
- Implement monitoring, logging, and error tracking.
- Maintain detailed technical documentation and user guides.
 regularly update detailed project documentation, including technical architecture, API specifications, database schema, and comprehensive user guides for both students and faculty. This documentation should be simple, clear, and structured with numbered sections, subheadings, sample tables, and bullet lists, similar to the provided source documents [37, 38].
--------------------------------------------------------------------------------
This detailed roadmap, incorporating the specific features, technologies, and institutional context from the provided sources, offers a comprehensive guide for the successful implementation and launch of the FRCRCE CSE NPTEL/SWAYAM Credit Substitution Portal.
