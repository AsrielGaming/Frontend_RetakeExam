<template>
  <div>
    <!-- Main container -->
    <div class="main-container">

      <!-- Overlay for darkening background -->
      <div v-if="showEditPopup" class="overlay"></div>

      <!-- Popup container -->
      <div v-if="showEditPopup" class="popup-container">
        <div class="popup-content">
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
                              label="typeName"
                              track-by="typeName"
                              placeholder="Select camp types"
                              :multiple="true"
                              :close-on-select="true"
                              id="dropdown4"
                              class="boxes camp-types-multiselect">
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
              <!-- Buttons -->
              <button @click="updateCampingSpot" class="update-button">Update Listing</button>
              <button @click="closePopup" class="cancel-button">Cancel</button>
              <p v-if="successMessage" class="success-message">{{ successMessage }}</p>
              <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
            </div>
          </div>
        </div>
      </div>

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
                          label="typeName"
                          track-by="typeName"
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

            <!-- New buttons container -->
            <div class="buttons">
              <button @click="openPopup(spot.id)">Edit CampingSpot</button>
              <button @click="deleteCampingSpot(spot.id)">Delete CampingSpot</button>
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
      showEditPopup: false,
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
        // Log the fetched amenities data
        console.log('Fetched amenities:', response.data);
        // Extracting names from amenities objects
        this.amenities = response.data.map(amenity => {
          //console.log('Processing amenity:', amenity);
          return amenity.name;
        });
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
    formatAmenities(amenityIds) {
      console.log('Selected amenity IDs:', amenityIds);
      console.log('Available amenities:', this.amenities);

      // Find names based on the selected IDs
      const amenityNames = amenityIds.map(id => {
        const amenity = this.amenities.find(a => a.id === id);
        console.log('Finding amenity ID:', id);
        console.log('Found amenity:', amenity);
        return amenity ? amenity.name : 'undefined';
      }).filter(name => name !== 'undefined'); // Remove 'undefined' entries

      console.log('Formatted amenity names:', amenityNames);
      return amenityNames.length > 0 ? amenityNames.join(', ') : 'undefined';
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
      this.errorMessage = 'Error while adding camping grounds.';
      this.successMessage = '';
      return;
    }
    const campingGroundId = campingGround.id;

    const amenities = this.newListing.dropdown3.map(selectedAmenity => { 
      const foundAmenity = this.amenities.find(amenity => { 
        console.log(`Checking: ${amenity.name} against selected: ${selectedAmenity}`); 
        return amenity.name === selectedAmenity; 
      }); 
      console.log(`Selected Amenity: ${selectedAmenity}, Found Amenity:`, foundAmenity); 
      return foundAmenity ? foundAmenity.id : 0;
    }); 

    // Ensure no amenities have id of 0 (not found)
    if (amenities.includes(0)) {
      this.errorMessage = 'Error while adding amenities.';
      this.successMessage = '';
      return;
    }

    // Map selected camp types to their IDs
    const campTypes = this.newListing.dropdown4.map(selectedCampType => {
      const foundCampType = this.campTypes.find(campType => campType.typeName === selectedCampType.typeName);
      return foundCampType ? foundCampType.id : 0;
    });

    // Ensure no camp types have id of 0 (not found)
    if (campTypes.includes(0)) {
      this.errorMessage = 'Error while adding camp types.';
      this.successMessage = '';
      return;
    }

    // Prepare data for POST request
    const userId = this.userData ? this.userData.id : null;
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
    },
    async deleteCampingSpot(campingSpotId) {
      // Show confirmation dialog
      const confirmation = confirm("Are you sure you want to delete your listing?");
      
      // If the user confirms, proceed with deletion
      if (confirmation) {
        try {
          await axios.delete(`http://localhost:5235/CampingSpot/${campingSpotId}`);
          
          // Success alert
          alert('Successfully deleted listing.');
          
          // Refresh the list of camping spots after deletion
          this.fetchCampingSpots();
        } catch (error) {
          // Error alert
          alert('Error while trying to delete listing.');
          
          console.error('Error deleting CampingSpot:', error.response);
        }
      } else {
        // If the user cancels, do nothing
        alert('Deletion cancelled.');
      }
    },
    updateCampingSpot(campingSpotId) {
      console.log('Update CampingSpot with ID:', campingSpotId);
    },
    closePopup() {
      this.showEditPopup = false; // Close the popup
    },
    openPopup() {
      this.showEditPopup = true; // Opens the popup
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
  background-color: #008080;
  color: white;
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
  background-color: #f9f9f9;
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
  width: 100%;
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

/* Styling for the new buttons container */
.buttons {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

/* Styling for the buttons within the container */
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

/* Overlay styling */
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 999;
}

/* Ensure the popup container is fixed in size and centered */
.popup-container {
  position: fixed;
  top: 50%;
  left: 50%;
  width: 80%;
  height: 56%;
  background-color: white;
  display: flex;
  flex-direction: column; /* Stack content vertically */
  justify-content: center;
  align-items: center;
  z-index: 1000;
  transform: translate(-50%, -50%); /* Center the popup on the screen */
  overflow: visible; /* Allow overflow from the popup content */
  box-sizing: border-box;
}

/* Popup content should take full width and height but allow overflow */
.popup-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  overflow: visible; /* Allow overflow beyond this container */
}

/* Ensure multiselect can overflow */
.multiselect {
  overflow: visible; /* Allow multiselect to extend beyond its container */
}

/* Optional: styling for multiselect to visually indicate overflow */
.multiselect-container {
  overflow: visible; /* Ensure the container does not clip the multiselect */
  position: relative; /* Ensure multiselect positioning is correct */
}

.update-button {
  margin-right: 10px;
}

</style>
