<template>
  <div class="drop-zone" @drop="onDrop($event,1)" @dragenter.prevent @dragover.prevent>
    <div class="drag-el" v-for="item in getList(1)" :key="item.id" draggable="true" @dragstart="startDrag($event,item)">
      {{ item.title }}
    </div>
  </div>

  <div class="drop-zone" @drop="onDrop($event,2)" @dragenter.prevent @dragover.prevent>
    <div class="drag-el" v-for="item in getList(2)" :key="item.id" draggable="true" @dragstart="startDrag($event,item)">
      {{ item.title }}
    </div>
  </div>
</template>
<script>
import {ref} from 'vue'

export default {
  setup() {
    const items = ref([
      {id: 0, title: 'Item A', list: 1},
      {id: 1, title: 'Item B', list: 1},
      {id: 2, title: 'Item C', list: 2},
    ])
    const getList = (list) => {
      return items.value.filter((item) => item.list == list)
    }
    const startDrag = (event, item) => {
      console.log(item)
      event.dataTransfer.dropEffect = 'move'
      event.dataTransfer.effectAllowed = 'move'
      event.dataTransfer.setData('itemID', item.id)
    }
    const onDrop = (event, list) => {
      const itemID = event.dataTransfer.getData('itemID')
      const item = items.value.find((item) => item.id == itemID)
      item.list = list
    }
    return {
      getList,
      onDrop,
      startDrag
    }
  }
}
</script>


<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.drop-zone {
  width: 50%;
  margin: 50px auto;
  background: #ecf0f1;
  padding: 10px;
  min-height: 10px;
}

.drag-el {
  background: #3498db;
  color: #fff;
  padding: 5px;
  margin-bottom: 10px;
}

.drag-el:nth-last-of-type(1) {
  margin-bottom: 0;
}
</style>
