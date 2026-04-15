# PassOP - Your Own Password Manager

![image](https://github.com/user-attachments/assets/ccba746e-9115-4a5d-86ec-9e3421eeee80)

PassOp is a secure and user-friendly password management system built with the MERN stack. It helps users store and manage passwords efficiently, ensuring their data is safe and easily accessible.

---

## Tech Stack
- **Frontend**: React.js, TailwindCSS, React Toastify, Vite
- **Backend**: Node.js, Express.js
- **Database**: MongoDB (accessed via native mongodb driver)
- **Other Tools**: uuid (for unique ID generation on the frontend), lord-icon (for animated icons)

## System Architecture
The application follows a standard Client-Server architecture:
- **Client**: A React Single Page Application (SPA) that provides the user interface for adding, editing, deleting, and copying passwords.
- **Server**: An Express REST API that handles HTTP requests (GET, POST, DELETE) for managing password data.
- **Database**: MongoDB handles the persistent storage of password entries.

## Data Flow
1. **Fetching Passwords**: On load, `Manager.jsx` calls `GET http://localhost:3000/`. The backend Express server queries the MongoDB database and returns all saved passwords as JSON.
2. **Saving Passwords**: When a user fills the form and clicks "Save", a POST request is sent to the backend with the password data. The backend inserts the data into MongoDB.
3. **Deleting Passwords**: When a user deletes a password, a DELETE request is sent with the target `id`. The backend deletes the corresponding document from the database.

## Component Hierarchy
- `App` (Root Component)
  - `Navbar` (Application Navigation)
  - `Manager` (Core Component containing the password form and the list table)
    - Form Inputs (Site, Username, Password)
    - Passwords Table (to display saved items)
  - `Footer` (Application Footer)

## Technical Design Decisions
- **Database Decision (Why MongoDB?)**: Document databases map natively to JavaScript and JSON, keeping the frontend JSON structure identical to backend storage. The native `mongodb` integration provides speed and simplicity for an application of this scale.
- **Decoupled Frontend and Backend**: Maintains a clean separation of concerns, allowing standalone scaling and easier maintenance.
- **Local State Management**: Uses React hooks (`useState`, `useEffect`) directly in the `Manager` component as the state complexity is relatively low.
- **Optimistic UI Updates**: In some cases, the Local State is updated concurrently with network calls, keeping the interface feeling snappy and responsive for the user.

## Database Schema
The MongoDB database contains a single collection named `passwords`.
Each document follows this schema:
- `id` (String): A Unique UUID generated on the frontend.
- `site` (String): The URL or name of the site.
- `username` (String): The user's login ID or username.
- `password` (String): The user's password.

## Folder Structure
```
passwordManager-MERN/
├── backend/                  # Node/Express Backend Application
│   ├── .env                  # Environment Variables for backend
│   ├── server.js             # Main Express application and MongoDB connection
│   └── package.json          # Backend dependencies
├── src/                      # React Frontend Source Code
│   ├── components/           # React Components
│   │   ├── Footer.jsx        # Footer View
│   │   ├── Manager.jsx       # Main Password Manager Logic and View
│   │   └── Navbar.jsx        # Navigation Bar View
│   ├── assets/               # Static assets
│   ├── App.jsx               # Main React Application Component
│   ├── main.jsx              # React Entry Point
│   └── index.css / App.css   # Global Tailwind styles and custom CSS
├── public/                   # Public static files (icons, SVGs)
├── package.json              # Frontend workspace dependencies
├── tailwind.config.js        # Tailwind CSS configuration
└── vite.config.js            # Vite build configuration
```

## Getting Started

### Prerequisites
- Node.js installed
- MongoDB connection string (Atlas or Local)

### 1. Backend Setup
Open a terminal and navigate to the backend directory:
```bash
cd backend
npm install
```
Create a `.env` file in the `backend` directory mapping to your database setup:
```env
MONGO_URI=your_mongodb_connection_string
DB_NAME=your_database_name
```
Start the backend server:
```bash
node server.js
```

### 2. Frontend Setup
Open a new terminal window / tab.
```bash
# From the project root
npm install
npm run dev
```

### 3. Open Application
Navigate to the provided localhost URL (typically `http://localhost:5173/` for Vite). Make sure the backend remains running at `http://localhost:3000/`.

## Environment Variables
The application relies on the following environment variables:

**Backend (`backend/.env`)**
- `MONGO_URI`: The MongoDB connection string used to connect to your database cluster.
- `DB_NAME`: The name of the specific database to be used (e.g., `passop`).

(The frontend requires no specialized API keys for this baseline iteration given it points statically to localhost:3000).

---
**License**
This project is licensed under the MIT License.
