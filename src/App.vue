<template>
  <div>
    <h1 v-text="sitename"></h1>
    <div v-if="lessonView" class="sorting-container">
      <p>Select Sorting order ({{ selectedSortOrder }}):</p>
      <div class="input-group">
        <label for="sort">Sort By:</label>
        <select id="sort" v-model="selectedSort">
          <option value="">-- Select Option --</option>
          <option value="subject">Subject</option>
          <option value="location">Location</option>
          <option value="price">Price</option>
          <option value="spaces">Availability</option>
        </select>
      </div>
      <!-- Dropdown menu to select sorting order -->
      <div class="input-group">
        <label for="order">Order by:</label>
        <select id="order" v-model="selectedSortOrder">
          <option value="">-- Select Order --</option>
          <option value="Ascending">Ascending</option>
          <option value="Descending">Descending</option>
        </select>
      </div>
    </div>

    <!-- search funcionality -->

    <div class="search" v-show="lessonView">
      <label for="search">Search:</label>
      <input
        type="text"
        v-model="searchText"
        @input="searchFunction"
        placeholder="search"
        style="width: 40%"
      />
    </div>

    <div id="search-results" v-if="searchActive">
      <h2>Search Results:</h2>
      <ul>
        <li
          v-for="(lesson, index) in searchedItems"
          :key="index"
          class="lesson"
        >
          <i :class="['fas', lesson.icon, 'fa-3x']"></i>
          <p>Subject: {{ lesson.subject }}</p>
          <p>Location: {{ lesson.location }}</p>
          <p>Price: {{ lesson.price }}</p>
          <p>Spaces: {{ lesson.spaces }}</p>
          <button
            @click="bookLesson(lesson)"
            :class="{ 'hoverable-button': lesson.spaces > 0 }"
            :disabled="lesson.spaces === 0"
          >
            <i class="fas fa-shopping-cart"></i> Add to cart
          </button>
        </li>
      </ul>
    </div>

    <!-- Shopping Cart Button -->
    <!-- if you are on the lesson page and the cart is empty the button will be disabled-->
    <!-- if lesson view button true show count and icon else -->
    <button
      class="cart-button"
      v-if="!searchActive"
      @click="toggleCart"
      :disabled="lessonView && cartEmpty"
    >
      <i
        class="fas fa-shopping-cart"
        v-show="lessonView && toggleButtonText === 'Checkout'"
      ></i>
      {{
        toggleButtonText === "Checkout"
          ? toggleButtonText + " (" + cartCount + ")"
          : toggleButtonText
      }}
    </button>
    <!-- Render the CheckoutComponent only if showCheckoutComponent is true -->
    <!-- Pass necessary data to the CheckoutComponent as props -->
    <!-- Listen for specific events emitted by the CheckoutComponent -->
    <CheckoutComponent
      v-if="showCheckoutComponent"
      :cart="cart"
      :searchActive="searchActive"
      :phoneNumberIsValid="phoneNumberIsValid"
      :userNameIsValid="userNameIsValid"
      :checkoutMessage="checkoutMessage"
      :cartCount="cartCount"
      :userName="userName"
      :userNumber="userNumber"
      @remove-item-from-cart="removeFromCart"
      @update-user-name="updateUserName"
      @update-user-number="updateUserNumber"
      @checkout="checkout"
    ></CheckoutComponent>
    <!-- Test button -->
    <button class="Test-button" v-if="!testActive" @click="activateTests">
      Test
    </button>

    <!-- Test section -->
    <div v-if="testActive" class="test-section">
      <h2>Test Section</h2>
      <!-- Test buttons -->
      <button @click="reloadPage">Reload Page</button>
      <span style="margin-left: 10px"></span>
      <button @click="deleteCaches">Delete Caches</button>
      <span style="margin-left: 10px"></span>
      <button @click="unregisterServiceWorkers">
        Unregister Service Workers
      </button>
      <span style="margin-left: 10px"></span>
      <a
        href="https://lessonapp3-env.eba-mrkmnxsq.eu-west-2.elasticbeanstalk.com/collections/products"
        @click=""
        >Test HTTPS Link</a
      >
    </div>

    <!-- Lesson Component -->
    <!-- Render LessonComponent only when on the lesson view -->
    <LessonComponent
      v-if="lessonView && !searchActive"
      :lessons="lessons"
      @add-item-to-cart="bookLesson"
    ></LessonComponent>
  </div>
