<template>
  <div v-if="!searchActive" id="cart-checkout-button">
    <h2>Shopping Cart</h2>
    <div v-if="cart.length === 0">
      <p>Your cart is empty.</p>
    </div>
    <!-- "Checkout" button -->
    <button
      class="cart-checkout-button"
      @click="checkout"
      :disabled="
        cart.length === 0 ||
        !phoneNumberIsValid ||
        !userNameIsValid ||
        !userName ||
        !userNumber
      "
    >
      <i class="fas fa-shopping-cart"></i> Checkout({{ cartCount }})
    </button>

    <!-- Display the checkout completion message -->
    <div v-if="checkoutMessage" class="checkout-message">
      {{ checkoutMessage }}
    </div>

    <!-- User information input fields -->
    <div>
      <label for="userName">Name:</label>
      <input
        type="text"
        id="userName"
        :value="userName"
        @input="updateUserName"
        placeholder="Letters-only"
        pattern="^[A-Za-z\s\W]+$"
      />
    </div>
    <div>
      <label for="userNumber">Phone :</label>
      <input
        type="tel"
        id="userNumber"
        :value="userNumber"
        @input="updateUserNumber"
        pattern="[0-9]*"
        placeholder="Numbers-only"
      />
    </div>

    <!-- Cart items -->
    <div
      v-for="(item, index) in cart"
      :key="index"
      class="lesson"
      id="cart-page"
    >
      <p>Subject: {{ item.subject }}</p>
      <p>Location: {{ item.location }}</p>
      <p>Price: {{ item.price }}</p>
      <p>Spaces: {{ item.spaces }}</p>
      <button @click="removeFromCart(index)">Remove from cart</button>
    </div>
  </div>
</template>

<script>
export default {
  // Props received from the parent component
  props: [
    "cart",
    "searchActive",
    "phoneNumberIsValid",
    "userNameIsValid",
    "checkoutMessage",
    "cartCount",
    "userName",
    "userNumber",
  ],
  data() {
    return {};
  },
  methods: {
    // Emit an event to notify the parent component to remove an item from the cart
    removeFromCart(index) {
      this.$emit("remove-item-from-cart", index);
    },
    // Emit an event to notify the parent component to update the user name
    updateUserName(event) {
      this.$emit("update-user-name", event.target.value);
    },
    // Emit an event to notify the parent component to update the user number
    updateUserNumber(event) {
      this.$emit("update-user-number", event.target.value);
    },
    // Emit an event to notify the parent component to initiate the checkout process
    checkout() {
      this.$emit("checkout");
    },
  },
};
</script>
