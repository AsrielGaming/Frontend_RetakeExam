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
        <input type="text" v-model="popupInput" placeholder="Enter new value" />
        <div class="popup-buttons">
          <button @click="confirmPopup">Confirm</button>
          <button @click="cancelPopup">Cancel</button>
        </div>
      </div>
    </div>

    <!-- Success or Error Message Popup -->
    <popup-message v-if="showMessage" :message="messageText" :type="messageType" @close="showMessage = false"></popup-message>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'AccountPage',
  components: {
    PopupMessage: {
      props: ['message', 'type'],
      template: `
        <div class="popup-message" :class="type">
          <p>{{ message }}</p>
          <button @click="$emit('close')">Close</button>
        </div>
      `
    }
  },
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
      showMessage: false,
      messageText: '',
      messageType: '', // success or error
      updatedUserData: {} // Local data to hold updated user data
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
    },
    async confirmPopup() {
      if (this.popupTitle === 'Change username') {
        this.updatedUserData.username = this.popupInput;
      } else if (this.popupTitle === 'Change email') {
        this.updatedUserData.email = this.popupInput;
      }

      try {
        const { id, username, email, password } = Object.assign({}, this.userData, this.updatedUserData);
        const response = await axios.put(`http://localhost:5235/User/${id}`, {
          username,
          email,
          password
        });
        
        console.log('PUT request successful:', response.data);

        // Log updated user data
        console.log('Updated User Data:', {
          id,
          username,
          email,
          password
        });

        // Show success message
        this.showMessage = true;
        this.messageText = `Successfully changed ${this.popupTitle === 'Change username' ? 'username' : (this.popupTitle === 'Change email' ? 'email' : 'password')}`;
        this.messageType = 'success';

        // Auto close popup
        setTimeout(() => {
          this.showPopup = false;
          this.showMessage = false;
          this.popupInput = '';
        }, 100);

      } catch (error) {
        console.error('Failed to update:', error);

        // Show error message
        this.showMessage = true;
        this.messageText = 'Failed to update';
        this.messageType = 'error';
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
input[type="text"] {
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

/* Styling for success message */
.popup-message.success {
  background-color: #4CAF50; /* Green */
  color: white;
}

/* Styling for error message */
.popup-message.error {
  background-color: #f44336; /* Red */
  color: white;
}
</style>
