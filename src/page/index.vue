<template>
  <div id="myIndex">
    <el-button @click="areaVisible=true">选择地区</el-button>
    <area-select
      :visible="areaVisible"
      targetId="myIndex"
      @handleAreaClose="handleAreaClose"
      @handleAreaChoose="handleAreaChoose">
    </area-select>
  </div>
</template>

<script>
import AreaSelector from '@/components/AreaSelector.vue'
export default {
  name: '',
  components: {
    'area-select': AreaSelector
  },
  data: function () {
    return {
      areaVisible: false
    }
  },
  created () {},
  mounted () {},
  methods: {
    handleAreaClose () {
      this.areaVisible = false
    },
    handleAreaChoose (params) {
      console.log(params)
    }
  },
  computed: {},
  watch: {}
}
</script>

<style scoped>
</style>
