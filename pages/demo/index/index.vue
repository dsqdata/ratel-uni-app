<template>
  <list>
    <!-- 注意事项: 不能使用 index 作为 key 的唯一标识 -->
    <cell v-for="(item, index) in dataList" :key="item.id">
      <text>{{item.name}}</text>
    </cell>
  </list>
</template>

<script>
  export default {
    data () {
      return {
        dataList: [{id: "1", name: 'A'}, {id: "2", name: 'B'}, {id: "3", name: 'C'}]
      }
    }
  }
</script>

<style>
	
</style>
