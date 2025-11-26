# Dokumentasi Sistem Kasir Es Teh

## 1. Arsitektur Sistem

### 1.1 Gambaran Umum Arsitektur
Sistem kasir es teh menggunakan arsitektur **Client-Server** dengan pemisahan yang jelas antara frontend dan backend:

```
┌─────────────────┐    HTTP/REST API    ┌─────────────────┐    SQL Queries    ┌─────────────────┐
│   Frontend      │ ◄─────────────────► │    Backend      │ ◄───────────────► │    Database     │
│   (React.js)    │                     │   (Node.js)     │                   │    (MySQL)      │
└─────────────────┘                     └─────────────────┘                   └─────────────────┘
```

### 1.2 Komponen Utama Sistem

#### A. Frontend (Client-Side)
- **Framework**: React.js dengan Vite
- **Routing**: React Router DOM
- **Styling**: Tailwind CSS + Inline Styles
- **HTTP Client**: Axios
- **State Management**: React Hooks (useState, useEffect)

#### B. Backend (Server-Side)
- **Runtime**: Node.js
- **Framework**: Express.js
- **Authentication**: JWT (JSON Web Token)
- **Password Hashing**: bcryptjs
- **File Upload**: Multer
- **Database Driver**: MySQL2

#### C. Database
- **DBMS**: MySQL/MariaDB
- **Connection**: Connection Pool untuk optimasi performa

### 1.3 Struktur Direktori

```
kasir-es-teh/
├── frontend/                 # Aplikasi React
│   ├── src/
│   │   ├── components/       # Komponen reusable
│   │   ├── pages/           # Halaman aplikasi
│   │   ├── services/        # API services
│   │   └── App.jsx          # Main app component
│   └── package.json
├── backend/                  # Server Node.js
│   ├── src/
│   │   ├── config/          # Konfigurasi database
│   │   ├── controllers/     # Business logic
│   │   ├── middleware/      # Authentication middleware
│   │   ├── routes/          # API endpoints
│   │   └── app.js           # Main server file
│   └── package.json
└── sample_data.sql          # Data sample database
```

### 1.4 Alur Data dan Komunikasi

```
User Interface → API Request → Route Handler → Controller → Database → Response → UI Update
```

1. **User Interaction**: User berinteraksi dengan UI React
2. **API Call**: Frontend mengirim HTTP request ke backend
3. **Authentication**: Middleware memverifikasi JWT token
4. **Route Processing**: Express router mengarahkan ke controller
5. **Business Logic**: Controller memproses request dan query database
6. **Database Operation**: MySQL menjalankan query dan mengembalikan data
7. **Response**: Backend mengirim response JSON ke frontend
8. **UI Update**: React memperbarui state dan re-render komponen

## 2. Implementasi Fitur Frontend

### 2.1 Sistem Autentikasi

#### Login Component (`Login.jsx`)
```jsx
// Fitur utama:
- Form login dengan username/password
- Password visibility toggle
- Error handling
- JWT token storage
- Redirect setelah login berhasil

// State management:
const [username, setUsername] = useState("");
const [password, setPassword] = useState("");
const [error, setError] = useState("");
const [showPassword, setShowPassword] = useState(false);

// API integration:
const handleLogin = async () => {
  const res = await api.post("/auth/login", { username, password });
  localStorage.setItem("token", res.data.token);
  navigate("/dashboard");
};
```

### 2.2 Dashboard

#### Dashboard Component (`Dashboard.jsx`)
```jsx
// Fitur utama:
- Menu navigasi ke semua fitur
- Statistik real-time (transaksi, pendapatan, item terjual)
- Logout functionality
- Responsive card layout

// State untuk statistik:
const [stats, setStats] = useState({
  total_transaksi: 0,
  total_pendapatan: 0,
  total_item: 0
});

// Menu items dengan routing:
const menuItems = [
  { title: "Penjualan", path: "/penjualan", icon: MdShoppingCart },
  { title: "Kelola Menu", path: "/kelola-menu", icon: MdRestaurantMenu },
  { title: "Laporan Harian", path: "/laporan-harian", icon: MdBarChart },
  { title: "Keuangan", path: "/keuangan", icon: MdAttachMoney }
];
```

### 2.3 Sistem Routing

