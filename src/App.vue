<template>
  <div id="app">
    <!-- Navigation -->
    <nav style="display: flex; margin-bottom: 10px;">
      <!-- Unordered list of pages, when clicked active page becomes page clicked -->
      <ul style="display: flex; list-style-type: none; padding: 0; margin: 0;">
        <li v-for="(page, index) in filteredPages" :key="index" @click="changeActivePage(page)" :style="getActiveStyling(page)">
          <span style="margin-right: 10px;">{{ page }}</span>
        </li>
      </ul>
    </nav>

    <!-- Logout button -->
    <button v-if="activePage !== 'Login' && activePage !== 'Sign Up'" @click="logout" style="position: absolute; top: 10px; right: 10px;">Logout</button>

    <!-- Pages -->
    <div>
      <LoginPage v-if="activePage === 'Login'" @login-success="handleLoginSuccess" @login-simulated="changeActivePage('Homepage')" />
      <SignUpPage v-if="activePage === 'Sign Up'" @signup-success="handleSignupSuccess" />
      <HomePage v-if="activePage === 'Homepage'" :user-data="userData"/>
      <CampingSpotPage v-if="activePage === 'Camping spots'" :user-data="userData"/>
      <BookingPage v-if="activePage === 'Bookings'" :user-data="userData"/>
      <ListingPage v-if="activePage === 'Listings'" :user-data="userData"/>
      <AccountPage v-if="activePage === 'Account'" :user-data="userData" @logoutPostDelete="logout"/>
    </div>
  </div>
</template>

<script>
// imports
import LoginPage from './Pages/LoginPage.vue';
import SignUpPage from './Pages/SignUpPage.vue';
import HomePage from './Pages/HomePage.vue';
import CampingSpotPage from './Pages/CampingSpotPage.vue';
import AccountPage from './Pages/AccountPage.vue';
import ListingPage from './Pages/ListingPage.vue';
import BookingPage from './Pages/BookingPage.vue';

export default {
  name: 'App',

  components: {
    LoginPage,
    SignUpPage,
    HomePage, 
    CampingSpotPage,
    AccountPage,
    ListingPage,
    BookingPage
  },

  data() {
    return {
      activePage: 'Login',
      pages: [
        "Login",
        "Sign Up",
        "Homepage",
        "Camping spots",
        "Bookings",
        "Listings",
        "Account"
      ]
    }
  },

  computed: {
    filteredPages() {
      // only show login and signup in navigation bar on login and signup page, else show all pages except login and signup
      if (this.activePage === 'Login' || this.activePage === 'Sign Up') {
        return ["Login", "Sign Up"];
      } else {
        return this.pages.filter(page => page !== 'Login' && page !== 'Sign Up');
      }
    }
  },

  methods: {
    changeActivePage(page) {
      //change active page
      this.activePage = page;
    },

    getActiveStyling(page) {
      // styling for the activepage
      return page === this.activePage ? "text-decoration: underline; font-weight:bold;" : "";
    },
    
    handleLoginSuccess(userData) {
      //console.log("Received userData:", userData);
      // change active page to homepage and initialize userdata
      this.userData = userData;
      this.userName = userData.username;
      this.changeActivePage('Homepage');
    },
    
    handleSignupSuccess() {
      //Change active page to login
      this.changeActivePage('Login');
    },

    logout() {
      // Clear user data and set active page to Login when logout button is clicked
      this.userData = null;
      this.activePage = 'Login';
    }
  }
}
</script>

<style scoped>
nav {
  background-color: #374B4A;
  padding: 10px;
  border-radius: 5px;
  margin: 0px;
}

nav ul li {
  cursor: pointer;
  padding: 5px 10px;
  border-radius: 3px;
  color: white;
}

nav ul li:hover {
  background-color: #45615F;
}

nav ul li span {
  margin-right: 0;
}

button {
  margin-top: 10px;
  padding: 10px 20px;
  background-color: #374B4A;
  color: white;
  border: 1px solid white;
  cursor: pointer;
  border-radius: 5px;
}

button:hover {
  background-color: #45615F;
}
</style>
