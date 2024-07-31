<template>
  <div>
    <!-- Main container -->
    <div class="main-container">
      
      <!-- Banner container -->
      <div class="banner">
        <h3>All Bookings</h3>
      </div>

      <!-- Content container -->
      <div class="content">
        <!-- Booking container -->
        <div class="bookings-container">
          <div v-if="isLoading">
            <p>Loading bookings...</p>
          </div>
          <div v-else-if="filteredBookings.length > 0">
            <div v-for="booking in filteredBookings" :key="booking.id" class="booking-container">
              <!-- Left side for booking details -->
              <div class="booking-details">
                <h3>Booking: {{ getCampingSpotName(booking.spotId) }}</h3>
                <p><strong>User:</strong> {{ booking.username }}</p>
                <p><strong>Start Date:</strong> {{ formatDate(booking.startDate) }}</p>
                <p><strong>End Date:</strong> {{ formatDate(booking.endDate) }}</p>
                <p><strong>Total Price:</strong> {{ booking.totalPrice }} €</p>
              </div>
              <!-- Right side for future use -->
              <div class="empty-section">
                <!-- Future content can go here -->
              </div>
            </div>
          </div>
          <div v-else>
            <p>No available bookings.</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'BookingPage',
  props: {
    userData: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      bookings: [],
      users: [],
      campingSpots: [],
      isLoading: true
    };
  },
  computed: {
    // Filtered bookings array based on userData.id
    filteredBookings() {
      return this.bookings
        .filter(booking => booking.userId === this.userData.id)
        .map(booking => ({
          ...booking,
          username: this.getUserName(booking.userId)
        }));
    }
  },
  mounted() {
    this.initializeData();
  },
  methods: {
    async initializeData() {
      try {
        await Promise.all([
          this.fetchBookings(),
          this.fetchUsers(),
          this.fetchCampingSpots()
        ]);
      } catch (error) {
        console.error('Error initializing data:', error);
      } finally {
        this.isLoading = false;
      }
    },
    async fetchBookings() {
      try {
        const response = await axios.get('http://localhost:5235/Booking');
        this.bookings = response.data;
      } catch (error) {
        console.error('Error fetching bookings:', error);
      }
    },
    async fetchUsers() {
      try {
        const response = await axios.get('http://localhost:5235/User');
        this.users = response.data;
      } catch (error) {
        console.error('Error fetching users:', error);
      }
    },
    async fetchCampingSpots() {
      try {
        const response = await axios.get('http://localhost:5235/CampingSpot');
        this.campingSpots = response.data;
      } catch (error) {
        console.error('Error fetching camping spots:', error);
      }
    },
    formatDate(dateString) {
      const options = { day: '2-digit', month: '2-digit', year: 'numeric' };
      return new Date(dateString).toLocaleDateString('en-GB', options);
    },
    getCampingSpotName(spotId) {
      const spot = this.campingSpots.find(spot => spot.id === spotId);
      return spot ? spot.spotName : 'Unknown Camp Name';
    },
    getUserName(userId) {
      const user = this.users.find(user => user.id === userId);
      return user ? user.username : 'Unknown User';
    }
  }
};
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
  background-color: #008080;
  color: white;
}

/* Styling for content container */
.content {
  display: flex;
  border: 1px solid #ccc;
  padding: 10px;
  background-color: #f9f9f9;
}

/* Styling for bookings container */
.bookings-container {
  width: 100%;
}

/* Styling for individual booking container */
.booking-container {
  border: 1px solid #ccc;
  margin-bottom: 10px;
  padding: 10px;
  display: flex;
}

/* Styling for booking details */
.booking-details {
  flex: 1;
}

/* Empty section styling */
.empty-section {
  flex: 1;
  border-left: 1px solid #ccc;
  padding-left: 10px;
}
</style>
