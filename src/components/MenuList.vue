<template>
    <div>
      <!-- Form Login -->
      <div v-if="!isLoggedIn">
        <h2>Login</h2>
        <form @submit.prevent="handleLogin">
          <input v-model="loginData.username" placeholder="Username" required />
          <input v-model="loginData.password" placeholder="Password" type="password" required />
          <button type="submit">Login</button>
        </form>
      </div>
  
      <!-- Konten Setelah Login -->
      <div v-else>
        <h2>Selamat Datang!</h2>
        <button @click="logout">Logout</button>
  
        <!-- Form Tambah Menu -->
        <form @submit.prevent="addItem">
          <input v-model="newItem.name" placeholder="Nama Menu" required />
          <input v-model.number="newItem.price" placeholder="Harga Menu" type="number" required />
          <input v-model="newItem.category" placeholder="Kategori (Makanan/Minuman)" required />
          <button type="submit">Tambah Menu</button>
        </form>
  
        <!-- Input Nomor Meja -->
        <div>
          <label for="table-number">Nomor Meja: </label>
          <input v-model="tableNumber" type="number" id="table-number" placeholder="Nomor Meja" />
        </div>
  
        <!-- Daftar Menu -->
        <h2>Daftar Menu:</h2>
        <button @click="sortMenu('asc')">Urutkan Berdasarkan Harga (Termurah)</button>
        <button @click="sortMenu('desc')">Urutkan Berdasarkan Harga (Termahal)</button>
        <button @click="filterMenu('Makanan')">Tampilkan Makanan</button>
        <button @click="filterMenu('Minuman')">Tampilkan Minuman</button>
        <button @click="selectKnapsack">Pilih Menu Berdasarkan Anggaran</button>
  
        <ul>
          <li v-for="item in displayedMenu" :key="item.name">
            {{ item.name }} - Rp{{ item.price }} ({{ item.category }})
          </li>
        </ul>
  
        <!-- Fitur Knapsack -->
        <div v-if="showKnapsackInput">
          <input v-model.number="budget" type="number" placeholder="Masukkan Anggaran" />
          <button @click="knapsack">Cari Menu dengan Anggaran</button>
          <ul>
            <li v-for="item in knapsackResult" :key="item.name">
              {{ item.name }} - Rp{{ item.price }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        // Status login
        isLoggedIn: false,
        loginData: {
          username: '',
          password: '',
        },
        credentials: {
          username: 'admin',
          password: 'password123',
        },
  
        // Data menu
        tableNumber: '',
        budget: 0,
        showKnapsackInput: false,
        knapsackResult: [],
        newItem: {
          name: '',
          price: 0,
          category: '',
        },
        menu: [
          { name: 'Coffee', price: 5000, category: 'Minuman' },
          { name: 'Pancong', price: 8000, category: 'Makanan' },
          { name: 'Mie Instan', price: 10000, category: 'Makanan' },
          { name: 'Teh Manis', price: 4000, category: 'Minuman' },
          { name: 'Es Jeruk', price: 6000, category: 'Minuman' },
          { name: 'Martabak Mini', price: 12000, category: 'Makanan' },
          { name: 'Ayam Penyet', price: 25000, category: 'Makanan' },
          { name: 'Burger', price: 20000, category: 'Makanan' },
          { name: 'Pizza Slice', price: 15000, category: 'Makanan' },
        ],
        displayedMenu: [],
      };
    },
    methods: {
      // Login handler
      handleLogin() {
        if (
          this.loginData.username === this.credentials.username &&
          this.loginData.password === this.credentials.password
        ) {
          this.isLoggedIn = true;
          alert('Login berhasil!');
        } else {
          alert('Username atau password salah!');
        }
      },
      logout() {
        this.isLoggedIn = false;
        this.loginData.username = '';
        this.loginData.password = '';
      },
  
      // Menu handlers
      addItem() {
        if (!this.newItem.name || this.newItem.price <= 0 || !this.newItem.category) {
          alert('Semua kolom harus diisi dengan benar.');
          return;
        }
        this.menu.push({ ...this.newItem });
        this.newItem.name = '';
        this.newItem.price = 0;
        this.newItem.category = '';
        this.displayedMenu = this.menu;
      },
      sortMenu(order) {
        this.displayedMenu = [...this.menu];
        if (order === 'asc') {
          this.displayedMenu.sort((a, b) => a.price - b.price);
        } else if (order === 'desc') {
          this.displayedMenu.sort((a, b) => b.price - a.price);
        }
      },
      filterMenu(category) {
        this.displayedMenu = this.menu.filter(item => item.category === category);
      },
      selectKnapsack() {
        this.showKnapsackInput = true;
      },
      knapsack() {
        if (this.budget <= 0) {
          alert('Anggaran harus lebih besar dari 0!');
          return;
        }
        const selectedItems = this.knapsackAlgorithm(this.menu, this.budget);
        this.knapsackResult = selectedItems;
        if (this.knapsackResult.length === 0) {
          alert('Tidak ada menu yang sesuai dengan anggaran.');
        }
      },
      knapsackAlgorithm(items, budget) {
        const n = items.length;
        const dp = Array(n + 1).fill(null).map(() => Array(budget + 1).fill(0));
  
        for (let i = 1; i <= n; i++) {
          for (let w = 1; w <= budget; w++) {
            if (items[i - 1].price <= w) {
              dp[i][w] = Math.max(
                items[i - 1].price + dp[i - 1][w - items[i - 1].price],
                dp[i - 1][w]
              );
            } else {
              dp[i][w] = dp[i - 1][w];
            }
          }
        }
  
        const result = [];
        let w = budget;
        for (let i = n; i > 0; i--) {
          if (dp[i][w] !== dp[i - 1][w]) {
            result.push(items[i - 1]);
            w -= items[i - 1].price;
          }
        }
        return result;
      },
    },
    mounted() {
      this.displayedMenu = this.menu;
    },
  };
  </script>
  
  <style scoped>
  form {
    margin-bottom: 20px;
  }
  
  ul {
    list-style: none;
    padding: 0;
  }
  
  li {
    margin: 5px 0;
  }
  
  button {
    margin: 5px;
  }
  
  input {
    margin: 5px;
    padding: 5px;
  }
  </style>
  