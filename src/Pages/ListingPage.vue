<template>
  <div>
    <!-- Main container -->
    <div class="main-container">

      <!-- Banner container -->
      <div class="banner">
        <h3>All Listings</h3>
      </div>

      <!-- Create Listing container -->
      <div class="create-listing-container">
        <!-- Top section -->
        <div class="create-listing-container-top">
          <!-- Left side -->
          <div class="create-listing-container-left">
            <!-- Name field -->
            <div class="field">
              <label for="name">Name: <span class="required-symbol">*</span></label>
              <input type="text" id="name" v-model="newListing.name" class="boxes" required />
            </div>
            <!-- Size picker -->
            <div class="field">
              <label for="size">Size: <span class="required-symbol">*</span></label>
              <input type="number" id="size" v-model.number="newListing.size" class="boxes" min="1" required />
            </div>
            <!-- Description field -->
            <div class="field">
              <label for="description">Description: <span class="required-symbol">*</span></label>
              <input type="text" id="description" v-model="newListing.description" class="boxes" required />
            </div>
            <!-- Price picker -->
            <div class="field">
              <label for="price">Price: <span class="required-symbol">*</span></label>
              <input type="number" id="price" v-model.number="newListing.price" class="boxes" min="1" required />
            </div>
          </div>

          <!-- Right side -->
          <div class="create-listing-container-right">
            <!-- Dropdown boxes -->
            <div class="field">
              <label for="dropdown2">Camping Ground: <span class="required-symbol">*</span></label>
              <select id="dropdown2" v-model="newListing.dropdown2" class="boxes" required>
                <option disabled value="">Please select one</option>
                <option v-for="ground in campingGrounds" :key="ground.id" :value="ground.name">{{ ground.name }}</option>
              </select>
            </div>

            <div class="field">
              <label for="dropdown3">Amenity:</label>
              <multiselect v-model="newListing.dropdown3"
                           :options="amenities"
                           placeholder="Select amenities"
                           :multiple="true"
                           :close-on-select="true"
                           id="dropdown3"
                           class="boxes">
              </multiselect>
            </div>

            <div class="field">
              <label for="dropdown4">Camp Type: <span class="required-symbol">*</span></label>
              <multiselect v-model="newListing.dropdown4"
                           :options="campTypes"
                           placeholder="Select camp types"
                           :multiple="true"
                           :close-on-select="true"
                           id="dropdown4"
                           class="boxes">
              </multiselect>
            </div>

            <!-- Availability toggle -->
            <div class="field checkbox-group">
              <label for="isAvailable">Is Available:</label>
              <input type="checkbox" id="isAvailable" v-model="newListing.isAvailable" style="align-self: flex-start;" />
            </div>
          </div>
        </div>

        <!-- Bottom section -->
        <div class="create-listing-container-bottom">
          <!-- Button -->
          <button @click="createListing">Create Listing</button>
          <p v-if="successMessage" class="success-message">{{ successMessage }}</p>
          <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
        </div>
      </div>

      <!-- Content container -->
      <div class="content">
        <!-- Listing container -->
        <div class="listing-container">
          <div v-for="spot in filteredCampingSpots" :key="spot.id" class="listing-spot-container">
            <!-- Left container -->
            <div class="listing-container-left">
              <h4>{{ spot.spotName }}</h4>
              <p>Description: {{ spot.description }}</p>
              <p>Camp Types: {{ formatCampTypes(spot.campTypes) }}</p>
              <p>Size: {{ spot.size }} m²</p>
              <p>Price: {{ spot.price }} €</p>
              <p>Availability: {{ spot.isAvailable ? 'Available' : 'Not Available' }}</p>
              <p>Camping Ground: {{ getCampingGroundName(spot.campingGroundId) }}</p>
            </div>
            <!-- Right container -->
            <div class="listing-container-right">
              <!-- Top part for rating -->
              <div class="listing-container-right-top">
                <p v-if="getRating(spot.id) !== undefined">Average Rating: {{ getRating(spot.id) }}</p>
                <p v-else>Average Rating: undefined</p>
              </div>

              <!-- Bottom part for comments -->
              <div class="listing-container-right-bottom">
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

            <!-- Amenities section -->
            <div class="listing-container-amenities">
              <h4>Amenities</h4>
              <ul>
                <template v-if="spot.amenities.length > 0">
                  <li v-for="amenityId in spot.amenities" :key="amenityId">
                    <!-- Find the amenity name from the array using the ID -->
                    {{ amenities.find(amenity => amenity.id === amenityId)?.name || 'undefined' }}
                  </li>
                </template>
                <template v-else>
                  <li>undefined</li>
                </template>
              </ul>
            </div>

          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Multiselect from 'vue-multiselect';