#### App Component (`App.jsx`)
```jsx
// Routing configuration:
<Routes>
  <Route path="/" element={<Login />} />
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/penjualan" element={<Penjualan />} />
  <Route path="/checkout" element={<Checkout />} />
  <Route path="/qr-payment" element={<QRPayment />} />
  <Route path="/cash-payment" element={<CashPayment />} />
  <Route path="/kelola-menu" element={<KelolaMenu />} />
  <Route path="/laporan-harian" element={<LaporanHarian />} />
  <Route path="/keuangan" element={<Keuangan />} />
</Routes>
```

### 2.4 API Service Layer

#### API Configuration (`api.js`)
```jsx
// Axios instance dengan base URL:
const api = axios.create({
  baseURL: "http://localhost:3001",
});

// Auto token injection:
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

### 2.5 Komponen Reusable

#### Components Directory
- **MenuCard.jsx**: Card untuk menampilkan item menu
- **CounterButton.jsx**: Button untuk increment/decrement quantity
- **CategoryDropdown.jsx**: Dropdown untuk filter kategori
- **Navbar.jsx**: Navigation bar component
- **ProtectedRoute.jsx**: Route protection dengan authentication

## 3. Implementasi Fitur Backend

### 3.1 Server Configuration

#### Main Server (`app.js`)
```javascript
// Express server setup:
const express = require("express");
const cors = require("cors");
const app = express();

// Middleware:
app.use(cors());
app.use(express.json());
app.use('/uploads', express.static(__dirname + '/uploads'));

// Routes:
app.use("/auth", require("./routes/auth.routes"));
app.use("/menu", require("./routes/menu.routes"));
app.use("/transaksi", require("./routes/transaksi.routes"));
app.use("/laporan", require("./routes/laporan.routes"));
app.use("/keuangan", require("./routes/keuangan.routes"));
```

### 3.2 Database Configuration

#### Database Connection (`db.js`)
```javascript
// MySQL connection pool:
const db = mysql.createPool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});
```

### 3.3 Authentication System

#### Auth Controller (`auth.controller.js`)
```javascript
// Login functionality:
exports.login = (req, res) => {
  const { username, password } = req.body;
  
  // Database query untuk user:
  db.query("SELECT * FROM pengguna WHERE username = ?", [username], (err, results) => {
    if (results.length === 0) {
      return res.status(401).json({ message: "Username tidak ditemukan" });
    }
    
    // Password verification:
    bcrypt.compare(password, user.password, (err, isMatch) => {
      if (!isMatch) {
        return res.status(401).json({ message: "Password salah" });
      }
      
      // JWT token generation:
      const token = jwt.sign(
        { id_user: user.id_user, username: user.username },
        process.env.JWT_SECRET,
        { expiresIn: "1d" }
      );
      
      return res.json({ message: "Login berhasil", token });
    });
  });
};
```

### 3.4 Menu Management

#### Menu Controller (`menu.controller.js`)
```javascript
// Get all menu items:
getMenu: (req, res) => {
  db.query("SELECT * FROM menu", (err, result) => {
    if (err) throw err;
    res.json(result);
  });
},

// Add new menu item:
addMenu: (req, res) => {
  const { id_series, nama_menu, harga, stok } = req.body;
  const gambar = req.file?.filename || null;
  
  db.query(
    "INSERT INTO menu (id_series, nama_menu, harga, stok, gambar) VALUES (?,?,?,?,?)",
    [id_series, nama_menu, harga, stok, gambar],
    (err) => {
      if (err) throw err;
      res.json({ msg: "Menu berhasil ditambahkan" });
    }
  );
}
```

### 3.5 Middleware Authentication

#### Auth Middleware (`auth.js`)
```javascript
// JWT token verification:
const verifyToken = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ message: "Token tidak ditemukan" });
  }
  
  jwt.verify(token, process.env.JWT_SECRET, (err, decoded) => {
    if (err) {
      return res.status(401).json({ message: "Token tidak valid" });
    }
    req.user = decoded;
    next();
  });
};
```

### 3.6 API Routes Structure

#### Route Organization
```javascript
// auth.routes.js
router.post('/login', authController.login);

// menu.routes.js
router.get('/', menuController.getMenu);
router.post('/', upload.single('gambar'), menuController.addMenu);

// transaksi.routes.js
router.get('/', verifyToken, transaksiController.getTransaksi);
router.post('/', verifyToken, transaksiController.createTransaksi);

// laporan.routes.js
router.get('/harian', verifyToken, laporanController.getLaporanHarian);

