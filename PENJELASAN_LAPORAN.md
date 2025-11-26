# Laporan Sistem Kasir Es Teh

## 1. Pendahuluan

### 1.1 Latar Belakang
Sistem kasir es teh merupakan aplikasi web yang dikembangkan untuk membantu pengelolaan bisnis minuman es teh. Sistem ini dirancang untuk mempermudah proses transaksi penjualan, manajemen menu, dan pelaporan keuangan dengan menggunakan teknologi web modern.

### 1.2 Tujuan Pengembangan
- Mengotomatisasi proses transaksi penjualan
- Menyediakan sistem manajemen menu yang mudah digunakan
- Memberikan laporan penjualan dan keuangan real-time
- Meningkatkan efisiensi operasional bisnis es teh

### 1.3 Ruang Lingkup
Sistem ini mencakup:
- Autentikasi pengguna
- Dashboard dengan statistik penjualan
- Manajemen menu dan kategori
- Proses transaksi penjualan
- Sistem pembayaran (cash dan QR code)
- Laporan harian dan keuangan

## 2. Analisis Sistem

### 2.1 Arsitektur Aplikasi
Sistem menggunakan arsitektur **Client-Server** dengan pemisahan yang jelas:

**Frontend (Client):**
- Framework: React.js 19.2.0
- Build Tool: Vite
- Styling: Tailwind CSS
- HTTP Client: Axios
- Routing: React Router DOM

**Backend (Server):**
- Runtime: Node.js
- Framework: Express.js 5.1.0
- Database: MySQL dengan driver MySQL2
- Authentication: JWT (JSON Web Token)
- Security: bcryptjs untuk hashing password

**Database:**
- DBMS: MySQL/MariaDB
- Connection: Pool connection untuk optimasi

### 2.2 Struktur Database
```sql
-- Tabel utama sistem:
pengguna (id_user, username, password)
series (id_series, nama_series)
menu (id_menu, id_series, nama_menu, harga, stok, gambar)
transaksi (id_transaksi, tanggal, total_harga, metode_pembayaran)
detail_transaksi (id_detail, id_transaksi, id_menu, jumlah, harga_satuan)
```

## 3. Implementasi Fitur

### 3.1 Sistem Autentikasi
**Frontend:**
```jsx
const handleLogin = async () => {
  const res = await api.post("/auth/login", { username, password });
  localStorage.setItem("token", res.data.token);
  navigate("/dashboard");
};
```

**Backend:**
```javascript
const token = jwt.sign(
  { id_user: user.id_user, username: user.username },
  process.env.JWT_SECRET,
  { expiresIn: "1d" }
);
```

**Fitur:**
- Login dengan username/password
- JWT token untuk session management
- Password hashing dengan bcryptjs
- Auto-redirect setelah login berhasil

### 3.2 Dashboard dan Navigasi
**Komponen Utama:**
- Statistik real-time (transaksi, pendapatan, item terjual)
- Menu navigasi berbasis card dengan icon
- Refresh data otomatis
- Logout functionality

**State Management:**
```jsx
const [stats, setStats] = useState({
  total_transaksi: 0,
  total_pendapatan: 0,
  total_item: 0
});
```

### 3.3 Manajemen Menu
**Backend Controller:**
```javascript
addMenu: (req, res) => {
  const { id_series, nama_menu, harga, stok } = req.body;
  const gambar = req.file?.filename || null;
  
  db.query(
    "INSERT INTO menu (id_series, nama_menu, harga, stok, gambar) VALUES (?,?,?,?,?)",
    [id_series, nama_menu, harga, stok, gambar]
  );
}
```

**Fitur:**
- CRUD operations untuk menu
- Upload gambar dengan Multer
- Kategorisasi menu berdasarkan series
- Validasi input data