export default {
  name: 'ListingPage',
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
      newListing: {
        name: '',
        size: null,
        description: '',
        price: null,
        isAvailable: true,
        dropdown2: '', // Will be filled with camping grounds names
        dropdown3: [], // Use array for multi-select
        dropdown4: []  // Use array for multi-select
      },
      campingSpots: [],
      campingGrounds: [],
      amenities: [],
      campTypes: [],
      ratings: [],
      comments: [],
      users: [], // Added users array
      isLoading: true,
      successMessage: '',
      errorMessage: ''
    };
  },
  computed: {
    filteredCampingSpots() {
      // Filter camping spots based on logged-in user's ID
      const userId = this.userData ? this.userData.id : null;
      return this.campingSpots.filter(spot => spot.userId === userId);
    }
  },
  mounted() {
    this.initializeData();
  },
  methods: {
    async initializeData() {
      try {
        await Promise.all([
          this.fetchCampingSpots(),
          this.fetchCampingGrounds(),
          this.fetchAmenities(),
          this.fetchCampTypes(),
          this.fetchRatings(),
          this.fetchComments(),
          this.fetchUsers() // Fetch users data
        ]);
      } catch (error) {
        console.error('Error initializing data:', error);
      } finally {
        this.isLoading = false;
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
    async fetchCampingGrounds() {
      try {
        const response = await axios.get('http://localhost:5235/CampingGround');
        this.campingGrounds = response.data;
      } catch (error) {
        console.error('Error fetching camping grounds:', error);
      }
    },
    async fetchAmenities() {
      try {
        const response = await axios.get('http://localhost:5235/Amenity');
        // Extracting names from amenities objects
        this.amenities = response.data.map(amenity => amenity.name);
      } catch (error) {
        console.error('Error fetching amenities:', error);
      }
    },
    async fetchCampTypes() {
      try {
        const response = await axios.get('http://localhost:5235/Camptype');
        // Store campTypes as an array of objects
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
    async fetchUsers() {
      try {
        const response = await axios.get('http://localhost:5235/User');
        this.users = response.data;
      } catch (error) {
        console.error('Error fetching users:', error);
      }
    },
    formatCampTypes(campTypeIds) {
      // Map campTypeIds to their names
      const campTypeNames = campTypeIds.map(id => {
        const campType = this.campTypes.find(type => type.id === id);
        return campType ? campType.typeName : '';
      }).filter(name => name !== ''); // Remove empty strings

      return campTypeNames.length > 0 ? campTypeNames.join(', ') : 'undefined';
    },
    getCampingGroundName(campingGroundId) {
      // Get camping ground name based on ID
      const campingGround = this.campingGrounds.find(ground => ground.id === campingGroundId);
      return campingGround ? campingGround.name : '';
    },
    getCommenterUsername(userId) {
      // Get username based on userId
      const user = this.users.find(user => user.id === userId);
      return user ? user.username : 'Unknown User';
    },
    createListing() {
      // Validate required fields
      if (!this.newListing.name ||
          !this.newListing.size ||
          !this.newListing.description ||
          !this.newListing.price ||
          !this.newListing.dropdown2) {
        this.errorMessage = 'Please fill in all required fields.';
        this.successMessage = '';
        return;
      }

      // Validate size and price
      if (this.newListing.size <= 0) {
        this.errorMessage = 'Size must be greater than 0.';
        this.successMessage = '';
        return;
      }
      if (this.newListing.price <= 0) {
        this.errorMessage = 'Price must be greater than 0.';
        this.successMessage = '';
        return;
      }

      // Find selected camping ground object to get its ID
      const campingGround = this.campingGrounds.find(ground => ground.name === this.newListing.dropdown2);
      if (!campingGround) {
        this.errorMessage = 'Invalid camping ground selected.';
        this.successMessage = '';
        return;
      }
      const campingGroundId = campingGround.id;

      // Map selected amenities and camp types to their IDs or leave them empty if not selected
      const amenities = this.newListing.dropdown3.map(selectedAmenity => {
        // `selectedAmenity` is a string
        const foundAmenity = this.amenities.find(amenity => amenity.name === selectedAmenity);
        return foundAmenity ? foundAmenity.id : 0;
      });

      const campTypes = this.newListing.dropdown4.map(selectedCampType => {
        // `selectedCampType` is an object
        const foundCampType = this.campTypes.find(campType => campType.typeName === selectedCampType.typeName);
        return foundCampType ? foundCampType.id : 0;
      });

      // Prepare data for POST request
      const userId = this.userData ? this.userData.id : null; // Assuming userData.id exists
      const postData = {
        spotName: this.newListing.name,
        size: this.newListing.size,
        description: this.newListing.description,
        price: this.newListing.price,
        isAvailable: this.newListing.isAvailable,
        userId: userId,
        campingGroundId: campingGroundId,
        amenities: amenities,
        campTypes: campTypes
      };

      // POST request to create new camping spot
      axios.post('http://localhost:5235/CampingSpot', postData)
        .then(response => {
          console.log('New Camping Spot created:', response.data);
          this.successMessage = 'New Camping Spot created successfully!';
          this.errorMessage = '';
          this.resetForm();
        })
        .catch(error => {
          console.error('Error creating new Camping Spot:', error.response);
          if (error.response && error.response.data && error.response.data.errors) {
            this.errorMessage = 'Failed to create new Camping Spot. Please check the form and try again.';
          } else {
            this.errorMessage = 'Failed to create new Camping Spot. Please try again later.';
          }
          this.successMessage = '';
        });
    },
    resetForm() {
      // Reset the form fields
      this.newListing = {
        name: '',
        size: null,
        description: '',
        price: null,
        isAvailable: true,
        dropdown2: '',
        dropdown3: [],
        dropdown4: []
      };
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

/* Styling for create listing container */
.create-listing-container {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
  background-color: #f9f9f9;
  display: flex;
  flex-direction: column;
}

/* Styling for create listing container top section */
.create-listing-container-top {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

/* Styling for create listing container bottom section */
.create-listing-container-bottom {
  margin-top: 10px;
  text-align: center;
}

/* Styling for create listing container left section */
.create-listing-container-left {
  width: 48%;
}

/* Styling for create listing container right section */
.create-listing-container-right {
  width: 48%;
  display: flex;
  flex-direction: column;
}

/* Add this style to the existing scoped styles */
.listing-container-amenities {
  width: 48%;
  padding: 10px;
}

.listing-container-amenities h4 {
  margin-left: 25px;
}

/* Styling for field */
.field {
  margin-bottom: 10px;
}

.field label {
  display: block;
  margin-bottom: 5px;
}

.field .boxes {
  width: 40%; 
  padding: 5px;
  box-sizing: border-box;
}

/* Styling for isAvailable checkbox */
.checkbox-group {
  display: flex;
  align-items: center;
}

.checkbox-group label {
  margin-right: 10px; 
}

/* Styling for content container */
.content {
  display: flex;
  border: 1px solid #ccc;
  padding: 10px;
}

/* Styling for listing container */
.listing-container {
  width: 100%;
}

.listing-container-right-bottom{
  overflow-y: scroll;
  max-height: 200px;
}

/* Styling for individual listing container */
.listing-spot-container {
  border: 1px solid #ccc;
  margin-bottom: 10px;
  padding: 10px;
  display: flex;
  justify-content: space-between;
}

/* Styling for left and right containers within each listing spot */
.listing-container-left, .listing-container-right {
  width: 48%;
}

/* Styling for listing information */
.listing-info {
  width: 100%; /* Adjust as per your layout */
}

/* Styling for required symbol */
.required-symbol {
  color: red;
  margin-left: 5px;
}

/* Styling for success and error messages */
.success-message {
  color: green;
  margin-top: 10px;
}

.error-message {
  color: red;
  margin-top: 10px;
}
</style>
