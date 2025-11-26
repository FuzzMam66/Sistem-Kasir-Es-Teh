# Implementasi Sistem Kasir Es Teh - Detail Lengkap

## 1. IMPLEMENTASI FRONTEND (React.js)

### 1.1 Struktur Aplikasi React

#### App Component (Main Router)
```jsx
import { Routes, Route } from "react-router-dom";
import Login from "./pages/Login";
import Dashboard from "./pages/Dashboard";
import Penjualan from "./pages/Penjualan";

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<Login />} />
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/penjualan" element={<Penjualan />} />
      <Route path="/checkout" element={<Checkout />} />
      <Route path="/kelola-menu" element={<KelolaMenu />} />
      <Route path="/laporan-harian" element={<LaporanHarian />} />
      <Route path="/keuangan" element={<Keuangan />} />
    </Routes>
  );
}
```

### 1.2 Sistem Autentikasi Frontend

#### Login Component
```jsx
import { useState } from "react";
import api from "../services/api";
import { useNavigate } from "react-router-dom";

function Login() {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [showPassword, setShowPassword] = useState(false);
  const navigate = useNavigate();

  const handleLogin = async () => {
    try {
      const res = await api.post("/auth/login", { username, password });
      localStorage.setItem("token", res.data.token);
      navigate("/dashboard");
    } catch (err) {
      setError(err.response?.data?.message || "Login gagal");
    }
  };

  return (
    <div className="login-container">
      <form onSubmit={(e) => { e.preventDefault(); handleLogin(); }}>
        <input
          type="text"
          placeholder="Username"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
        />
        <input
          type={showPassword ? "text" : "password"}
          placeholder="Password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
        <button type="submit">Login</button>
        {error && <div className="error">{error}</div>}
      </form>
    </div>
  );
}
```

### 1.3 Dashboard dengan State Management

#### Dashboard Component
```jsx
import { useState, useEffect } from "react";
import api from "../services/api";

function Dashboard() {
  const [stats, setStats] = useState({
    total_transaksi: 0,
    total_pendapatan: 0,
    total_item: 0
  });

  const fetchStats = async () => {
    try {
      const res = await api.get("/dashboard/stats");
      if (res.data.status === "success") {
        setStats(res.data.data);
      }
    } catch (err) {
      console.error("Error fetching stats:", err);
    }
  };

  useEffect(() => {
    fetchStats();
  }, []);

  const menuItems = [
    {
      title: "Penjualan",
      path: "/penjualan",
      icon: "🛒",
      color: "bg-blue-500"
    },
    {
      title: "Kelola Menu",
      path: "/kelola-menu",
      icon: "📋",
      color: "bg-green-500"
    }
  ];

  return (
    <div className="dashboard">
      <div className="stats-grid">
        <div className="stat-card">
          <h3>Transaksi Hari Ini</h3>
          <p>{stats.total_transaksi}</p>
        </div>
        <div className="stat-card">
          <h3>Total Pendapatan</h3>
          <p>Rp {stats.total_pendapatan.toLocaleString()}</p>
        </div>
        <div className="stat-card">
          <h3>Item Terjual</h3>
          <p>{stats.total_item}</p>
        </div>
      </div>
      
      <div className="menu-grid">
        {menuItems.map((item, index) => (
          <div key={index} className="menu-card" onClick={() => navigate(item.path)}>
            <div className="icon">{item.icon}</div>
            <h3>{item.title}</h3>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 1.4 API Service Layer

#### API Configuration
```jsx
import axios from "axios";

const api = axios.create({
  baseURL: "http://localhost:5000",
});

// Auto inject JWT token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Handle 401 responses
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem("token");
      window.location.href = "/";
    }
    return Promise.reject(error);
  }
);

