Smart Campus Issue Management Portal
1. Problem Statement
Smart Campus Issue Management Portal

The Smart Campus Issue Management Portal is a web-based platform designed to help students report and track campus-related problems such as classroom equipment failures, network issues, cleanliness problems, and infrastructure damage.

The system allows students to submit complaints with relevant details such as category, priority, location, description, and supporting images. Administrators can monitor complaints, prioritize them, assign them to responsible staff, and track their progress until resolution.

Key Features

Student registration and login

Campus issue/complaint submission

Issue category and priority selection

Image/file attachment

Unique complaint/ticket ID

Admin dashboard

Issue assignment to staff

Status tracking

Notifications

Complaint history

Analytics and reports

Role-based access control

User Roles

Student – Submit complaints and track their status

Admin – Manage, prioritize, assign, and monitor complaints

Staff/Technician – Handle assigned complaints and update their status

2. Architecture Flowchart
                         ┌─────────────────────┐
                         │       Student       │
                         │   Login / Register  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Submit Campus    │
                         │       Issue        │
                         └──────────┬──────────┘
                                    │
                                    ▼
                    ┌──────────────────────────────┐
                    │ Category + Priority +       │
                    │ Location + Description      │
                    │ + Image Attachment          │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                         ┌─────────────────────┐
                         │     Backend API     │
                         │ Node.js + Express   │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │      Database       │
                         │       MySQL         │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Admin Dashboard   │
                         └──────────┬──────────┘
                                    │
                         ┌──────────┴──────────┐
                         ▼                     ▼
                ┌────────────────┐    ┌────────────────┐
                │ Assign Issue   │    │ Set Priority   │
                │ to Staff       │    │ & Status       │
                └───────┬────────┘    └───────┬────────┘
                        │                     │
                        └──────────┬──────────┘
                                   ▼
                         ┌─────────────────────┐
                         │ Staff / Technician  │
                         │ Handles the Issue   │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Update Issue Status │
                         │ In Progress/Resolved│
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Student Notification│
                         │ & Status Tracking   │
                         └─────────────────────┘

System Architecture
┌─────────────────────────────────────────────┐
│                  Frontend                   │
│              React.js + CSS                │
└──────────────────────┬──────────────────────┘
                       │ REST API
                       ▼
┌─────────────────────────────────────────────┐
│                  Backend                    │
│          Node.js + Express.js              │
│                                             │
│ Authentication │ Issue Management │ Admin  │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│                  Database                   │
│                   MySQL                    │
│                                             │
│ Users │ Issues │ Categories │ History      │
└─────────────────────────────────────────────┘

3. Technical Stack
Frontend

React.js – Build the user interface

HTML5 – Structure

CSS3 – Styling and responsive design

Bootstrap / Tailwind CSS – UI components and responsive layouts

Axios – API communication

Recharts / Chart.js – Analytics and dashboard charts

Backend

Node.js – Server-side runtime

Express.js – REST API development

JWT – Authentication and authorization

bcrypt – Password hashing

Multer – File/image uploads

Database

MySQL – Store users, complaints, categories, assignments, status history, and notifications

Development Tools

Git & GitHub – Version control

VS Code – Development environment

Postman – API testing

npm – Package management

Optional Technologies

Cloudinary – Store uploaded complaint images

Nodemailer – Email notifications

Socket.IO – Real-time status notifications

Technology Architecture
React.js
   │
   │ Axios / REST API
   ▼
Node.js + Express.js
   │
   ├── JWT Authentication
   ├── Issue Management
   ├── Assignment System
   ├── Status Management
   └── Analytics API
   │
   ▼
MySQL Database
   │
   ├── Users
   ├── Issues
   ├── Categories
   ├── Assignments
   ├── Issue History
   └── Notifications

Project Challenge Level

Moderate – High

The project involves authentication, role-based access control, CRUD operations, issue assignment, status tracking, file uploads, notifications, and analytics.

4. Team Member Name:-
  1. G.Srinath
  2. M.Keerthigaraj
  3. J.Vishnu Vardhan
  4. S.K.Tharani

5.College Name:-
Mahendra Engineering College (Autonomous) Namakkal.

