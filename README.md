# Freelance Platform (SQL + Appsmith)

A mini freelance job platform built using **Supabase (PostgreSQL)** as the backend and **Appsmith** for building UI and frontend logic. This project focuses on implementing **SQL concepts** in a real-world application environment.

---

## ⚙️ Tech Stack

- **Frontend/UI**: [Appsmith](https://www.appsmith.com/) (low-code builder)
- **Database**: [Supabase](https://supabase.com/) (PostgreSQL)
- **Backend Logic**: SQL queries inside Appsmith

---

## 🌟 Key Features

- 👤 **User Authentication** (Login via email for all roles)
- 📋 **Role-based Dashboards** (Client, Freelancer, Admin)
- 🧾 **Project Posting** (Clients can post projects)
- 🧑‍💻 **Proposal Submission** (Freelancers apply to projects)
- ✅ **Proposal Review** (Clients accept/reject proposals)
- ✏️ **CRUD Operations** for users, projects, and proposals
- 🔒 **Auth Guard**: Protect dashboard pages based on login role
- 🔄 **Auto refresh** after actions like Apply, Delete, Accept

---

## 📂 Database Tables

### `users`
| Column       | Type      | Description          |
|--------------|-----------|----------------------|
| user_id      | UUID/INT  | Primary Key          |
| name         | TEXT      | Full Name            |
| email        | TEXT      | Unique Email         |
| role         | TEXT      | 'admin' / 'client' / 'freelancer' |

### `projects`
| Column       | Type     | Description          |
|--------------|----------|----------------------|
| project_id   | UUID/INT | Primary Key          |
| title        | TEXT     | Project Title        |
| description  | TEXT     | Project Description  |
| budget       | INT      | Project Budget       |
| category     | TEXT     | Project Category     |
| deadline     | DATE     | Submission Deadline  |
| client_id    | UUID/INT | FK from users        |

### `proposals`
| Column         | Type     | Description                   |
|----------------|----------|-------------------------------|
| proposal_id    | UUID/INT | Primary Key                   |
| project_id     | UUID/INT | FK from projects              |
| user_id        | UUID/INT | FK from users (freelancer)    |
| proposal_text  | TEXT     | Proposal Description          |
| status         | TEXT     | 'pending' / 'accepted' / 'rejected' |

---

## 🧠 Key SQL Queries & Use Cases

### 🔐 Login Query
```sql
SELECT * FROM users
WHERE email = {{ inputLoginEmail.text }};
```

### 📢 Post New Project (Client)
```sql
INSERT INTO projects (title, description, budget, category, deadline, client_id)
VALUES (
  {{ inputTitle.text }},
  {{ inputDesc.text }},
  {{ inputBudget.text }},
  {{ dropdownCategory.selectedOptionValue }},
  {{ dateDeadline.selectedDate }},
  {{ appsmith.store.loggedInUser.user_id }}
);
```

### 🔍 Open Projects for Freelancers (Excludes Applied Ones)
```sql
SELECT * FROM projects
WHERE project_id NOT IN (
  SELECT project_id FROM proposals
  WHERE user_id = {{ appsmith.store.loggedInUser.user_id }}
);
```

### ✍️ Submit Proposal (Freelancer)
```sql
INSERT INTO proposals (project_id, user_id, proposal_text)
VALUES (
  {{ tblOpenProjects.selectedRow.project_id }},
  {{ appsmith.store.loggedInUser.user_id }},
  {{ inputProposal.text }}
);
```

### 📄 Proposals for Client Projects
```sql
SELECT p.*, u.name AS freelancer_name
FROM proposals p
JOIN users u ON u.user_id = p.user_id
WHERE p.project_id IN (
  SELECT project_id FROM projects
  WHERE client_id = {{ appsmith.store.loggedInUser.user_id }}
);
```

### ✅ Accept/Reject Proposal
```sql
UPDATE proposals
SET status = 'accepted'
WHERE proposal_id = {{ tblProposals.selectedRow.proposal_id }};
```
```sql
UPDATE proposals
SET status = 'rejected'
WHERE proposal_id = {{ tblProposals.selectedRow.proposal_id }};
```
---

## 🔐 User Roles & Dashboards

| Role        | Dashboard View                   | Permissions                                |
|-------------|----------------------------------|---------------------------------------------|
| **Admin**   | View all users                   | View, edit, and delete users                |
| **Client**  | Post and manage projects         | Create projects, view proposals, accept/reject proposals |
| **Freelancer** | Browse and apply to projects | View open projects, submit proposals, check proposal status |

---

## 🚀 How to Use

1. **Login** as one of the roles (admin, client, freelancer).
2. **Client Dashboard**:
   - Post a new project with details like title, description, category, budget, deadline.
   - View proposals submitted by freelancers.
   - Accept or reject proposals (disabled after first action).
3. **Freelancer Dashboard**:
   - Browse open projects (excluding ones already applied to).
   - Submit a proposal with a short description.
   - View status of submitted proposals.
4. **Admin Dashboard**:
   - View list of all registered users.
   - Edit or delete users from the platform.

---

## 📌 Setup Instructions (Optional)

If you're setting up this project:

1. **Supabase Setup**:
   - Create a Supabase project.
   - Set up tables: `users`, `projects`, `proposals` (schema shared above).
   - Enable Row Level Security (RLS) if needed.

2. **Appsmith Setup**:
   - Connect Supabase as a PostgreSQL datasource.
   - Create pages: Login, Client Dashboard, Freelancer Dashboard, Admin Dashboard.
   - Use `storeValue`, `navigateTo`, and `showModal` for user interactions.
   - Create JS Objects to manage login/auth, button actions, and guards.

