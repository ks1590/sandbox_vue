<template>
  <div class="dropzone">
    <span>Drag or Drop File</span>
    <span>OR</span>
    <label for="dropzoneFile">Select File</label>
    <input type="file" id="dropzoneFile">
  </div>
</template>

<script>
export default {
  name: 'DropZone',
}
</script>

<style scoped lang="scss">
  .dropzone {
    width: 400px;
    height: 200px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    row-gap: 16px;
    border: 2px dashed #41b883;
    background: #fff;
    transition: 0.3s ease all;
  }

  label {
    padding: 8px 12px;
    color: #fff;
    background: #41b883;
    transition: 0.3s ease all;
  }

  input {
    display: none;
  }
</style>
