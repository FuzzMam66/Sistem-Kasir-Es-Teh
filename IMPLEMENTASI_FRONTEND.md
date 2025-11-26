# Implementasi Fitur Frontend - Sistem Kasir Es Teh

## 1. Struktur Komponen React

### 1.1 Hierarki Komponen
```
App.jsx (Root)
├── Login.jsx
├── Dashboard.jsx
├── Penjualan.jsx
├── Checkout.jsx
├── QRPayment.jsx
├── CashPayment.jsx
├── KelolaMenu.jsx
├── LaporanHarian.jsx
├── Keuangan.jsx
└── Components/
    ├── Navbar.jsx
    ├── MenuCard.jsx
    ├── CounterButton.jsx
    ├── CategoryDropdown.jsx
    └── ProtectedRoute.jsx
```

## 2. Implementasi Halaman Utama

### 2.1 Login Page
```jsx
// State Management
const [username, setUsername] = useState("");
const [password, setPassword] = useState("");
const [error, setError] = useState("");
const [showPassword, setShowPassword] = useState(false);

// Authentication Logic
const handleLogin = async () => {
  try {
    const res = await api.post("/auth/login", { username, password });
    localStorage.setItem("token", res.data.token);
    navigate("/dashboard");
  } catch (err) {
    setError(err.response?.data?.message || "Login gagal");
  }
};

// UI Features:
- Modern glassmorphism design
- Password visibility toggle
- Form validation
- Error handling display
- Responsive layout
```

### 2.2 Dashboard Page
```jsx
// State untuk statistik real-time
const [stats, setStats] = useState({
  total_transaksi: 0,
  total_pendapatan: 0,
  total_item: 0
});

// Menu navigasi dengan icon
const menuItems = [
  {
    title: "Penjualan",
    description: "Kelola transaksi penjualan",
    icon: <MdShoppingCart size={48} />,
    path: "/penjualan",
    color: "bg-blue-500"
  },
  // ... menu lainnya
];

// Features:
- Card-based navigation
- Real-time statistics
- Refresh functionality
- Gradient backgrounds
- Hover animations
```

## 3. Sistem State Management

### 3.1 Local State dengan Hooks
```jsx
// useState untuk form data
const [formData, setFormData] = useState({
  nama_menu: "",
  harga: "",
  kategori: "",
  stok: ""
});

// useEffect untuk data fetching
useEffect(() => {
  const fetchData = async () => {
    try {
      const response = await api.get("/menu");
      setMenuItems(response.data);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  };
  fetchData();
}, []);
```

### 3.2 Form Handling
```jsx
// Generic form handler
const handleInputChange = (e) => {
  const { name, value } = e.target;
  setFormData(prev => ({
    ...prev,
    [name]: value
  }));
};

// Form submission
const handleSubmit = async (e) => {
  e.preventDefault();
  try {
    await api.post("/menu", formData);
    setFormData(initialState);
    fetchMenuItems(); // Refresh data
  } catch (error) {
    setError(error.response?.data?.message);
  }
};
```

## 4. API Integration

### 4.1 Axios Configuration
```jsx
// Base API setup
const api = axios.create({
  baseURL: "http://localhost:3001",
});

// Request interceptor untuk token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor untuk error handling
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
```

### 4.2 API Service Functions
```jsx
// Menu services
export const menuService = {
  getAll: () => api.get("/menu"),
  create: (data) => api.post("/menu", data),
  update: (id, data) => api.put(`/menu/${id}`, data),
  delete: (id) => api.delete(`/menu/${id}`)
};

// Transaction services
export const transactionService = {
  create: (data) => api.post("/transaksi", data),
  getDaily: (date) => api.get(`/laporan/harian?date=${date}`),
  getStats: () => api.get("/dashboard/stats")
};
```

## 5. Komponen Reusable

### 5.1 MenuCard Component
```jsx
const MenuCard = ({ menu, onAddToCart, quantity, onQuantityChange }) => {
  return (
    <div className="menu-card">
      <img src={menu.gambar} alt={menu.nama_menu} />
      <h3>{menu.nama_menu}</h3>
      <p>Rp {menu.harga.toLocaleString()}</p>
      <div className="quantity-controls">
        <CounterButton 
          value={quantity}
          onChange={onQuantityChange}
          min={0}
          max={menu.stok}
        />
      </div>
      <button onClick={() => onAddToCart(menu)}>
        Tambah ke Keranjang
      </button>
    </div>
  );
};
```

### 5.2 CounterButton Component
```jsx
const CounterButton = ({ value, onChange, min = 0, max = 999 }) => {
  const increment = () => {
    if (value < max) onChange(value + 1);
  };
  
  const decrement = () => {
    if (value > min) onChange(value - 1);
  };
  
  return (
    <div className="counter-button">
      <button onClick={decrement} disabled={value <= min}>-</button>
      <span>{value}</span>
      <button onClick={increment} disabled={value >= max}>+</button>
    </div>
  );
};
```

### 5.3 CategoryDropdown Component
```jsx
const CategoryDropdown = ({ categories, selected, onSelect }) => {
  return (
    <select 
      value={selected} 
      onChange={(e) => onSelect(e.target.value)}
      className="category-dropdown"
    >
      <option value="">Semua Kategori</option>
      {categories.map(category => (
        <option key={category.id} value={category.id}>
          {category.nama_series}
        </option>
      ))}
    </select>
  );
};
```

