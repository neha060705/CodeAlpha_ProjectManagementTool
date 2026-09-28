# CodeAlpha_ProjectManagementTool

**Task 3 — Project Management Tool** (CodeAlpha Full Stack Development Internship)

A Trello/Asana-style collaborative tool: create group projects, invite members, manage
tasks on a Kanban board (To Do / In Progress / Done), assign tasks to teammates, and
comment on tasks — with **real-time live updates over WebSockets** (the bonus feature).

## Features
- User registration & login (JWT-based auth, hashed passwords)
- Create projects; creator becomes the owner automatically
- Invite existing users to a project by email
- Kanban board with three columns: To Do, In Progress, Done
- Create tasks, assign them to any project member, change status via dropdown
- Click a task to open it: view description, comment thread, delete task
- **Bonus — real-time updates:** everyone viewing the same project's board sees new
  tasks, status changes, and comments appear live via WebSockets, no refresh needed

## Tech Stack
- **Frontend:** HTML, CSS, JavaScript (no framework, no build step)
- **Backend:** Node.js, Express.js
- **Real-time:** the `ws` WebSocket library, sharing the same HTTP server/port as the API
- **Database:** SQLite via Node's built-in `node:sqlite` module — no native compiler
  or Visual Studio needed, no separate DB server, just one local file. Requires
  Node.js 22.5+ (24+ recommended, where it's stable with no warning).
- **Auth:** JSON Web Tokens (JWT) + bcrypt password hashing

---

## 1. Prerequisites
Install **Node.js** (v22.5 or newer; v24+ recommended) from https://nodejs.org.
Check it's installed:
```bash
node -v
npm -v
```

## 2. Setup
Unzip the project, then open a terminal **in the project's root folder** (the one that
contains both `backend` and `frontend`), then:

```bash
cd backend
npm install
```

This installs Express, bcryptjs, jsonwebtoken, cors, dotenv, and `ws`. The database
file (`projects.db`) and all tables are created automatically on first run.

> `.env` already has a default `JWT_SECRET` and `PORT=4002` (different from Tasks 1
> and 2, so all three can run at once without a port clash). If 4002 is somehow taken
> on your machine, just change `PORT` in `.env` to any free number.

## 3. Run the app
```bash
npm start
```
You should see:
```
✅ Server running at http://localhost:4002
✅ WebSocket server ready at /ws (real-time task & comment updates)
```
Open your browser at **http://localhost:4002**.

## 4. Try it out (including the real-time feature)
1. Register two accounts — open a **second browser window (or incognito tab)** for
   the second one, so you're logged in as both people at once.
2. From account A: create a project, then in the board page's "Members" panel, add
   account B by their email.
3. From account A: create a task and assign it to B.
4. Open the same project's board URL in account B's window (copy the `board.html?id=...`
   link, or just click into "My Projects" → the project — since B is now a member,
   it'll show up in their list too).
5. With **both windows open side-by-side**: change a task's status, or add a comment,
   from one window — watch it update instantly in the other window without refreshing.
   The small "Live updates" dot next to the project title turns green once the
   WebSocket connects.

## Project Structure
```
CodeAlpha_ProjectManagementTool/
├── backend/
│   ├── server.js                # Express + HTTP server + WebSocket wiring
│   ├── db.js                    # SQLite schema (node:sqlite)
│   ├── realtime.js              # WebSocket server: auth, project rooms, broadcast
│   ├── .env
│   ├── package.json
│   ├── middleware/auth.js       # JWT auth + project/task membership guards
│   └── routes/
│       ├── auth.js              # register, login
│       ├── projects.js          # create/list/detail, member management
│       ├── projectTasks.js      # list/create tasks within a project
│       └── tasks.js             # get/update/delete a task, its comments
├── frontend/
│   ├── index.html               # Dashboard: your projects + create new
│   ├── board.html                # Kanban board, members panel, task modal
│   ├── login.html / register.html
│   ├── css/style.css
│   └── js/                      # api.js, dashboard.js, board.js, auth.js
└── README.md
```

## API Reference
| Method | Endpoint                                | Auth | Description                        |
|--------|-------------------------------------------|------|-------------------------------------|
| POST   | /api/auth/register                         | No   | Create account, returns JWT         |
| POST   | /api/auth/login                            | No   | Log in, returns JWT                 |
| GET    | /api/projects                              | Yes  | Projects you belong to              |
| POST   | /api/projects                              | Yes  | Create a project (you become owner) |
| GET    | /api/projects/:id                          | Yes* | Project detail + members            |
| POST   | /api/projects/:id/members                  | Yes* | Add a member by email               |
| DELETE | /api/projects/:id/members/:userId          | Yes* | Remove a member (owner) or leave (self) |
| GET    | /api/projects/:id/tasks                    | Yes* | List tasks in the project           |
| POST   | /api/projects/:id/tasks                    | Yes* | Create a task                       |
| GET    | /api/tasks/:id                             | Yes* | Task detail                         |
| PUT    | /api/tasks/:id                             | Yes* | Update title/description/status/assignee |
| DELETE | /api/tasks/:id                             | Yes* | Delete a task                       |
| GET    | /api/tasks/:id/comments                    | Yes* | List comments                       |
| POST   | /api/tasks/:id/comments                    | Yes* | Add a comment                       |

`Yes*` = requires being a member of that task's/project's project, not just logged in.

**WebSocket:** connect to `ws://localhost:4002/ws?project_id=<id>&token=<jwt>` to
receive `task_created`, `task_updated`, `task_deleted`, and `comment_added` events for
that project in real time.

---

## 5. Submission Steps (per CodeAlpha instructions)

1. **Create a GitHub repository** named exactly: `CodeAlpha_ProjectManagementTool`
2. Push this project (run these from the project's root folder — the one containing
   both `backend` and `frontend`, not from inside `backend`):
   ```bash
   git init
   git add .
   git commit -m "Task 3: Project Management Tool - CodeAlpha Internship"
   git branch -M main
   git remote add origin https://github.com/<your-username>/CodeAlpha_ProjectManagementTool.git
   git push -u origin main
   ```
3. **Record a short screen-recording** — this is a great one to show off the real-time
   feature: have two browser windows open side by side, logged in as two different
   users, and show a task update/comment appearing live in both. Upload it to LinkedIn,
   tag **@CodeAlpha**, and include your GitHub repo link in the post.
4. **Submit the task** using the Submission Form shared in your CodeAlpha WhatsApp
   group — paste your GitHub link + LinkedIn post link there.

With Tasks 1, 2, and 3 done, you've completed 3 of the 4 listed tasks — comfortably
above CodeAlpha's minimum of 2–3 for the completion certificate.

## Notes
- To reset the database, stop the server and delete `backend/projects.db` (and its
  `-wal`/`-shm` files).
- A project member can only be assigned a task if they're a member of that project —
  the backend enforces this even if the frontend didn't.
- Only the project owner (or the member themself) can remove a member; the owner
  can't be removed.
