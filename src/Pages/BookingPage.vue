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
          <div v-else-if="bookings.length">
            <div v-for="booking in bookings" :key="booking.id" class="booking-container">
              <!-- Left side for booking details -->
              <div class="booking-details">
                <h3>Booking Details</h3>
                <p><strong>User ID:</strong> {{ booking.userId }}</p>
                <p><strong>Spot ID:</strong> {{ booking.spotId }}</p>
                <p><strong>Start Date:</strong> {{ booking.startDate }}</p>
                <p><strong>End Date:</strong> {{ booking.endDate }}</p>
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
  data() {
    return {
      bookings: [],
      users: [],
      campingSpots: [],
      isLoading: true
    };
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
  background-color: #f9f9f9;
}

/* Styling for content container */
.content {
  display: flex;
  border: 1px solid #ccc;
  padding: 10px;
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
