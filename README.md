# JWT_Auth

---

### **Project Structure**
1. **Backend**: Node.js + Express.js for handling API requests, user authentication, and JWT management.
2. **Frontend**: Next.js + React.js for the UI, with Tailwind CSS for styling.
3. **Authentication**: JWT (JSON Web Tokens) for user authentication.

---

### **Step 1: Backend Setup**
#### 1. Initialize the Backend Project
```bash
mkdir backend
cd backend
npm init -y
npm install express jsonwebtoken bcryptjs cors dotenv
```

#### 2. Create the Backend Structure
- `app.js`: Main entry point for the backend.
- `routes/auth.js`: Handles authentication routes (login, register).
- `middleware/authMiddleware.js`: Middleware to verify JWT tokens.
- `.env`: Store environment variables (e.g., JWT secret).

#### 3. Implement the Backend Code
**`app.js`**
```javascript
const express = require('express');
const cors = require('cors');
const authRoutes = require('./routes/auth');
const dotenv = require('dotenv');

dotenv.config();
const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(express.json());

// Routes
app.use('/api/auth', authRoutes);

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**`routes/auth.js`**
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const router = express.Router();

// Mock database
const users = [];

// Register Route
router.post('/register', async (req, res) => {
  const { username, password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  users.push({ username, password: hashedPassword });
  res.status(201).send('User registered');
});

// Login Route
router.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = users.find(u => u.username === username);
  if (!user) return res.status(400).send('User not found');

  const validPassword = await bcrypt.compare(password, user.password);
  if (!validPassword) return res.status(400).send('Invalid password');

  const token = jwt.sign({ username: user.username }, process.env.JWT_SECRET, { expiresIn: '1h' });
  res.json({ token });
});

module.exports = router;
```

**`middleware/authMiddleware.js`**
```javascript
const jwt = require('jsonwebtoken');

module.exports = (req, res, next) => {
  const token = req.header('Authorization')?.replace('Bearer ', '');
  if (!token) return res.status(401).send('Access denied');

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(400).send('Invalid token');
  }
};
```

**`.env`**
```
JWT_SECRET=your_jwt_secret_key
```

---

### **Step 2: Frontend Setup**
#### 1. Initialize the Frontend Project
```bash
npx create-next-app@latest frontend
cd frontend
npm install axios
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

#### 2. Configure Tailwind CSS
Update `tailwind.config.js`:
```javascript
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

Add Tailwind to `styles/globals.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

#### 3. Create Frontend Pages and Components
- `pages/index.js`: Home page.
- `pages/login.js`: Login page.
- `pages/register.js`: Register page.
- `components/Navbar.js`: Navigation bar.

**`pages/index.js`**
```javascript
import Navbar from '../components/Navbar';

export default function Home() {
  return (
    <div>
      <Navbar />
      <h1 className="text-3xl font-bold text-center mt-10">Welcome to the Home Page</h1>
    </div>
  );
}
```

**`pages/login.js`**
```javascript
import { useState } from 'react';
import axios from 'axios';
import { useRouter } from 'next/router';

export default function Login() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const router = useRouter();

  const handleLogin = async () => {
    try {
      const res = await axios.post('http://localhost:5000/api/auth/login', { username, password });
      localStorage.setItem('token', res.data.token);
      router.push('/');
    } catch (err) {
      console.error(err);
    }
  };

  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h1 className="text-2xl font-bold mb-4">Login</h1>
      <input
        type="text"
        placeholder="Username"
        className="p-2 border mb-2"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
      />
      <input
        type="password"
        placeholder="Password"
        className="p-2 border mb-2"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button onClick={handleLogin} className="p-2 bg-blue-500 text-white">
        Login
      </button>
    </div>
  );
}
```

**`pages/register.js`**
```javascript
import { useState } from 'react';
import axios from 'axios';
import { useRouter } from 'next/router';

export default function Register() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const router = useRouter();

  const handleRegister = async () => {
    try {
      await axios.post('http://localhost:5000/api/auth/register', { username, password });
      router.push('/login');
    } catch (err) {
      console.error(err);
    }
  };

  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h1 className="text-2xl font-bold mb-4">Register</h1>
      <input
        type="text"
        placeholder="Username"
        className="p-2 border mb-2"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
      />
      <input
        type="password"
        placeholder="Password"
        className="p-2 border mb-2"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button onClick={handleRegister} className="p-2 bg-green-500 text-white">
        Register
      </button>
    </div>
  );
}
```

**`components/Navbar.js`**
```javascript
import Link from 'next/link';

export default function Navbar() {
  return (
    <nav className="bg-gray-800 p-4">
      <div className="container mx-auto flex justify-between">
        <Link href="/" className="text-white text-lg font-bold">Home</Link>
        <div className="space-x-4">
          <Link href="/login" className="text-white">Login</Link>
          <Link href="/register" className="text-white">Register</Link>
        </div>
      </div>
    </nav>
  );
}
```

---

### **Step 3: Run the Project**
1. Start the backend:
   ```bash
   cd backend
   node app.js
   ```
2. Start the frontend:
   ```bash
   cd frontend
   npm run dev
   ```

---

### **Step 4: Test the Application**
- Register a user at `/register`.
- Login at `/login` and receive a JWT.
- Use the JWT to authenticate requests to protected routes (if added).

---
