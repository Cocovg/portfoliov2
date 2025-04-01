<template>
  <div class="point-content">
    <h1>Point 7</h1>
    <p>This point represents the balance between different aspects of your journey.</p>
    
    <div class="additional-content">
      <h2>Additional Content</h2>
      <p>You can add any HTML content here, including:</p>
      <ul>
        <li>Lists</li>
        <li><strong>Bold text</strong></li>
        <li><em>Italic text</em></li>
        <li>Links</li>
        <li>Images</li>
        <li>And more!</li>
      </ul>
      <p>Feel free to edit this component to add your own content.</p>
    </div>
  </div>
</template>

<script setup>
// Add any component logic here
</script>

<style scoped>
.point-content {
  padding: 20px;
  color: #ffffff;
}

h1 {
  font-size: 2em;
  margin-bottom: 1em;
  color: #00ffff;
}

h2 {
  font-size: 1.5em;
  margin: 1em 0;
  color: #00ffff;
}

p {
  line-height: 1.6;
  margin-bottom: 1em;
}

ul {
  list-style-type: none;
  padding-left: 20px;
}

li {
  margin-bottom: 0.5em;
  position: relative;
}

li::before {
  content: "•";
  color: #00ffff;
  position: absolute;
  left: -20px;
}

.additional-content {
  margin-top: 2em;
  padding-top: 1em;
  border-top: 1px solid rgba(0, 255, 255, 0.2);
}
</style> 