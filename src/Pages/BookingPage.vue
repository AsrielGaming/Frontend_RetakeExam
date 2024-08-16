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

              <!-- Right side for comments and ratings -->
              <div class="comments-ratings-section">
                <!-- Top part for rating -->
                <div class="rating-section">
                  <p v-if="getRating(booking.spotId) !== undefined">Average Rating: {{ getRating(booking.spotId) }}</p>
                  <p v-else>Average Rating: Not available</p>
                </div>

                <!-- Bottom part for comments -->
                <div class="comment-section" v-if="getComments(booking.spotId).length > 0">
                  <h5>Comments:</h5>
                  <div v-for="comment in getComments(booking.spotId)" :key="comment.id" class="comment">
                    <p>{{ comment.text }}</p>
                    <small>Posted by: {{ getCommenterUsername(comment.userId) }}</small>
                  </div>
                </div>
                <p v-else>No comments available.</p>
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
      ratings: [],
      comments: [],
      isLoading: true
    };
  },
  computed: {
    // Filter bookings based on userData.id
    filteredBookings() {
      return this.bookings
        .filter(booking => booking.userId === this.userData.id)
        .map(booking => ({
          ...booking,
          username: this.getUserName(booking.userId),
          campingSpotName: this.getCampingSpotName(booking.campingSpotId),
          rating: this.getRating(booking.campingSpotId),
          comments: this.getComments(booking.campingSpotId)
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
          this.fetchCampingSpots(),
          this.fetchRatings(),
          this.fetchComments()
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
    async fetchRatings() {
      try {
        const response = await axios.get('http://localhost:5235/Rating');
        this.ratings = response.data;
      } catch (error) {
        console.error('Error fetching ratings:', error);
      }
    },
    async fetchComments() {
      try {
        const response = await axios.get('http://localhost:5235/Comment');
        this.comments = response.data;
      } catch (error) {
        console.error('Error fetching comments:', error);
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
    },
    getRating(spotId) {
      const ratings = this.ratings.filter(rating => rating.campingSpotId === spotId);
      if (ratings.length === 0) {
        return undefined;
      }
      const sum = ratings.reduce((total, rating) => total + rating.score, 0);
      const average = sum / ratings.length;
      return average.toFixed(1);
    },
    getComments(spotId) {
      return this.comments.filter(comment => comment.campingSpotId === spotId);
    },
    getCommenterUsername(userId) {
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
  max-height: 200px;
  justify-content: space-between;
}

/* Styling for booking details */
.booking-details {
  flex: 1;
  width: 50%;
}

/* Empty section styling */
.empty-section {
  flex: 1;
  border-left: 1px solid #ccc;
  padding-left: 10px;
}

/* Styling for the comments and rating container */
.comments-ratings-section {
  flex: 1;
  width: 50%;
  text-align: left;
}

/* Styling for comment box */
.comment-section {
  max-height: 150px;
  overflow-y: auto;  /* Enable vertical scrolling */
}

</style>
