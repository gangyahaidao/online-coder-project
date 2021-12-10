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
        userName: '',
        schoolName: '',
        className: ''
      }
    }
  }
}
</script>
