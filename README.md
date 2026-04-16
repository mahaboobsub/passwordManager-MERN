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

## UI Layout & Component Architecture
The user interface is built as a responsive Single Page Application (SPA) leveraging **React** and **TailwindCSS**, utilizing local component state for snappy interactions.
- **Navbar**: Main top navigation for the application.
- **Manager Section (`Manager.jsx`)**: The core centralized component containing state logic (`form`, `passwordArray`) and UI.
  - **Background Styling**: Uses dynamic Tailwind utility classes to create a green gradient grid layout `bg-[linear-gradient(...)]` with a blurred circular backdrop `blur-[100px]`.
  - **Password Form**: Three primary controlled inputs (`site`, `username`, `password`) bound to a React `useState` object (`form`). Features a dynamic password visibility toggle utilizing `useRef` for direct DOM manipulation.
  - **Save Button**: An action button featuring a Lord-icon hover animation. Validates input length (>3 chars) before submission.
  - **Password Table**: A responsive tabular display mapping over the `passwordArray` state. It includes interactive elements to copy data (using `navigator.clipboard.writeText`) coupled with **React Toastify** for visual feedback. Contains Edit and Delete action buttons rendering animated Lord-icons.
- **Footer**: Located at the bottom of the page.

## Full-Stack API Flows (Frontend to Backend)
The data flow relies heavily on **Optimistic UI Updates**—meaning the frontend state updates immediately prior to or alongside the asynchronous backend request, giving the user an instant, snappy experience.

- **Read (Fetching Data)**: 
  - On initial mount, `Manager.jsx` triggers a `useEffect` hook with an empty dependency array.
  - It calls `fetch("http://localhost:3000/")`.
  - The Express backend handles the `GET /` route, queries MongoDB using `collection.find({}).toArray()`, and returns a JSON payload.
  - Frontend parses JSON and updates the `passwordArray` state, rendering the table.
- **Create (Saving Data)**: 
  - User fills out the form and clicks "Save".
  - Frontend generates a unique ID using `uuidv4()` and immediately appends the new record to `passwordArray` using state spread syntax (`[...passwordArray, newItem]`).
  - Simultaneously, a `fetch` `POST` request is sent to the backend with the new payload.
  - The Node server receives the request payload, parses it using `body-parser`, and stores it via `collection.insertOne()`.
- **Update (Editing Data)**: 
  - Clicking "Edit" populates the `form` state with the selected entry's details and immediately removes that entry from the visible `passwordArray`.
  - The user modifies the details in the form.
  - Upon clicking "Save", the frontend logic understands this as essentially a "new" record. To prevent database duplicates, the frontend sequentially executes a `DELETE` API call utilizing the old `uuid`, followed by a `POST` request sending a newly generated `uuid` for the modified data.
- **Delete**: 
  - Clicking "Delete" prompts a native browser `confirm()` dialog.
  - If approved, the frontend optimistically filters the entry out of `passwordArray`.
  - A `DELETE` request is sent to the backend with the corresponding `id`.
  - Backend executes `collection.deleteOne({ id })` in MongoDB.

## API Endpoints Reference
The Express backend provides RESTful endpoints mounted on `http://localhost:3000` connected to MongoDB via the native `mongodb` driver.

- **`GET /`**
  - **Description**: Retrieves all stored password records.
  - **Response**: Array of JSON objects `[{ _id, id, site, username, password }, ...]`.
- **`POST /`**
  - **Description**: Stores a newly created or updated password entry.
  - **Headers**: `"Content-Type": "application/json"`
  - **Body**: JSON object containing `id` (uuid), `site`, `username`, and `password`.
  - **Response**: `{ success: true, result: { acknowledged: true, insertedId: ... } }`
- **`DELETE /`**
  - **Description**: Removes an existing password entry based on frontend UUID.
  - **Headers**: `"Content-Type": "application/json"`
  - **Body**: JSON object containing the `id`.
  - **Response**: `{ success: true, result: { acknowledged: true, deletedCount: 1 } }`

## Database Connection & Schema Model
The application utilizes a local or Atlas-hosted **MongoDB** cluster.
- **Connection**: Managed via `MongoClient` natively connecting through a `MONGO_URI` loaded from `.env`.
- **Database Selection**: Defaults to the `DB_NAME` environment variable.
- **Collection**: Data is stored inside a single collection named `passwords`.

Though MongoDB is schema-less, the document structure implicitly enforced by the frontend is:
- `_id` *(ObjectId)*: Automatically generated by MongoDB upon insertion.
- `id` *(String)*: A universally unique identifier (UUID v4) generated client-side. This is the primary key used for frontend lookups, React loop keys, and targeted CRUD operations.
- `site` *(String)*: Target website URL or service domain.
- `username` *(String)*: Linked account identifier/email.
- `password` *(String)*: Plain-text secret credential.

## Deep-dive Technical FAQs
**Q1: Why use `useRef` for toggling password visibility instead of `useState`?**
**Answer**: In `Manager.jsx`, `useRef` directly accesses the DOM nodes for the input field type (`passwordRef.current.type`) and the eye icon image source (`ref.current.src`). Bypassing `useState` for this specific visual-only action prevents unnecessary React component re-renders, keeping front-end performance optimal.

**Q2: Why generate a UUID (`uuidv4()`) on the frontend when MongoDB generates its own `_id`?**
**Answer**: Generating unique IDs directly from the frontend enables True Optimistic UI Updates. The React state can natively append new records to the local list with a guaranteed unique key for iteration without waiting for an asynchronous server round-trip to retrieve a newly minted Mongo `_id`. 

**Q3: Is the "Edit" functionally handling data efficiently?**
**Answer**: The edit function utilizes a "Delete and Re-Create" mechanism. It clears the old record with the target UUID via DELETE and sends a completely new record with a new UUID via POST. While seamless for this architecture, a traditional RESTful system should implement a dedicated `PUT` or `PATCH` endpoint to execute `collection.updateOne()` inside MongoDB to reduce processing overhead and maintain consistency.

**Q4: How does the application implement the "Copy to Clipboard" functionality?**
**Answer**: It leverages the native Web API `navigator.clipboard.writeText(text)` inside an `onClick` handler. Once triggered, it simultaneously invokes a stylized `toast()` notification from the `react-toastify` library to give the user immediate visual success feedback.

**Q5: Are the passwords stored securely in the database?**
**Answer**: **No.** This project is fundamentally an educational MVP. Passwords are sent over HTTP in plaintext and stored as raw string fields in MongoDB. In a production environment, this is highly insecure. A real-world solution must implement strong one-way hashing (like `bcrypt`) if acting as an authenticator, or robust bidirectional encryption (like `AES-256`) if acting as a centralized password vault where the user retains the master decryption key.

**Q6: How does the app bypass Cross-Origin Resource Sharing (CORS) errors?**
**Answer**: The React Vite server operates on port `5173`, while the Express API runs on port `3000`. Web browsers inherently block cross-origin requests. This is resolved by importing the `cors` npm package in `server.js` and applying it as top-level Express middleware (`app.use(cors())`), signaling to the browser that the Node backend safely accepts cross-origin API payloads from the Vite frontend.

## Component Hierarchy & Technical Decisions
- **Database Decision (Why MongoDB?)**: Document databases map natively to JavaScript and JSON, keeping the frontend JSON structure identical to backend storage.
- **Decoupled Architecture**: Maintains clean separation.
- **Local State & Overlapping Networks**: React state is modified optimistically alongside non-blocking Fetch calls.

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