### 3.4 API Integration
**Axios Configuration:**
```jsx
const api = axios.create({
  baseURL: "http://localhost:3001",
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

## 4. Teknologi dan Tools

### 4.1 Frontend Dependencies
```json
{
  "react": "^19.2.0",
  "react-router-dom": "^7.9.6",
  "axios": "^1.13.2",
  "@tailwindcss/vite": "^4.1.17"
}
```

### 4.2 Backend Dependencies
```json
{
  "express": "^5.1.0",
  "mysql2": "^3.15.3",
  "jsonwebtoken": "^9.0.2",
  "bcryptjs": "^3.0.3",
  "multer": "^2.0.2"
}
```

## 5. Keunggulan Sistem

### 5.1 User Experience
- **Modern UI Design**: Menggunakan glassmorphism dan gradient
- **Responsive Layout**: Adaptif untuk berbagai ukuran layar
- **Interactive Elements**: Hover animations dan smooth transitions
- **Intuitive Navigation**: Card-based menu dengan icon yang jelas

### 5.2 Security Features
- **JWT Authentication**: Token-based security
- **Password Hashing**: bcryptjs untuk keamanan password
- **Protected Routes**: Middleware untuk route protection
- **Input Validation**: Client dan server-side validation

### 5.3 Performance Optimization
- **Connection Pooling**: Optimasi koneksi database
- **React Hooks**: Efficient state management
- **Axios Interceptors**: Centralized request/response handling
- **Component Reusability**: Modular component architecture

## 6. Struktur File dan Organisasi

### 6.1 Frontend Structure
```
frontend/src/
├── components/     # Komponen reusable
├── pages/         # Halaman aplikasi
├── services/      # API integration
└── App.jsx        # Main component
```

### 6.2 Backend Structure
```
backend/src/
├── config/        # Database configuration
├── controllers/   # Business logic
├── middleware/    # Authentication middleware
├── routes/        # API endpoints
└── app.js         # Server configuration
```

## 7. Fitur yang Telah Diimplementasi

### 7.1 Completed Features ✅
- **Authentication System**: Login dengan JWT
- **Dashboard**: Statistik dan navigasi
- **Menu Management**: CRUD menu dengan upload gambar
- **Database Integration**: MySQL dengan connection pooling
- **API Layer**: RESTful API dengan Express.js
- **Routing System**: Multi-page navigation dengan React Router

### 7.2 Features in Development 🔄
- **Sales Transaction**: Interface transaksi penjualan
- **Checkout Process**: Sistem checkout dan pembayaran
- **Payment Methods**: QR Code dan Cash payment
- **Daily Reports**: Laporan penjualan harian
- **Financial Management**: Manajemen keuangan dan pengeluaran

## 8. Kesimpulan

### 8.1 Pencapaian
Sistem kasir es teh telah berhasil diimplementasi dengan arsitektur modern menggunakan React.js dan Node.js. Fitur-fitur dasar seperti autentikasi, dashboard, dan manajemen menu telah berfungsi dengan baik.

### 8.2 Kelebihan Sistem
- **Scalable Architecture**: Mudah dikembangkan dan dipelihara
- **Modern Technology Stack**: Menggunakan teknologi terkini
- **User-Friendly Interface**: Design yang intuitif dan menarik
- **Security Implementation**: Keamanan yang memadai dengan JWT dan hashing

### 8.3 Rekomendasi Pengembangan
- Implementasi fitur transaksi dan pembayaran
- Penambahan sistem inventory management
- Integrasi dengan payment gateway
- Implementasi real-time notifications
- Penambahan fitur backup dan restore data

### 8.4 Manfaat Bisnis
- **Efisiensi Operasional**: Otomatisasi proses manual
- **Akurasi Data**: Mengurangi human error
- **Real-time Monitoring**: Statistik dan laporan real-time
- **Scalability**: Dapat dikembangkan sesuai kebutuhan bisnis

---

*Sistem kasir es teh ini merupakan solusi modern untuk pengelolaan bisnis minuman dengan teknologi web terkini, memberikan foundation yang solid untuk pengembangan fitur-fitur lanjutan.*