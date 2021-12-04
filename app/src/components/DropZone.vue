<template>
  <div
    @dragenter.prevent="toggleActive"
    @dragleave.prevent="toggleActive"
    @dragover.prevent
    @drop.prevent="toggleActive"
    :class="{ 'active-dropzone':active }"
    class="dropzone"
  >
    <span>Drag or Drop File</span>
    <span>OR</span>
    <label for="dropzoneFile">Select File</label>
    <input type="file" id="dropzoneFile">
    
    <div>
      <input type="number" v-model="price">
      <h1 v-if="today === '2021年12月4日'">Today</h1>
      <h1 v-else>Not today</h1>
      <div>値段: {{ price }}</div>
      <div>税込み: {{ priceInTax }}</div>
    </div>
  </div>

</template>

<script>
import { ref } from "vue"
import moment from 'moment'

export default {
  name: 'DropZone',
  setup() {
    const active = ref(false);

    const toggleActive = () => {
      active.value = !active.value;
    };

    return {active, toggleActive}
  },
  data: () => {
    return {
      price: 0,
    }
  },
  computed: {
    priceInTax() {
      const price = parseInt(this.price);
      return (price + price * 0.1) || 0
    },
    today() {
      const date = new Date();
      // console.log(moment(date).format("Y年M月D日"));    
      return moment(date).format("Y年M月D日");
    }
  },
}
</script>

<style scoped lang="scss">
  .dropzone {
    width: 400px;
    height: 500px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    row-gap: 16px;
    border: 2px dashed #41b883;
    background: #fff;
    transition: 0.3s ease all;
    
    label {
      padding: 8px 12px;
      color: #fff;
      background: #41b883;
      transition: 0.3s ease all;
    }

    // input {
    //   display: none;
    // }
  }

  .active-dropzone {
    color: #fff;
    border-color: #fff;
    background: #41b883;

    label {
      background: #fff;
      color: #41b883;
    }
  }
</style>
