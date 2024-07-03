<template>
  <div>
    <h1>Login here</h1>
    
    <!-- Main container -->
    <div class="main-container">
      
      <!-- Content container -->
      <div class="content">
        
        <!-- Email -->
        <div class="section">
          <p style="display:inline-block; margin-right: 10px;">Email:</p>
          <input type="text" v-model="email" placeholder="Enter your email here" />
        </div>

        <!-- Password -->
        <div class="section">
          <p style="display:inline-block; margin-right: 10px;">Password:</p>
          <input type="password" v-model="password" placeholder="Enter your password here" />
        </div>
        
        <!-- Button -->
        <div class="section">
          <button @click="login">Login</button>
        </div>

      </div>
    </div>
    
    <!-- Temporary button -->
    <button @click="simulateLogin">Simulate Login</button>
    
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: "LoginPage",
  data() {
    return {
      email: '',
      password: '',
    };
  },
  methods: {
    changeActivePage(page) {
      this.$emit('change-active-page', page);
    },
    login() {
      // Check if email and password are filled
      if (!this.email || !this.password) {
        alert("Please fill in all fields.");
        return;
      }

      // Fetch users data
      axios.get('http://localhost:5235/User')
        .then(response => {
          //console.log("Users data:", response.data); // Log users data to console
          const users = response.data; // Extract users data
              
          // Check if email and password belong to any user
          const userMatch = users.find(user => user.email === this.email && user.password === this.password);
          if (userMatch) {
            // Login successful for user
            const userData = {
              id: userMatch.id,
              username: userMatch.username,
              email: userMatch.email,
              password: userMatch.password
            };
            this.$emit('login-success', userData); // Emit login success event
            this.changeActivePage('Homepage'); // Change active page to Homepage
          } else {
            // Email or password is incorrect
            alert("Username or password is incorrect, please try again.");
          }
        })
        .catch(error => {
          console.error("Error fetching users:", error);
          alert("An error occurred during login. Please try again.");
        });
    },
    showInfo(event) {
      const infoText = event.target.getAttribute('data-info'); // Get info text from data-info attribute
      alert(infoText); // Show info alert
    },
    simulateLogin() {
      this.$emit('login-simulated');
    }
  }
}
</script>

<style scoped>
/* Styling for main container */
.main-container {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

/* Styling for content */
.content {
  border: 1px solid #ccc;
  padding: 10px;
  margin-top: 10px;
}

/* Styling for section */
.section {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
</style>
