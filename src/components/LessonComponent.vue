<template>
    <div class="lesson-container">
      <!-- Loop through the lessons and display them -->
      <div v-for="(lesson, index) in lessons" :key="index" class="lesson">
        <i :class="['fas', lesson.icon, 'fa-3x']"></i>
        <p>Subject: {{ lesson.subject }}</p>
        <p>Location: {{ lesson.location }}</p>
        <p>Price: {{ lesson.price }}</p>
        <p>Spaces: {{ lesson.spaces }}</p>
        <button @click="bookLesson(lesson)" :class="{ 'hoverable-button': lesson.spaces > 0 }"
          :disabled="lesson.spaces === 0">
          <i class="fas fa-shopping-cart"></i> Add to cart
        </button>
      </div>
    </div>
</template>

<script>
export default {
  props: ["lessons"],
  methods: {
    bookLesson(lesson) {
      // Emit a custom event to notify the parent component to add the lesson to the cart
      this.$emit("add-item-to-cart", lesson);
      console.log("Lesson added to cart:", lesson);
    }
  }
};
</script>