export default api;
```

### 1.5 Komponen Reusable

#### MenuCard Component
```jsx
const MenuCard = ({ menu, onAddToCart, quantity, onQuantityChange }) => {
  return (
    <div className="menu-card">
      <img src={`/uploads/${menu.gambar}`} alt={menu.nama_menu} />
      <h3>{menu.nama_menu}</h3>
      <p className="price">Rp {menu.harga.toLocaleString()}</p>
      <p className="stock">Stok: {menu.stok}</p>
      
      <div className="quantity-controls">
        <button onClick={() => onQuantityChange(quantity - 1)}>-</button>
        <span>{quantity}</span>
        <button onClick={() => onQuantityChange(quantity + 1)}>+</button>
      </div>
      
      <button 
        className="add-to-cart"
        onClick={() => onAddToCart(menu)}
        disabled={menu.stok === 0}
      >
        Tambah ke Keranjang
      </button>
    </div>
  );
};
```

## 2. IMPLEMENTASI BACKEND (Node.js + Express)

### 2.1 Server Configuration

#### Main App (app.js)
```javascript
require("dotenv").config();
const express = require("express");
const cors = require("cors");
const app = express();

// Middleware
app.use(cors());
app.use(express.json());
app.use('/uploads', express.static(__dirname + '/uploads'));

// Routes
app.use("/auth", require("./routes/auth.routes"));
app.use("/menu", require("./routes/menu.routes"));
app.use("/transaksi", require("./routes/transaksi.routes"));
app.use("/laporan", require("./routes/laporan.routes"));
app.use("/keuangan", require("./routes/keuangan.routes"));

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

### 2.2 Database Configuration

#### Database Connection (config/db.js)
```javascript
const mysql = require("mysql2");

const db = mysql.createPool({
  host: process.env.DB_HOST || 'localhost',
  port: process.env.DB_PORT || 3306,
  user: process.env.DB_USER || 'root',
  password: process.env.DB_PASS || '',
  database: process.env.DB_NAME || 'kasir_es_teh',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

// Test connection
db.getConnection((err, connection) => {
  if (err) {
    console.error("Database connection failed:", err.message);
  } else {
    console.log("MySQL Connected successfully");
    connection.release();
  }
});

module.exports = db;
```

### 2.3 Authentication System

#### Auth Controller (controllers/auth.controller.js)
```javascript
const db = require("../config/db");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcryptjs");

exports.login = (req, res) => {
  const { username, password } = req.body;

  // Validate input
  if (!username || !password) {
    return res.status(400).json({ 
      message: "Username dan password harus diisi" 
    });
  }

  // Check user in database
  db.query(
    "SELECT * FROM pengguna WHERE username = ?",
    [username],
    (err, results) => {
      if (err) {
        return res.status(500).json({ message: "Server error" });
      }
      
      if (results.length === 0) {
        return res.status(401).json({ 
          message: "Username tidak ditemukan" 
        });
      }

      const user = results[0];

      // Verify password
      bcrypt.compare(password, user.password, (err, isMatch) => {
        if (err) {
          return res.status(500).json({ message: "Server error" });
        }
        
        if (!isMatch) {
          return res.status(401).json({ message: "Password salah" });
        }

        // Generate JWT token
        const token = jwt.sign(
          { 
            id_user: user.id_user, 
            username: user.username 
          },
          process.env.JWT_SECRET || 'default_secret',
          { expiresIn: "1d" }
        );

        return res.json({
          message: "Login berhasil",
          token,
          user: {
            id_user: user.id_user,
            username: user.username
          }
        });
      });
    }
  );
};

exports.register = (req, res) => {
  const { username, password } = req.body;

  // Hash password
  bcrypt.hash(password, 10, (err, hashedPassword) => {
    if (err) {
      return res.status(500).json({ message: "Server error" });
    }

    db.query(
      "INSERT INTO pengguna (username, password) VALUES (?, ?)",
      [username, hashedPassword],
      (err, result) => {
        if (err) {
          if (err.code === 'ER_DUP_ENTRY') {
            return res.status(400).json({ 
              message: "Username sudah digunakan" 
            });
          }
          return res.status(500).json({ message: "Server error" });
        }

        res.status(201).json({ 
          message: "User berhasil didaftarkan" 
        });
      }
    );
  });
};
```

#### Auth Middleware (middleware/auth.js)
```javascript
const jwt = require("jsonwebtoken");

const verifyToken = (req, res, next) => {
  const authHeader = req.headers.authorization;
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ 
      message: "Access token tidak ditemukan" 
    });
  }

  jwt.verify(token, process.env.JWT_SECRET || 'default_secret', (err, decoded) => {
    if (err) {
      return res.status(401).json({ 
        message: "Token tidak valid" 
      });
    }
    
    req.user = decoded;
    next();
  });
};

module.exports = { verifyToken };
```

