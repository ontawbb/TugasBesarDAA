<template>
    <div>
      <h1>Dashboard Kasir</h1>
      <h2>Pesanan Masuk</h2>
      <ul>
        <li v-for="order in orders" :key="order.id">
          <h3>Meja {{ order.tableNumber }}</h3>
          <ul>
            <li v-for="item in order.items" :key="item.name">
              {{ item.name }} - Rp{{ item.price }} x {{ item.quantity }}
            </li>
          </ul>
          <p><strong>Total:</strong> Rp{{ calculateTotal(order.items) }}</p>
          <button @click="processOrder(order.id)">Proses Pesanan</button>
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        orders: [
          {
            id: 1,
            tableNumber: 5,
            items: [
              { name: 'Coffee', price: 5000, quantity: 2 },
              { name: 'Pancong', price: 8000, quantity: 1 },
            ],
          },
          {
            id: 2,
            tableNumber: 3,
            items: [
              { name: 'Mie Instan', price: 10000, quantity: 1 },
              { name: 'Teh Manis', price: 4000, quantity: 2 },
            ],
          },
        ],
      };
    },
    methods: {
      calculateTotal(items) {
        return items.reduce((total, item) => total + item.price * item.quantity, 0);
      },
      processOrder(orderId) {
        this.orders = this.orders.filter(order => order.id !== orderId);
        alert(`Pesanan ID ${orderId} telah diproses.`);
      },
    },
  };
  </script>
  
  <style scoped>
  h1, h2 {
    text-align: center;
  }
  
  button {
    margin-top: 10px;
  }
  </style>
  