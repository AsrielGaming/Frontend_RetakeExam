<template>
  <div id="app">
    <!-- Navigation -->
    <nav style="display: flex; margin-bottom: 10px;">
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
      if (this.activePage === 'Login' || this.activePage === 'Sign Up') {
        return ["Login", "Sign Up"];
      } else {
        return this.pages.filter(page => page !== 'Login' && page !== 'Sign Up');
      }
    }
  },

  methods: {
    changeActivePage(page) {
      this.activePage = page;
    },

    getActiveStyling(page) {
      return page === this.activePage ? "text-decoration: underline; font-weight:bold;" : "";
    },
    
    handleLoginSuccess(userData) {
      //console.log("Received userData:", userData);
      this.userData = userData;
      this.userName = userData.username;
      this.changeActivePage('Homepage');
    },
    
    handleSignupSuccess() {
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