### 2.4 Menu Management

#### Menu Controller (controllers/menu.controller.js)
```javascript
const db = require("../config/db");

module.exports = {
  // Get all menu items
  getMenu: (req, res) => {
    const query = `
      SELECT m.*, s.nama_series 
      FROM menu m 
      LEFT JOIN series s ON m.id_series = s.id_series
      ORDER BY m.id_menu DESC
    `;
    
    db.query(query, (err, result) => {
      if (err) {
        return res.status(500).json({ 
          message: "Error fetching menu", 
          error: err.message 
        });
      }
      res.json({
        status: "success",
        data: result
      });
    });
  },

  // Add new menu item
  addMenu: (req, res) => {
    const { id_series, nama_menu, harga, stok } = req.body;
    const gambar = req.file?.filename || null;

    // Validation
    if (!nama_menu || !harga || !id_series) {
      return res.status(400).json({ 
        message: "Nama menu, harga, dan series harus diisi" 
      });
    }

    db.query(
      "INSERT INTO menu (id_series, nama_menu, harga, stok, gambar) VALUES (?,?,?,?,?)",
      [id_series, nama_menu, parseFloat(harga), parseInt(stok) || 0, gambar],
      (err, result) => {
        if (err) {
          return res.status(500).json({ 
            message: "Error adding menu", 
            error: err.message 
          });
        }
        
        res.status(201).json({ 
          message: "Menu berhasil ditambahkan",
          id_menu: result.insertId
        });
      }
    );
  },

  // Update menu item
  updateMenu: (req, res) => {
    const { id } = req.params;
    const { id_series, nama_menu, harga, stok } = req.body;
    const gambar = req.file?.filename;

    let query = "UPDATE menu SET id_series=?, nama_menu=?, harga=?, stok=?";
    let params = [id_series, nama_menu, parseFloat(harga), parseInt(stok)];

    if (gambar) {
      query += ", gambar=?";
      params.push(gambar);
    }

    query += " WHERE id_menu=?";
    params.push(id);

    db.query(query, params, (err, result) => {
      if (err) {
        return res.status(500).json({ 
          message: "Error updating menu", 
          error: err.message 
        });
      }
      
      if (result.affectedRows === 0) {
        return res.status(404).json({ 
          message: "Menu tidak ditemukan" 
        });
      }

      res.json({ message: "Menu berhasil diupdate" });
    });
  },

  // Delete menu item
  deleteMenu: (req, res) => {
    const { id } = req.params;

    db.query(
      "DELETE FROM menu WHERE id_menu = ?",
      [id],
      (err, result) => {
        if (err) {
          return res.status(500).json({ 
            message: "Error deleting menu", 
            error: err.message 
          });
        }
        
        if (result.affectedRows === 0) {
          return res.status(404).json({ 
            message: "Menu tidak ditemukan" 
          });
        }

        res.json({ message: "Menu berhasil dihapus" });
      }
    );
  },

  // Get series/categories
  getSeries: (req, res) => {
    db.query("SELECT * FROM series ORDER BY nama_series", (err, result) => {
      if (err) {
        return res.status(500).json({ 
          message: "Error fetching series", 
          error: err.message 
        });
      }
      res.json({
        status: "success",
        data: result
      });
    });
  }
};
```

### 2.5 Transaction Management