</template>

<script>
//importing child components
import LessonComponent from "./components/LessonComponent.vue";
import CheckoutComponent from "./components/CheckoutComponent.vue";
export default {
  components: {
    LessonComponent,
    CheckoutComponent,
  },
  data() {
    return {
      sitename: "lesson Booking App",
      //array of lesson objects
      //data properties
      lessons: [],
      //store slections temporarily
      selectedSort: "",
      selectedSortOrder: "",
      cartEmpty: true,
      // Track the current view cart or lesson
      lessonView: true,
      cart: [],
      toggleButtonText: "Checkout",
      userName: "",
      userNumber: "",
      sortingOrderMessage: "",
      cartCount: 0,
      checkoutMessage: "",
      searchText: "",
      searchedItems: [],
      searchActive: false,
      testActive: false,
      showCheckoutComponent: false,
    };
  },
  //fetch promise to call data from express middleware
  created: function () {
    if ("serviceWorker" in navigator) {
      navigator.serviceWorker.register("service_worker.js");
    }
    fetch(
      "https://lessonapp3-env.eba-mrkmnxsq.eu-west-2.elasticbeanstalk.com/collections/products"
    ).then((res) => {
      res.json().then((json) => {
        this.lessons = json;
      });
    });
  },
  methods: {
    bookLesson(lesson) {
      if (lesson.spaces > 0) {
        // Decrease the spaces count by 1 and update the shopping cart
        lesson.spaces--;
        this.cart.push(lesson);

        // Update cart count and empty status
        this.cartCount++;
        this.cartEmpty = false;

        // Clear search box and reset search variables
        this.searchText = "";
        this.searchedItems = [];
        this.searchActive = false;

        console.log("Lesson added to cart:", lesson);
      }
    },
    sortLessons() {
      //  copy of lessons  array
      let sorted = [...this.lessons];

      // Conditionally checking the selected sorting criteria
      if (this.selectedSort === "subject" || this.selectedSort === "location") {
        // Sorting alphabetically by subject or location
        sorted.sort((a, b) => {
          const aValue = a[this.selectedSort].toLowerCase();
          const bValue = b[this.selectedSort].toLowerCase();
          return aValue.localeCompare(bValue);
        });
      } else if (
        this.selectedSort === "price" ||
        this.selectedSort === "spaces"
      ) {
        // Sorting numerically by price or spaces
        sorted.sort(
          (a, b) =>
            parseFloat(a[this.selectedSort]) - parseFloat(b[this.selectedSort])
        );
      }

      // Checking the selected sorting order (asc or desc) and reverse the array if needed
      if (this.selectedSortOrder === "Descending") {
        sorted.reverse();
      }

      // Populate  lessons array with the sorted results
      this.lessons = [...sorted];
    },

    toggleCart() {
      // Toggle between the lesson view and shopping cart
      this.lessonView = !this.lessonView;
      if (this.lessonView) {
        this.toggleButtonText = "Checkout";
        this.showCheckoutComponent = false; // Hide CheckoutComponent when switching to lessons view
      } else {
        this.toggleButtonText = "Back to Lessons";
        this.showCheckoutComponent = this.cart.length > 0; // Show CheckoutComponent only if there are items in the cart
      }
    },

    // removing cart item and increasing the available space when cart item is removed from the cart
    removeFromCart(index) {
      const removedLesson = this.cart[index];
      this.cart.splice(index, 1);
      removedLesson.spaces += 1;
      // Check to see if the cart is empty after removing an item
      if (this.cart.length === 0) {
        this.cartEmpty = true;
      }
      this.cartCount--;
    },

    checkout() {
      // Data to be sent to the server and database.
      const lessonIDs = this.cart.map((lesson) => lesson.lesson_id);
      const spaces = this.cart.map((lesson) => lesson.spaces);
      console.log("Checkout method called");
      const orderData1 = {
        userName: this.userName,
        userNumber: this.userNumber,
        cart: this.cart,
        lessonIDs: lessonIDs,
        spaces: spaces[0],
      };
      const orderData2 = {
        lessonIDs: lessonIDs,
        spaces: spaces,
      };
      console.log("Checkout method called");
      // POST request to save the new order
      fetch(
        "https://lessonapp3-env.eba-mrkmnxsq.eu-west-2.elasticbeanstalk.com/collections/orders",
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(orderData1),
        }
      )
        .then((response) => {
          if (!response.ok) {
            throw new Error("Failed to save the order");
          }
          return response.json();
        })
        .then((data) => {
          // Handle the successful response from the server
          console.log("Order saved successfully:", data);
          this.checkoutMessage = `Congratulations Checkout Successfull, Name: ${this.userName} , Number: ${this.userNumber}`;

          // Clear the cart and reset other data
          this.cart = [];
          this.cartCount = 0;
          this.userName = "";
          this.userNumber = "";

          setTimeout(() => {
            this.checkoutMessage = "";
          }, 5000);

          // PUT to Update lesson spaces after order is submitted
          return fetch(
            "https://lessonapp3-env.eba-mrkmnxsq.eu-west-2.elasticbeanstalk.com/collections/products",
            {
              method: "PUT",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify(orderData2),
            }
          );
        })
        .then((response) => {
          if (!response.ok) {
            throw new Error("Failed to update lesson spaces");
          }
          return response.json();
        })
        .then((data) => {
          console.log("Lesson spaces updated successfully:", data);
        })
        .catch((error) => {
          // Handling any errors during the POST or PUT requests
          console.error("Error during checkout:", error);
          this.checkoutMessage = "Error during checkout. Please try again.";
        });
    },
    // Function to handle search functionality
    searchFunction() {
      const searchText = this.searchText.toLowerCase().trim();

      console.log(searchText);
      if (searchText === "") {
        this.searchedItems = [];
        this.searchActive = false;
      } else {
        //Initiating a fetch request for search results
        fetch(
          "https://lessonapp3-env.eba-mrkmnxsq.eu-west-2.elasticbeanstalk.com/search/collections/products/" +
            searchText
        )
          .then((response) => {
            if (!response.ok) {
              throw new Error(`HTTP error! Status: ${response.status}`);
            }
            return response.json();
          })
          .then((data) => {
            this.searchedItems = data;
            this.searchActive = true;
          })
          .catch((error) => {
            console.error("Error during search:", error);
          });
      }
    },

    updateUserName(value) {
      this.userName = value;
    },
    updateUserNumber(value) {
      this.userNumber = value;
    },
    activateTests() {
      // Activate the test section
      this.testActive = true;
    },
    reloadPage() {
      // Reload the page
      location.reload();
    },
    async deleteCaches() {
      try {
        // Delete all caches
        const keys = await caches.keys();
        await Promise.all(keys.map((key) => caches.delete(key)));
        alert("Caches deleted successfully.");
      } catch (error) {
        console.error("Error deleting caches:", error);
        alert("Failed to delete caches.");
      }
    },
    unregisterServiceWorkers() {
      // Unregister all service workers
      navigator.serviceWorker
        .getRegistrations()
        .then((registrations) => {
          registrations.forEach((registration) => registration.unregister());
          alert("Service workers unregistered successfully.");
        })
        .catch((error) => {
          console.error("Error unregistering service workers:", error);
          alert("Failed to unregister service workers.");
        });
    },
  },

  computed: {
    //  property that returns the sorted lessons
    sortedLessons() {
      return this.lessons;
    },

    phoneNumberIsValid() {
      return /^[0-9]*$/.test(this.userNumber);
    },
    userNameIsValid() {
      return /^[A-Za-z ]+$/.test(this.userName);
    },

    //determine current view state
    isCheckoutView() {
      return this.lessonView && this.toggleButtonText === "Checkout";
    },
    isBackToLessonsView() {
      return this.lessonView && this.toggleButtonText === "Back to Lessons";
    },
  },
  watch: {
    selectedSortOrder() {
      // track changes in selectedSortOrder
      // If a valid sorting order asc or desc is selected, the sortLessons method is called
      if (["Ascending", "Descending"].includes(this.selectedSortOrder)) {
        this.sortLessons();
      }
    },
  },
};
</script>
