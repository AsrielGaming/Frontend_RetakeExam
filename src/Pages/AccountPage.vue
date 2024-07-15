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
          <p>{{ userData.username }}</p>
          <button @click="openPopup('username')">Update</button>
        </div>

        <!-- Email -->
        <div class="section">
          <p style="display:inline-block; margin-right: 10px;">Email:</p>
          <p>{{ userData.email }}</p>
          <button @click="openPopup('email')">Update</button>
        </div>

        <!-- Password -->
        <div class="section">
          <p style="display:inline-block; margin-right: 10px;">Password:</p>
          <p>{{ maskedPassword }}</p>
          <button @click="openPopup('password')">Update</button>
        </div>

        <!-- Button -->
        <div class="section">
          <button>Delete my account</button>
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
  </div>
</template>

<script>
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
      popupInput: ''
    };
  },
  computed: {
    maskedPassword() {
      // Masks the password except for the first and last characters
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
    confirmPopup() {
      // Functionality to be added
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
</style>