## 6. Routing dan Navigation

### 6.1 Protected Routes
```jsx
const ProtectedRoute = ({ children }) => {
  const token = localStorage.getItem("token");
  
  if (!token) {
    return <Navigate to="/" replace />;
  }
  
  return children;
};

// Usage in App.jsx
<Route 
  path="/dashboard" 
  element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  } 
/>
```

### 6.2 Navigation Component
```jsx
const Navbar = () => {
  const navigate = useNavigate();
  
  const handleLogout = () => {
    localStorage.removeItem("token");
    navigate("/");
  };
  
  return (
    <nav className="navbar">
      <div className="nav-brand">Kasir Es Teh</div>
      <div className="nav-links">
        <Link to="/dashboard">Dashboard</Link>
        <Link to="/penjualan">Penjualan</Link>
        <Link to="/kelola-menu">Menu</Link>
        <Link to="/laporan-harian">Laporan</Link>
        <button onClick={handleLogout}>Logout</button>
      </div>
    </nav>
  );
};
```

## 7. Styling dan UI/UX

### 7.1 Tailwind CSS Classes
```jsx
// Card styling
const cardClasses = `
  bg-white rounded-xl shadow-lg p-6 
  hover:shadow-xl transition-all duration-300
  border border-gray-200
`;

// Button styling
const buttonClasses = `
  bg-blue-500 hover:bg-blue-600 
  text-white font-semibold py-2 px-4 
  rounded-lg transition-colors duration-200
`;

// Form input styling
const inputClasses = `
  w-full px-4 py-2 border border-gray-300 
  rounded-lg focus:ring-2 focus:ring-blue-500 
  focus:border-transparent outline-none
`;
```

### 7.2 Inline Styles untuk Animasi
```jsx
// Hover animations
const hoverStyle = {
  transition: 'all 0.3s ease',
  transform: 'translateY(0)',
  onMouseOver: (e) => {
    e.target.style.transform = 'translateY(-4px)';
    e.target.style.boxShadow = '0 20px 25px -5px rgba(0, 0, 0, 0.1)';
  },
  onMouseOut: (e) => {
    e.target.style.transform = 'translateY(0)';
    e.target.style.boxShadow = '0 10px 15px -3px rgba(0, 0, 0, 0.1)';
  }
};

// Gradient backgrounds
const gradientStyle = {
  background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
  color: 'white'
};
```

## 8. Error Handling dan Loading States

### 8.1 Error Boundary Component
```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return <div>Something went wrong.</div>;
    }
    return this.props.children;
  }
}
```

### 8.2 Loading States
```jsx
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);

const fetchData = async () => {
  setLoading(true);
  setError(null);
  try {
    const response = await api.get("/data");
    setData(response.data);
  } catch (err) {
    setError(err.message);
  } finally {
    setLoading(false);
  }
};

// Loading UI
if (loading) return <div className="loading-spinner">Loading...</div>;
if (error) return <div className="error-message">Error: {error}</div>;
```

## 9. Form Validation

### 9.1 Client-side Validation
```jsx
const validateForm = (data) => {
  const errors = {};
  
  if (!data.nama_menu.trim()) {
    errors.nama_menu = "Nama menu harus diisi";
  }
  
  if (!data.harga || data.harga <= 0) {
    errors.harga = "Harga harus lebih dari 0";
  }
  
  if (!data.kategori) {
    errors.kategori = "Kategori harus dipilih";
  }
  
  return errors;
};

// Usage in component
const [errors, setErrors] = useState({});

const handleSubmit = (e) => {
  e.preventDefault();
  const validationErrors = validateForm(formData);
  
  if (Object.keys(validationErrors).length > 0) {
    setErrors(validationErrors);
    return;
  }
  
  // Submit form
  submitForm(formData);
};
```

## 10. Performance Optimization

### 10.1 React.memo untuk Komponen
```jsx
const MenuCard = React.memo(({ menu, onAddToCart }) => {
  return (
    <div className="menu-card">
      {/* Component content */}
    </div>
  );
});
```

### 10.2 useCallback dan useMemo
```jsx
// Memoize expensive calculations
const filteredMenus = useMemo(() => {
  return menus.filter(menu => 
    menu.nama_menu.toLowerCase().includes(searchTerm.toLowerCase())
  );
}, [menus, searchTerm]);

// Memoize callback functions
const handleAddToCart = useCallback((menu) => {
  setCart(prev => [...prev, menu]);
}, []);
```

## 11. Local Storage Management

### 11.1 Cart Persistence
```jsx
// Save cart to localStorage
useEffect(() => {
  localStorage.setItem('cart', JSON.stringify(cart));
}, [cart]);

// Load cart from localStorage
useEffect(() => {
  const savedCart = localStorage.getItem('cart');
  if (savedCart) {
    setCart(JSON.parse(savedCart));
  }
}, []);
```

### 11.2 User Preferences
```jsx
// Theme preference
const [theme, setTheme] = useState(() => {
  return localStorage.getItem('theme') || 'light';
});

useEffect(() => {
  localStorage.setItem('theme', theme);
  document.body.className = theme;
}, [theme]);
```

---

*Dokumentasi ini mencakup implementasi lengkap fitur frontend dengan fokus pada React.js, state management, API integration, dan best practices untuk pengembangan aplikasi web modern.*