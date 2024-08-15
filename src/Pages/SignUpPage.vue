<template>
  <div class="Page">
    <h1>Sign up here</h1>
    
    <!-- Main container -->
    <div class="main-container">
      
      <!-- Content container -->
      <div class="content">
        
        <!-- Username -->
        <div class="section">
          <p>Username:</p>
          <input type="text" v-model="username" placeholder="Enter your username here" />
          <span class="info-button" id="info-username" data-info="Username must be unique and contain only letters, numbers, or underscores.">?</span>
        </div>

        <!-- Email -->
        <div class="section">
          <p>Email:</p>
          <input type="text" v-model="email" placeholder="Enter your email here" />
          <span class="info-button" id="info-email" data-info="Email must be in the correct email address format.">?</span>
        </div>

        <!-- Password -->
        <div class="section">
          <p>Password:</p>
          <input type="password" v-model="password" placeholder="Enter your password" />
          <span class="info-button" id="info-password" data-info="Password must contain at least 1 capital letter, 1 number, and be 8 characters in length.">?</span>
        </div>

        <!-- Re Password -->
        <div class="section">
          <p>Re Password:</p>
          <input type="password" v-model="rePassword" placeholder="Re-enter your password" />
          <span class="info-button" id="info-repassword" data-info="Re-enter your password to confirm.">?</span>
        </div>

        <!-- Button -->
        <div class="section">
          <button @click="signup">Signup</button>
        </div>

      </div>
    </div>
    
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: "SignupPage",
  data() {
    return {
      //Initialize properties
      username: '',
      email: '',
      password: '',
      rePassword: '',
      users: [] //Userlist
    };
  },
  mounted() {
    this.loadUsers(); // Load users data when the page is loaded
  },
  methods: {
    async loadUsers() {
      try {
        const usersResponse = await axios.get('http://localhost:5235/User'); // Fetch users data
        this.users = usersResponse.data; // Set users data
      } catch (error) {
        console.error("Error fetching users:", error);
      }
    },
    //Check if all fields are filled in
    async signup() {
      if (!this.username || !this.email || !this.password || !this.rePassword) {
        alert("Please fill in all fields.");
        return;
      }
      //Check if email is in correct format
      if (!this.validateEmail(this.email)) {
        alert("Email is not in the correct format.");
        return;
      }
      //Check if password is in correct format
      if (!this.validatePassword(this.password)) {
        alert("Password must contain at least 1 capital letter, 1 number, and be 8 characters in length.");
        return;
      }
      //Check if passwords are the same
      if (this.password !== this.rePassword) {
        alert("Passwords do not match.");
        return;
      }

      try {
        // search if a user with the same username or email already exists
        const existingUser = this.users.find(user => user.username === this.username || user.email === this.email);
        if (existingUser) {
          alert("A user with the same username or email already exists.");
          return;
        }

        // Signup success
        const newUser = { username: this.username, email: this.email, password: this.password };
        await axios.post('http://localhost:5235/User', newUser); // Post new user data
        alert("User created successfully.");
        
        // Emit event to app.vue
        this.$emit('signup-success');
      } catch (error) {
        console.error("Error creating user:", error);
        alert("An error occurred during signup. Please try again.");
      }
    },
    validateEmail(email) {
      const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return emailPattern.test(email);
    },
    validatePassword(password) {
      const passwordPattern = /^(?=.*[A-Z])(?=.*\d).{8,}$/;
      return passwordPattern.test(password);
    }
  }
}
</script>

<style scoped>
/* Css for full page */
.Page {
  height: 89vh; /* view height */
  width: 99vw; /* view width */
  background-image: url('@/Images/LoginBackground.jpg');
  background-size: cover;
  background-position: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

/* Center the H1 */
h1 {
  text-align: center;
  color: white;
  margin: 0;
  padding: 10px;
  border-radius: 10px;
  width: 80%;
  position: relative;
}

/* Styling for main container */
.main-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 30%;
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 10px;
}

/* Styling for content */
.content {
  width: 100%;
}

/* Styling for section */
.section {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  justify-content: center;
}

/* Styling for textbox */
input[type="text"], input[type="password"] {
  width: 100%;
  max-width: 300px;
}

/* Styling for information button */
.info-button {
  margin-left: 5px;
  cursor: pointer;
  display: inline-block;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  text-align: center;
  background-color: #ccc;
  color: #fff;
  line-height: 20px;
  position: relative;
}

/* Styling for information button hover effect */
.info-button:hover::after {
  content: attr(data-info);
  position: absolute;
  background-color: #000;
  color: #fff;
  padding: 5px;
  border-radius: 5px;
  font-size: 12px;
  white-space: nowrap;
  left: 50%;
  transform: translateX(-50%);
  top: 25px;
}

button {
  margin-top: 10px;
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

button:hover {
  background-color: #45a049;
}

p {
  padding-right: 5px;
}
</style>