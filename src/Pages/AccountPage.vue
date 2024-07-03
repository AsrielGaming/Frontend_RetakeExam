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
          <button>Update</button>
        </div>

        <!-- Email -->
        <div class="section">
          <p style="display:inline-block; margin-right: 10px;">Email:</p>
          <p>{{ userData.email }}</p>
          <button>Update</button>
        </div>

        <!-- Password -->
        <div class="section">
          <p style="display:inline-block; margin-right: 10px;">Password:</p>
          <p>{{ maskedPassword }}</p>
          <button>Update</button>
        </div>

        <!-- Button -->
        <div class="section">
          <button>Delete my account</button>
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
</style>
