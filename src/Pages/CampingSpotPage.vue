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
          <button>Comment</button>
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
      isLoading: true,
      showModal: false,
      modalMessage: '',
      selectedSpot: null,
      totalPrice: 0,
      userId: null,  // Initialize userId to null initially
      selectedRating: 'any',
      selectedSize: 'any',
      selectedPrice: 'any'
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
          this.fetchCampTypes()
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
      return user ? `${user.firstName} ${user.lastName}` : 'Unknown';
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
    filterAll() {
      this.filteredCampingSpots = this.campingSpots;
    },
    filterAvailable() {
      this.filteredCampingSpots = this.campingSpots.filter(spot => spot.isAvailable);
    },
    filterByRating() {
      if (this.selectedRating === 'any') {
        this.filteredCampingSpots = this.campingSpots;
      } else {
        this.filteredCampingSpots = this.campingSpots.filter(spot => spot.rating === parseInt(this.selectedRating));
      }
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
    }
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
}

/* Styling for spot details */
.spot-details {
  flex: 1;
}

/* Styling for booking section */
.booking-section {
  flex: 1;
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
  flex: 1;
  padding-left: 10px;
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
</style>

