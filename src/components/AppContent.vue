<template>
<div>
  <v-container>
    <v-btn color="primary" @click="getData">Обработать данные</v-btn>
    <simple-page :tableProps="props" v-if="props"></simple-page>
    <div v-if="!!error">Ошибка при загрузке данных</div>
  </v-container>
</div>
</template>

<script>
import SimplePage from '@/components/SimplePage';
import {
  mapState
} from 'vuex';

export default {
  data() {
    return {}
  },
  methods: {
    getData() {
      // метод загрузки данных с сервера
      this.$store.dispatch('getData');
    }
  },
  computed: mapState({ // из стора подгружаем свойства props и error . пропсы передаем в simple-page пропсами
    props: state => state.data,
    error: state => state.error,
  }),
  components: {
    SimplePage
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">

</style>
