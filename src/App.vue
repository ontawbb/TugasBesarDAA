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
          <button @click="selectMenu(item)">Pilih</button>
        </li>
      </ul>

      <!-- Fitur Knapsack -->
      <div v-if="showKnapsackInput" class="knapsack">
        <input v-model.number="budget" type="number" placeholder="Masukkan Anggaran" />
        <button @click="knapsack">Cari Menu dengan Anggaran</button>
        <ul>
          <li v-for="item in knapsackResult" :key="item.name">
            {{ item.name }} - Rp{{ item.price }}
          </li>
        </ul>
      </div>

      <!-- Menampilkan Struk -->
      <div v-if="selectedItems.length > 0" class="struk">
        <h2>Struk Pembelian</h2>
        <p>Nomor Meja: {{ tableNumber }}</p> <!-- Menampilkan nomor meja -->
        <ul>
          <li v-for="item in selectedItems" :key="item.name">
            {{ item.name }} - Rp{{ item.price }}
          </li>
        </ul>
        <h3>Total: Rp{{ totalAmount }}</h3>
        <button @click="printReceipt">Cetak Struk</button> <!-- Tombol Cetak Struk -->
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
      selectedItems: [],  // Menyimpan menu yang dipilih
    };
  },
  computed: {
    // Menghitung total harga
    totalAmount() {
      return this.selectedItems.reduce((sum, item) => sum + item.price, 0);
    }
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
      this.selectedItems = [];  // Reset menu yang dipilih
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
    selectMenu(item) {
      this.selectedItems.push(item);  // Tambahkan item yang dipilih ke struk
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
    printReceipt() {
      // Fungsi untuk mencetak struk hanya untuk item yang dipilih
      const receiptContent = `
        <h2>Struk Pembelian</h2>
        <p>Nomor Meja: ${this.tableNumber}</p> <!-- Menampilkan nomor meja -->
        <ul>
          ${this.selectedItems.map(item => `<li>${item.name} - Rp${item.price}</li>`).join('')}
        </ul>
        <h3>Total: Rp${this.totalAmount}</h3>
      `;
      
      const printWindow = window.open('', '', 'height=500,width=800');
      printWindow.document.write('<html><head><title>Struk Pembelian</title></head><body>');
      printWindow.document.write(receiptContent);
      printWindow.document.write('</body></html>');
      printWindow.document.close();
      printWindow.print();
    }
  },
  mounted() {
    this.displayedMenu = this.menu;
  },
};
</script>

<style scoped>
/* Styling Umum */
body {
  font-family: 'Arial', sans-serif;
  background-color: #f4f4f9;
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

h2 {
  font-size: 24px;
  color: #333;
  text-align: center;
}

h3 {
  font-size: 20px;
  color: #333;
  font-weight: bold;
  margin-top: 10px;
  text-align: center;
}

button {
  background-color: #28a745;
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  border-radius: 5px;
  margin: 5px;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #218838;
}

input {
  padding: 10px;
  font-size: 16px;
  border-radius: 5px;
  border: 1px solid #ccc;
  margin-bottom: 10px;
  width: 100%;
  max-width: 300px;
}

input:focus {
  outline: none;
  border-color: #28a745;
}

form {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  max-width: 400px;
  width: 100%;
}

ul {
  list-style: none;
  padding: 0;
  margin: 20px 0;
}

li {
  padding: 10px;
  border-bottom: 1px solid #ddd;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

li:last-child {
  border-bottom: none;
}

li button {
  background-color: #007bff;
}

li button:hover {
  background-color: #0056b3;
}

#table-number {
  width: 100%;
  max-width: 300px;
}

.struk {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
  max-width: 400px;
  width: 100%;
}

.struk ul {
  padding-left: 0;
}

.struk li {
  padding: 8px 0;
  border-bottom: 1px solid #ddd;
}

.struk h3 {
  color: #333;
  margin-top: 10px;
}

.struk button {
  background-color: #dc3545;
}

.struk button:hover {
  background-color: #c82333;
}

.knapsack {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
  max-width: 400px;
  width: 100%;
}

.knapsack ul {
  padding-left: 0;
}

.knapsack li {
  padding: 8px 0;
  border-bottom: 1px solid #ddd;
}

/* Tampilan Responsif */
@media (max-width: 768px) {
  .struk, .knapsack, form {
    width: 90%;
  }

  h2, h3 {
    font-size: 20px;
  }

  button {
    font-size: 14px;
    padding: 8px 16px;
  }
}
</style>