#### Transaction Controller (controllers/transaksi.controller.js)
```javascript
const db = require("../config/db");

module.exports = {
  // Create new transaction
  createTransaksi: (req, res) => {
    const { items, total_harga, metode_pembayaran } = req.body;

    // Start transaction
    db.beginTransaction((err) => {
      if (err) {
        return res.status(500).json({ message: "Transaction error" });
      }

      // Insert main transaction
      db.query(
        "INSERT INTO transaksi (total_harga, metode_pembayaran) VALUES (?, ?)",
        [total_harga, metode_pembayaran],
        (err, result) => {
          if (err) {
            return db.rollback(() => {
              res.status(500).json({ message: "Error creating transaction" });
            });
          }

          const id_transaksi = result.insertId;

          // Insert transaction details
          const detailPromises = items.map(item => {
            return new Promise((resolve, reject) => {
              db.query(
                "INSERT INTO detail_transaksi (id_transaksi, id_menu, jumlah, harga_satuan) VALUES (?, ?, ?, ?)",
                [id_transaksi, item.id_menu, item.jumlah, item.harga_satuan],
                (err, result) => {
                  if (err) reject(err);
                  else resolve(result);
                }
              );
            });
          });

          Promise.all(detailPromises)
            .then(() => {
              // Update stock
              const stockPromises = items.map(item => {
                return new Promise((resolve, reject) => {
                  db.query(
                    "UPDATE menu SET stok = stok - ? WHERE id_menu = ?",
                    [item.jumlah, item.id_menu],
                    (err, result) => {
                      if (err) reject(err);
                      else resolve(result);
                    }
                  );
                });
              });

              return Promise.all(stockPromises);
            })
            .then(() => {
              db.commit((err) => {
                if (err) {
                  return db.rollback(() => {
                    res.status(500).json({ message: "Commit error" });
                  });
                }

                res.status(201).json({
                  message: "Transaksi berhasil",
                  id_transaksi: id_transaksi
                });
              });
            })
            .catch((err) => {
              db.rollback(() => {
                res.status(500).json({ 
                  message: "Error processing transaction details",
                  error: err.message 
                });
              });
            });
        }
      );
    });
  },

  // Get transactions
  getTransaksi: (req, res) => {
    const { date } = req.query;
    let query = `
      SELECT t.*, dt.id_menu, dt.jumlah, dt.harga_satuan, m.nama_menu
      FROM transaksi t
      LEFT JOIN detail_transaksi dt ON t.id_transaksi = dt.id_transaksi
      LEFT JOIN menu m ON dt.id_menu = m.id_menu
    `;
    
    let params = [];
    if (date) {
      query += " WHERE DATE(t.tanggal) = ?";
      params.push(date);
    }
    
    query += " ORDER BY t.tanggal DESC";

    db.query(query, params, (err, result) => {
      if (err) {
        return res.status(500).json({ 
          message: "Error fetching transactions", 
          error: err.message 
        });
      }
      res.json({
        status: "success",
        data: result
      });
    });
  }
};
```

## 3. IMPLEMENTASI DATABASE (MySQL)

### 3.1 Database Schema

#### Create Database
```sql
CREATE DATABASE kasir_es_teh;
USE kasir_es_teh;
```

