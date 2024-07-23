<template>
  <div>
    <!-- Main container -->
    <div class="main-container">
      
      <!-- Banner container -->
      <div class="banner">
        <h3>All camping spots</h3>
      </div>

      <!-- Filter container -->
      <div class="filter-container">
        <div class="filter-controls">
          <button @click="filterAll">All</button>
          <button @click="filterAvailable">Available</button>
          <button @click="filterByComments">Comment</button>
        </div>

        <!-- Additional dropdowns -->
        <div class="dropdown">
          <select @change="filterByRating" v-model="selectedRating">
            <option value="any">Any Rating</option>
            <option value="1">1 star rating</option>
            <option value="2">2 star rating</option>
            <option value="3">3 star rating</option>
            <option value="4">4 star rating</option>
            <option value="5">5 star rating</option>
          </select>
        </div>
        
        <div class="dropdown">
          <select @change="filterBySize" v-model="selectedSize">
            <option value="any">Any Size</option>
            <option value="small">Small (≤ 50 m²)</option>
            <option value="medium">Medium (51-100 m²)</option>
            <option value="large">Large (> 100 m²)</option>
          </select>
        </div>
        
        <div class="dropdown">
          <select @change="filterByPrice" v-model="selectedPrice">
            <option value="any">Any Price</option>
            <option value="cheap">Cheap (≤ 50 €)</option>
            <option value="normal">Normal (51-100 €)</option>
            <option value="luxury">Luxury (> 100 €)</option>
          </select>
        </div>

        <!-- Amenity dropdown (multi-select) using vue-multiselect -->
        <div class="dropdown multiselect-dropdown">
          <multiselect v-model="selectedAmenities"
                       :options="amenitiesList.map(amenity => amenity.name)"
                       placeholder="Select amenities"
                       :multiple="true"
                       :close-on-select="false"
                       :clear-on-select="false"
                       :preserve-search="true"
                       :preselect-first="false">
            <template slot="selection" slot-scope="{ values, isOpen }">
              <span class="multiselect__single" v-if="values.length && !isOpen">{{ values.length }} options selected</span>
            </template>
          </multiselect>
        </div>
      </div>
      
      <!-- Content container -->
      <div class="content">
        <!-- Campingspots container -->
        <div class="campingspots-container">
          <div v-if="isLoading">
            <p>Loading camping spots...</p>
          </div>
          <div v-else-if="filteredCampingSpots.length">
            <div v-for="spot in filteredCampingSpots" :key="spot.id" class="camping-spot-container">
              <!-- Left side for spot details -->
              <div class="spot-details">
                <div class="camping-spot">
                  <h3>{{ spot.spotName }}</h3>
                  <p><strong>Description:</strong> {{ spot.description }}</p>
                  <p><strong>CampTypes:</strong> {{ getCampTypeNames(spot.campTypes) }}</p>
                  <p><strong>Size:</strong> {{ spot.size }} m²</p>
                  <p><strong>Price:</strong> {{ spot.price }} €</p>
                  <p><strong>Availability:</strong> {{ spot.isAvailable ? 'Available' : 'Not Available' }}</p>
                  <p><strong>Camping Ground:</strong> {{ getCampingGroundName(spot.campingGroundId) }}</p>
                  <p><strong>Owner:</strong> {{ getUserName(spot.userId) }}</p>
                </div>
              </div>

              <!-- Middle section for booking -->
              <div class="booking-section">
                <h3>Book this spot</h3>
                <div class="date-picker-section">
                  <label><strong>Start Date:</strong></label>
                  <input type="date" v-model="spot.startingDate" @change="validateDates(spot)">
                </div>
                <div class="date-picker-section">
                  <label><strong>End Date:</strong></label>
                  <input type="date" v-model="spot.endDate" @change="validateDates(spot)">
                </div>
                <div v-if="spot.dateError" class="error-message">
                  <p>{{ spot.dateError }}</p>
                </div>
                <div class="book-button-container">
                  <button :disabled="!spot.startingDate || !spot.endDate || spot.dateError" @click="calculateAndShowTotalPrice(spot)">Book</button>
                </div>
              </div>

              <!-- Right side for amenities -->
              <div class="amenities-section">
                <h3>Amenities</h3>
                <ul v-if="spot.amenities && spot.amenities.length">
                  <li v-for="amenityId in spot.amenities" :key="amenityId">
                    {{ getAmenityName(amenityId) }}
                  </li>
                </ul>
                <p v-else>No amenities</p>
              </div>

              <!-- New section for comments or additional amenities -->
              <div class="comment-amenity-section">
                <!-- Top container for rating -->
                <div class="comment-amenity-section-top">
                  <p v-if="getRating(spot.id) !== undefined">Average Rating: {{ getRating(spot.id) }}</p>
                  <div class="rating-system">
                    <span>Your rating:</span>
                    <span v-for="star in 5" :key="star" class="star"
                          :class="{ 'filled': star <= spot.userRating }"
                          @click="setUserRating(spot, star)">
                      ★
                    </span>
                  </div>
                </div>

                <!-- Textbox for writing a comment -->
                <div class="comment-input-section">
                  <textarea v-model="newComment[spot.id]" placeholder="Write your comment here..." rows="4"></textarea>
                  <button @click="submitComment(spot.id)">Submit</button>
                </div>

                <!-- Bottom container for comments -->
                <div class="comment-amenity-section-bottom">
                  <div class="comment-container" v-if="getComments(spot.id).length > 0">
                    <h5>Comments:</h5>
                    <div v-for="comment in getComments(spot.id)" :key="comment.id" class="comment">
                      <p>{{ comment.text }}</p>
                      <small>Posted by: {{ getCommenterUsername(comment.userId) }}</small>
                    </div>
                  </div>
                  <p v-else>No comments available.</p>
                </div>
              </div>

            </div>
          </div>
          <div v-else>
            <p>No available camping spots.</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Confirmation Dialog -->
    <div v-if="showModal" class="confirmation-dialog">
      <div class="dialog-content">
        <p>{{ modalMessage }}</p>
        <div class="dialog-actions">
          <button @click="closeModal">Cancel</button>
          <button @click="handleProceed">Proceed</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Multiselect from 'vue-multiselect';