// keuangan.routes.js
router.get('/', verifyToken, keuanganController.getKeuangan);
router.post('/', verifyToken, keuanganController.addPengeluaran);
```

## 4. Database Schema

### 4.1 Tabel Utama

```sql
-- Tabel pengguna untuk authentication
CREATE TABLE pengguna (
  id_user INT PRIMARY KEY AUTO_INCREMENT,
  username VARCHAR(50) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL
);

-- Tabel series/kategori menu
CREATE TABLE series (
  id_series INT PRIMARY KEY AUTO_INCREMENT,
  nama_series VARCHAR(100) NOT NULL
);

-- Tabel menu items
CREATE TABLE menu (
  id_menu INT PRIMARY KEY AUTO_INCREMENT,
  id_series INT,
  nama_menu VARCHAR(100) NOT NULL,
  harga DECIMAL(10,2) NOT NULL,
  stok INT DEFAULT 0,
  gambar VARCHAR(255),
  FOREIGN KEY (id_series) REFERENCES series(id_series)
);

-- Tabel transaksi
CREATE TABLE transaksi (
  id_transaksi INT PRIMARY KEY AUTO_INCREMENT,
  tanggal DATETIME DEFAULT CURRENT_TIMESTAMP,
  total_harga DECIMAL(10,2) NOT NULL,
  metode_pembayaran ENUM('cash', 'qr') NOT NULL
);

-- Tabel detail transaksi
CREATE TABLE detail_transaksi (
  id_detail INT PRIMARY KEY AUTO_INCREMENT,
  id_transaksi INT,
  id_menu INT,
  jumlah INT NOT NULL,
  harga_satuan DECIMAL(10,2) NOT NULL,
  FOREIGN KEY (id_transaksi) REFERENCES transaksi(id_transaksi),
  FOREIGN KEY (id_menu) REFERENCES menu(id_menu)
);
```

## 5. Fitur-Fitur Sistem

### 5.1 Fitur yang Sudah Diimplementasi
- ✅ **Authentication**: Login dengan JWT
- ✅ **Dashboard**: Statistik dan navigasi
- ✅ **Menu Management**: CRUD menu items
- ✅ **File Upload**: Upload gambar menu
- ✅ **Routing**: Multi-page navigation

### 5.2 Fitur yang Perlu Dikembangkan
- 🔄 **Penjualan**: Interface untuk transaksi
- 🔄 **Checkout**: Proses pembayaran
- 🔄 **Payment Methods**: QR Code dan Cash
- 🔄 **Laporan**: Laporan harian dan keuangan
- 🔄 **Inventory**: Manajemen stok

## 6. Teknologi dan Dependencies

### 6.1 Frontend Dependencies
```json
{
  "react": "^19.2.0",
  "react-dom": "^19.2.0",
  "react-router-dom": "^7.9.6",
  "axios": "^1.13.2",
  "@tailwindcss/vite": "^4.1.17"
}
```

### 6.2 Backend Dependencies
```json
{
  "express": "^5.1.0",
  "mysql2": "^3.15.3",
  "jsonwebtoken": "^9.0.2",
  "bcryptjs": "^3.0.3",
  "cors": "^2.8.5",
  "multer": "^2.0.2",
  "dotenv": "^17.2.3"
}
```

## 7. Keamanan

### 7.1 Implementasi Keamanan
- **Password Hashing**: bcryptjs untuk hash password
- **JWT Authentication**: Token-based authentication
- **CORS**: Cross-origin resource sharing protection
- **Input Validation**: Server-side validation
- **File Upload Security**: Multer dengan file type validation

### 7.2 Environment Variables
```env
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASS=
DB_NAME=kasir_es_teh
JWT_SECRET=your_jwt_secret_key
```

## 8. Deployment dan Development

### 8.1 Development Setup
```bash
# Backend
cd backend
npm install
npm run dev

# Frontend
cd frontend
npm install
npm run dev
```

### 8.2 Production Considerations
- Environment configuration
- Database optimization
- Static file serving
- Error logging
- Performance monitoring
- Backup strategy

---

*Dokumentasi ini memberikan gambaran lengkap tentang arsitektur dan implementasi sistem kasir es teh. Sistem ini dirancang dengan prinsip separation of concerns dan scalability untuk memudahkan maintenance dan pengembangan fitur baru.*