#### Tabel Pengguna
```sql
CREATE TABLE pengguna (
  id_user INT PRIMARY KEY AUTO_INCREMENT,
  username VARCHAR(50) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### Tabel Series/Kategori
```sql
CREATE TABLE series (
  id_series INT PRIMARY KEY AUTO_INCREMENT,
  nama_series VARCHAR(100) NOT NULL,
  deskripsi TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Tabel Menu
```sql
CREATE TABLE menu (
  id_menu INT PRIMARY KEY AUTO_INCREMENT,
  id_series INT,
  nama_menu VARCHAR(100) NOT NULL,
  harga DECIMAL(10,2) NOT NULL,
  stok INT DEFAULT 0,
  gambar VARCHAR(255),
  status ENUM('aktif', 'nonaktif') DEFAULT 'aktif',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (id_series) REFERENCES series(id_series) ON DELETE SET NULL
);
```

#### Tabel Transaksi
```sql
CREATE TABLE transaksi (
  id_transaksi INT PRIMARY KEY AUTO_INCREMENT,
  tanggal DATETIME DEFAULT CURRENT_TIMESTAMP,
  total_harga DECIMAL(10,2) NOT NULL,
  metode_pembayaran ENUM('cash', 'qr') NOT NULL,
  status ENUM('pending', 'completed', 'cancelled') DEFAULT 'completed',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Tabel Detail Transaksi
```sql
CREATE TABLE detail_transaksi (
  id_detail INT PRIMARY KEY AUTO_INCREMENT,
  id_transaksi INT NOT NULL,
  id_menu INT NOT NULL,
  jumlah INT NOT NULL,
  harga_satuan DECIMAL(10,2) NOT NULL,
  subtotal DECIMAL(10,2) GENERATED ALWAYS AS (jumlah * harga_satuan) STORED,
  FOREIGN KEY (id_transaksi) REFERENCES transaksi(id_transaksi) ON DELETE CASCADE,
  FOREIGN KEY (id_menu) REFERENCES menu(id_menu) ON DELETE RESTRICT
);
```

#### Tabel Keuangan
```sql
CREATE TABLE keuangan (
  id_keuangan INT PRIMARY KEY AUTO_INCREMENT,
  tanggal DATE NOT NULL,
  jenis ENUM('pemasukan', 'pengeluaran') NOT NULL,
  kategori VARCHAR(100),
  deskripsi TEXT,
  jumlah DECIMAL(10,2) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3.2 Sample Data

#### Insert Default User
```sql
-- Password: admin123 (hashed)
INSERT INTO pengguna (username, password) VALUES 
('admin', '$2b$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi');
```

#### Insert Series
```sql
INSERT INTO series (nama_series, deskripsi) VALUES 
('Minuman Dingin', 'Berbagai macam minuman dingin segar'),
('Minuman Panas', 'Minuman hangat untuk cuaca dingin'),
('Makanan Ringan', 'Camilan dan makanan ringan'),
('Makanan Berat', 'Makanan utama dan kenyang');
```

#### Insert Sample Menu
```sql
INSERT INTO menu (nama_menu, id_series, harga, stok) VALUES 
('Es Teh Manis', 1, 5000, 100),
('Es Jeruk', 1, 7000, 50),
('Es Kopi', 1, 8000, 75),
('Teh Panas', 2, 4000, 80),
('Kopi Hitam', 2, 6000, 60),
('Keripik Singkong', 3, 8000, 30),
('Nasi Goreng', 4, 15000, 20);
```

### 3.3 Database Views dan Stored Procedures

#### View untuk Laporan Harian
```sql
CREATE VIEW laporan_harian AS
SELECT 
  DATE(t.tanggal) as tanggal,
  COUNT(t.id_transaksi) as total_transaksi,
  SUM(t.total_harga) as total_pendapatan,
  SUM(dt.jumlah) as total_item_terjual
FROM transaksi t
LEFT JOIN detail_transaksi dt ON t.id_transaksi = dt.id_transaksi
WHERE t.status = 'completed'
GROUP BY DATE(t.tanggal)
ORDER BY tanggal DESC;
```

#### Stored Procedure untuk Update Stok
```sql
DELIMITER //
CREATE PROCEDURE UpdateStokMenu(
  IN p_id_menu INT,
  IN p_jumlah INT,
  IN p_operasi ENUM('tambah', 'kurang')
)
BEGIN
  DECLARE current_stok INT;
  
  SELECT stok INTO current_stok FROM menu WHERE id_menu = p_id_menu;
  
  IF p_operasi = 'tambah' THEN
    UPDATE menu SET stok = stok + p_jumlah WHERE id_menu = p_id_menu;
  ELSEIF p_operasi = 'kurang' THEN
    IF current_stok >= p_jumlah THEN
      UPDATE menu SET stok = stok - p_jumlah WHERE id_menu = p_id_menu;
    ELSE
      SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Stok tidak mencukupi';
    END IF;
  END IF;
END //
DELIMITER ;
```

### 3.4 Database Indexes untuk Performance

#### Indexes untuk Optimasi Query
```sql
-- Index untuk pencarian menu berdasarkan series
CREATE INDEX idx_menu_series ON menu(id_series);

-- Index untuk pencarian transaksi berdasarkan tanggal
CREATE INDEX idx_transaksi_tanggal ON transaksi(tanggal);

-- Index untuk detail transaksi
CREATE INDEX idx_detail_transaksi ON detail_transaksi(id_transaksi);
CREATE INDEX idx_detail_menu ON detail_transaksi(id_menu);

-- Index untuk keuangan berdasarkan tanggal
CREATE INDEX idx_keuangan_tanggal ON keuangan(tanggal);
```

---

*Implementasi sistem ini memberikan foundation yang solid untuk aplikasi kasir es teh dengan arsitektur yang scalable dan maintainable. Setiap komponen dirancang dengan prinsip separation of concerns dan best practices untuk pengembangan web modern.*