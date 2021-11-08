<template>
  <div class="home">
    <h1>DropZone</h1>
    <DropZone/>
    <span class="file-info">File: {{ dropzoneFile.name }}</span>
  </div>
</template>

<script>
// @ is an alias to /src
import DropZone from '@/components/DropZone.vue'
import { ref } from "vue";

export default {
  name: 'Home',
  components: {
    DropZone
  },
  setup() {
    let dropzoneFile = ref("");

    return { dropzoneFile }
  }
}
</script>

<style lang="scss" scoped>
  .home {
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background: #f1f1f1;

    h1 {
      font-size: 40px;
      margin-bottom: 32px;
    }

    .file-info {
      margin-top: 32px;
    }
  }
</style>
