<template>
  <div>
    <!-- Main container -->
    <div class="main-container">
      
      <!-- Banner container -->
      <div class="banner">
        <h3>{{ userData.username }}</h3>
      </div>
      
      <!-- Content container -->
      <div class="content">
        
        <!-- Username -->
        <div class="section">
          <p style="display:inline-block; margin-right: 10px;">Username:</p>
          <p>{{ updatedUserData.username || userData.username }}</p>
          <button @click="openPopup('username')">Update</button>
        </div>

        <!-- Email -->
        <div class="section">
          <p style="display:inline-block; margin-right: 10px;">Email:</p>
          <p>{{ updatedUserData.email || userData.email }}</p>
          <button @click="openPopup('email')">Update</button>
        </div>

        <!-- Password -->
        <div class="section">
          <p style="display:inline-block; margin-right: 10px;">Password:</p>
          <p>{{ maskedPassword }}</p>
          <button @click="openPopup('password')">Update</button>
        </div>

        <!-- Button to delete account -->
        <div class="section">
          <button @click="deleteAccount">Delete my account</button>
        </div>

      </div>
    </div>

    <!-- Popup container -->
    <div v-if="showPopup" class="popup-container">
      <div class="popup">
        <h3>{{ popupTitle }}</h3>
        <input v-if="popupTitle !== 'Change password'" type="text" v-model="popupInput" placeholder="Enter new value" @input="validateField" />
        <input v-else type="text" v-model="passwordInput" placeholder="Enter new password" @input="validateField" />
        <p v-if="popupTitle === 'Change email' && !isValidEmail(popupInput)" class="validation-message">Invalid email format</p>
        <p v-if="popupTitle === 'Change password' && !isValidPassword(passwordInput)" class="validation-message">Password must contain at least one uppercase letter, one number, and be at least 8 characters long</p>
        <div class="popup-buttons">
          <button @click="confirmPopup" :disabled="!isValidInput">Confirm</button>
          <button @click="cancelPopup">Cancel</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'AccountPage',
  props: {
    userData: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      showPopup: false,
      popupTitle: '',
      popupInput: '',
      passwordInput: '',
      updatedUserData: {}, // Local data to hold updated user data
      isValidInput: false // Flag to track if input is valid
    };
  },
  computed: {
    maskedPassword() {
      if (this.userData && this.userData.password) {
        const password = this.userData.password;
        const length = password.length;
        const masked = password.charAt(0) + '*'.repeat(length - 2) + password.charAt(length - 1);
        return masked;
      }
      return '';
    }
  },
  methods: {
    openPopup(field) {
      this.showPopup = true;
      this.popupInput = '';
      this.passwordInput = ''; // Clear password input when opening popup
      if (field === 'username') {
        this.popupTitle = 'Change username';
      } else if (field === 'email') {
        this.popupTitle = 'Change email';
      } else if (field === 'password') {
        this.popupTitle = 'Change password';
      }
    },
    cancelPopup() {
      this.showPopup = false;
      this.popupInput = '';
      this.passwordInput = '';
      this.isValidInput = false; // Reset validity check
    },
    isValidEmail(email) {
      const re = /\S+@\S+\.\S+/;
      return re.test(email);
    },
    isValidPassword(password) {
      const re = /^(?=.*[A-Z])(?=.*[0-9]).{8,}$/;
      return re.test(password);
    },
    validateField() {
      if (this.popupTitle === 'Change email') {
        this.isValidInput = this.isValidEmail(this.popupInput);
      } else if (this.popupTitle === 'Change password') {
        this.isValidInput = this.isValidPassword(this.passwordInput);
      } else {
        this.isValidInput = !!this.popupInput.trim(); // For username or any other input
      }
    },
    async confirmPopup() {
      if (this.popupTitle === 'Change username') {
        this.updatedUserData.username = this.popupInput;
      } else if (this.popupTitle === 'Change email') {
        this.updatedUserData.email = this.popupInput;
      } else if (this.popupTitle === 'Change password') {
        this.updatedUserData.password = this.passwordInput;
      }

      try {
        const { id, username, email, password } = Object.assign({}, this.userData, this.updatedUserData);
        const response = await axios.put(`http://localhost:5235/User/${id}`, {
          username,
          email,
          password
        });
        
        console.log('PUT request successful:', response.data);
        
        // Show success alert
        alert('User updated successfully!');

        // Auto close popup
        setTimeout(() => {
          this.showPopup = false;
          this.popupInput = '';
          this.passwordInput = '';
          this.isValidInput = false; // Reset validity check
        }, 100);

      } catch (error) {
        console.error('Failed to update:', error);
      } finally {
        // Reset updatedUserData
        this.updatedUserData = {};
      }
    },
    deleteAccount() {
      // Placeholder for account deletion logic
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

/* Styling for banner container */
.banner {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
  background-color: #f9f9f9;
}

/* Styling for content container */
.content {
  border: 1px solid #ccc;
  padding: 10px;
}

/* Styling for section */
.section {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

/* Styling for textbox */
input[type="text"],
input[type="password"] {
  width: 200px;
  margin-right: 10px;
}

/* Styling for button */
button {
  margin-left: 10px;
}

/* Styling for popup container */
.popup-container {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

/* Styling for popup */
.popup {
  background: white;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

/* Styling for popup buttons */
.popup-buttons {
  margin-top: 10px;
  display: flex;
  justify-content: space-between;
}

/* Styling for validation messages */
.validation-message {
  color: red;
  font-size: 0.8em;
  margin-top: 5px;
}
</style>