export default {
  name: 'CampingSpotPage',
  components: {
    Multiselect
  },
  props: {
    userData: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      amenitiesList: [],
      selectedAmenities: [],
      campingGrounds: [],
      campingSpots: [],
      filteredCampingSpots: [],
      users: [],
      campTypes: [],
      ratings: [],
      comments: [],
      isLoading: true,
      showModal: false,
      modalMessage: '',
      selectedSpot: null,
      totalPrice: 0,
      userId: null,  // Initialize userId to null initially
      selectedRating: 'any',
      selectedSize: 'any',
      selectedPrice: 'any',
      newComment: {}
    };
  },
  mounted() {
    this.initializeData();
  },
  methods: {
    async initializeData() {
      try {
        await Promise.all([
          this.fetchAmenities(),
          this.fetchCampingGrounds(),
          this.fetchUsers(),
          this.fetchCampingSpots(),
          this.fetchCampTypes(),
          this.fetchRatings(),
          this.fetchComments(),
        ]);

        // Set userId based on userData.id if userData exists and has id
        if (this.userData && this.userData.id) {
          this.userId = this.userData.id;
        } else {
          console.error('User data or user id not available.');
        }

        this.filteredCampingSpots = this.campingSpots;
      } catch (error) {
        console.error('Error initializing data:', error);
      } finally {
        this.isLoading = false;
      }
    },
    async fetchAmenities() {
      try {
        const response = await axios.get('http://localhost:5235/Amenity');
        this.amenitiesList = response.data;
      } catch (error) {
        console.error('Error fetching amenities:', error);
      }
    },
    async fetchCampingGrounds() {
      try {
        const response = await axios.get('http://localhost:5235/CampingGround');
        this.campingGrounds = response.data;
      } catch (error) {
        console.error('Error fetching camping grounds:', error);
      }
    },
    async fetchCampingSpots() {
      try {
        const response = await axios.get('http://localhost:5235/CampingSpot');
        this.campingSpots = response.data.map(spot => ({
          ...spot,
          startingDate: '',
          endDate: ''
        }));
      } catch (error) {
        console.error('Error fetching camping spots:', error);
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
    async fetchCampTypes() {
      try {
        const response = await axios.get('http://localhost:5235/Camptype');
        this.campTypes = response.data;
      } catch (error) {
        console.error('Error fetching camp types:', error);
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
    getCampTypeNames(campTypeIds) {
      const ids = JSON.parse(JSON.stringify(campTypeIds));

      if (!ids || ids.length === 0) {
        return 'No camp types';
      }

      const names = ids.map(id => {
        const campType = this.campTypes.find(type => type.id === id);
        return campType ? campType.typeName : 'Undefined';
      });

      return names.length > 0 ? names.join(', ') : 'Undefined';
    },
    getAmenityName(amenityId) {
      const amenity = this.amenitiesList.find(amenity => amenity.id === amenityId);
      return amenity ? amenity.name : 'Undefined';
    },
    getAmenityIdByName(name) {
      const amenity = this.amenitiesList.find(a => a.name === name);
      return amenity ? amenity.id : null;
    },
    getCampingGroundName(campingGroundId) {
      const ground = this.campingGrounds.find(g => g.id === campingGroundId);
      return ground ? ground.name : 'Undefined';
    },
    getUserName(userId) {
      const user = this.users.find(u => u.id === userId);
      return user ? user.username : 'Undefined';
    },
    getCommenterUsername(userId) {
      // Get username based on userId
      const user = this.users.find(user => user.id === userId);
      return user ? user.username : 'Unknown User';
    },
    validateDates(spot) {
      if (spot.startingDate && spot.endDate) {
        if (new Date(spot.startingDate) > new Date(spot.endDate)) {
          alert('End date must be the same or later than start date');
          spot.endDate = ''; // Clear the end date
        }
      }
    },
    calculateAndShowTotalPrice(spot) {
      const startDate = new Date(spot.startingDate);
      const endDate = new Date(spot.endDate);
      const timeDiff = endDate.getTime() - startDate.getTime();
      const dayDiff = timeDiff / (1000 * 3600 * 24) + 1; // Including the start date
      const totalPrice = spot.price * dayDiff;
      this.modalMessage = `The total price will be: ${totalPrice} €. Are you sure you want to book this campingspot?`;
      this.showModal = true;
      this.selectedSpot = spot;
      this.totalPrice = totalPrice;
    },
    closeModal() {
      this.showModal = false;
    },
    async handleProceed() {
      if (!this.selectedSpot) {
        return;
      }

      const bookingData = {
        userId: this.userId,
        spotId: this.selectedSpot.id,
        startDate: this.selectedSpot.startingDate,
        endDate: this.selectedSpot.endDate,
        totalPrice: this.totalPrice
      };

      try {
        await axios.post('http://localhost:5235/Booking', bookingData);
        alert('Booking successful!');
        this.showModal = false;
      } catch (error) {
        console.error('Error creating booking:', error);
        alert('An error occurred while creating the booking.');
      }
    },
    getRating(campingSpotId) {
      // Get all ratings for a camping spot and compute average
      const ratings = this.ratings.filter(rating => rating.campingSpotId === campingSpotId);
      if (ratings.length === 0) {
        return undefined; // No ratings
      }
      const sum = ratings.reduce((total, rating) => total + rating.score, 0);
      const average = sum / ratings.length;
      return average.toFixed(1); // Display average with 1 decimal place
    },
    getComments(campingSpotId) {
      return this.comments.filter(comment => comment.campingSpotId === campingSpotId);
    },
    filterAll() {
      this.filteredCampingSpots = this.campingSpots;
    },
    filterAvailable() {
      this.filteredCampingSpots = this.campingSpots.filter(spot => spot.isAvailable);
    },
    filterBySize() {
      if (this.selectedSize === 'any') {
        this.filteredCampingSpots = this.campingSpots;
      } else if (this.selectedSize === 'small') {
        this.filteredCampingSpots = this.campingSpots.filter(spot => spot.size <= 50);
      } else if (this.selectedSize === 'medium') {
        this.filteredCampingSpots = this.campingSpots.filter(spot => spot.size > 50 && spot.size <= 100);
      } else if (this.selectedSize === 'large') {
        this.filteredCampingSpots = this.campingSpots.filter(spot => spot.size > 100);
      }
    },
    filterByPrice() {
      if (this.selectedPrice === 'any') {
        this.filteredCampingSpots = this.campingSpots;
      } else if (this.selectedPrice === 'cheap') {
        this.filteredCampingSpots = this.campingSpots.filter(spot => spot.price <= 50);
      } else if (this.selectedPrice === 'normal') {
        this.filteredCampingSpots = this.campingSpots.filter(spot => spot.price > 50 && spot.price <= 100);
      } else if (this.selectedPrice === 'luxury') {
        this.filteredCampingSpots = this.campingSpots.filter(spot => spot.price > 100);
      }
    },
    filterByAmenities() {
      if (this.selectedAmenities.length === 0) {
        this.filteredCampingSpots = this.campingSpots;
      } else {
        this.filteredCampingSpots = this.campingSpots.filter(spot =>
          this.selectedAmenities.every(selectedAmenity =>
            spot.amenities.includes(this.getAmenityIdByName(selectedAmenity))
          )
        );
      }
    },
     filterByComments() {
      this.filteredCampingSpots = this.campingSpots.filter(spot => this.getComments(spot.id).length > 0);
    },

    filterByRating() {
      if (this.selectedRating === 'any') {
        this.filteredCampingSpots = this.campingSpots;
      } else {
        const ratingFilter = parseInt(this.selectedRating);
        this.filteredCampingSpots = this.campingSpots.filter(spot => {
          const averageRating = this.getRating(spot.id);
          if (averageRating !== undefined) {
            const floorRating = Math.floor(averageRating); // Get integer part of the rating
            return floorRating === ratingFilter;
          }
          return false;
        });
      }
    },
    isEmpty(spotId) {
      if (!this.newComment[spotId]) {
        alert('Comment cannot be empty.');
        return true;
      }
      return false;
    },
    submitComment(spotId) {
      if (!this.isEmpty(spotId)) {
        this.postComment(spotId);
      }
    },
    postComment(spotId) {
      const commentData = {
        text: this.newComment[spotId],
        campingSpotId: spotId,
        userId: this.userId,
      };
      // Call API to save the comment
      axios.post('http://localhost:5235/Comment', commentData)
        .then(() => {
          alert('Comment submitted successfully!');
          this.fetchComments(); // Refresh comments after submission
          this.newComment[spotId] = ''; // Clear the comment input for the specific spot
        })
        .catch(error => {
          console.error('Error submitting comment:', error);
          alert('An error occurred while submitting the comment.');
        });
    },
    setUserRating(spot, rating) {
      // Update userRating for the specific spot
      this.$set(spot, 'userRating', rating);
    }
    },
  watch: {
    selectedRating() {
      this.filterByRating();
    },
    selectedSize() {
      this.filterBySize();
    },
    selectedPrice() {
      this.filterByPrice();
    },
    selectedAmenities() {
      this.filterByAmenities();
    },
  }
};
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>

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

/* Styling for filter container */
.filter-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  padding: 10px;
  position: relative;
}

.filter-controls {
  display: flex;
  align-items: center;
}

.filter-controls > *:not(:last-child) {
  margin-right: 5px;
}

.filter-container button {
  text-align: center;
  margin: 0 5px;
}

/* Styling for dropdown */
.dropdown {
  position: relative;
  margin: 0 5px;
}

.dropdown select {
  width: 200px;
  padding: 5px;
}

/* Styling for multiselect dropdown */
.multiselect-dropdown .multiselect__content-wrapper {
  position: absolute;
  top: 100%;
  left: 0;
  z-index: 1000;
}

/* Styling for content container */
.content {
  display: flex;
  border: 1px solid #ccc;
  padding: 10px;
}

/* Styling for campingspots container */
.campingspots-container {
  width: 100%;
}

/* Styling for individual camping spot container */
.camping-spot-container {
  border: 1px solid #ccc;
  margin-bottom: 10px;
  padding: 10px;
  display: flex;
  max-height: 500px;
}

/* Styling for spot details */
.spot-details {
  flex: 25%; /* 25% width */
}

/* Styling for booking section */
.booking-section {
  flex: 12.5%; /* 12.5% width */
  padding: 10px;
  border-left: 1px solid #ccc;
  border-right: 1px solid #ccc;
  display: flex;
  flex-direction: column;
}

.date-picker-section {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.date-picker-section label {
  margin-right: 10px;
}

.book-button-container {
  margin-top: auto;
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

/* Styling for amenities section */
.amenities-section {
  flex: 12.5%; /* 12.5% width */
  padding-left: 10px;
  border-right: 1px solid #ccc;
}

/* Styling for comment-amenity-section */
.comment-amenity-section {
  flex: 25%; /* 25% width */
  padding-left: 10px; /* Adjust padding as needed */
}

.comment-container{
  overflow-y: scroll;
  max-height: 150px;
}

/* Styling for confirmation dialog */
.confirmation-dialog {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.dialog-content {
  background: white;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
}

.dialog-actions {
  margin-top: 20px;
  display: flex;
  justify-content: space-around;
}

.dialog-actions button {
  padding: 10px 20px;
}

/* Styling for rating system */
.rating-system {
  display: flex;
  align-items: center; /* Align items vertically center */
}

.rating-label {
  margin-right: 10px; /* Space between label and stars */
  font-weight: bold;
  font-size: 1em; /* Ensure label font size matches stars */
}

.stars {
  display: flex; /* Ensure stars are displayed in a row */
}

.star {
  cursor: pointer;
  font-size: 1.5em;
  color: #ccc;
}

.star.filled {
  color: #FFD700;
}

/* Styling for comment input section */
.comment-input-section {
  margin-top: 5px;
}

.comment-input-section textarea {
  width: 100%;
  max-width: 95%;
  height: 100px; /* Set initial height */
  max-height: 200px; /* Set maximum height */
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  resize: vertical; /* Allow vertical resize */
}

.comment-input-section button {
  margin-top: 10px;
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

.comment-input-section button:hover {
  background-color: #45a049;
}

</style>
