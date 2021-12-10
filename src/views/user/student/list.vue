<template>
  <div class="app-container" />
</template>

<script>
import { mapGetters, mapState } from 'vuex'
import Pagination from '@/components/Pagination'
export default {
  components: { Pagination },
  data() {
    return {
      queryParam: {
        userName: '', // 用户名
        schoolName: '', // 学校名
        className: '' // 班级名
      }
    }
  }
}
</script>
