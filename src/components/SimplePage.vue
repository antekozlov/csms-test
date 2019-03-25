<template>
<div>
  <h1 class="text-xs-center">{{tableProps.H1Content}}</h1>
  <h2>{{tableProps.TableName}}</h2>
  <div class="scroll-wrapper">
    <table>
      <tr>
        <th v-for="(item, index) in colsOrder" :key="item" class="text-center">
          <div class="icon-wrap">
            <v-icon @click="move(index, -1)" class="icon-move">navigate_before</v-icon>
            <v-icon @click="move(index, 1)" class="icon-move">navigate_next</v-icon>
          </div>
          {{tableProps.ColsTitles[item]}}
        </th>
      </tr>
      <tr v-for="(cols, colsIndex) in tableProps.Data">
        <td v-for="item in colsOrder" @dblclick="editItem(colsIndex, item)" class="table-data text-center">
          <span class="unselected-item" v-if="!(editedItem.index === colsIndex && editedItem.itemName === item)">
            {{cols[item]}}
          </span>
          <span v-else>
            <input type="text" v-model="selectedItem" id="selected-item" @keyup.enter="setSelectedItem" @blur="setSelectedItem">
          </span>
        </td>
      </tr>
    </table>
  </div>

  <v-btn color="blue-grey btn-download" class="white--text" :href="href" :download="download" v-if="showLink">
    Скачать
    <v-icon right dark>get_app</v-icon>
  </v-btn>
</div>
</template>


<script>
import {
  mapState
} from 'vuex';

const name = 'myfile.json'; // имя файла, куда будет записываться измененный json

export default {
  props: ['tableProps'],
  data() {
    return {
      editedItem: {},
      selectedItem: '',
      showLink: null,
      href: '',
      download: null,
    }
  },
  created() {
    document.title = this.tableProps.PageTitle; // меняем заголовок title
    const sortCols = (a, b) => { // функция для фильтрации tableProps
      if (this.tableProps.ColsOrder[a] > this.tableProps.ColsOrder[b]) return 1;
      if (this.tableProps.ColsOrder[a] < this.tableProps.ColsOrder[b]) return -1;
    };
    const colsOrder = Object.keys(this.tableProps.ColsOrder) // берем ключи из ColsOrder (там порядок отображения и он для нас будет ключевым)
      .sort(sortCols) // сортируем
      .filter(item => this.tableProps.ColsShow[item] === 1); // фильтруем
    this.$store.commit('setColsOrder', colsOrder); // записываем в store
  },
  methods: {
    move(index, shift) { // метод для перемещения столбцов
      const colsOrder = [...this.colsOrder]; // копируем текущий объект (иммутабельность)
      const currentItem = colsOrder[index]; // запоминаем текущий
      if (colsOrder[index + shift]) { // проверяем пограничные условия
        const shiftedItem = colsOrder[index + shift];
        colsOrder[index] = shiftedItem;
        colsOrder[index + shift] = currentItem;
        this.$store.commit('setColsOrder', colsOrder); // записываем результат в стор
      }
    },
    editItem(index, itemName) { // редактирование содержимого ячейки
      this.editedItem = {
        index,
        itemName
      }; // координаты текущей ячейки
      this.selectedItem = this.tableProps.Data[index][itemName]; // редактируемое значение
      setTimeout(() => {
        const el = document.getElementById('selected-item');
        if (el) { // фокусим в инпуте. setTimeOut нужен для того, чтобы успел отрисоваться инпут
          el.focus();
        }
      }, 0);
    },
    setSelectedItem() { // записываем изменения в ячейке
      this.$store.commit('setSelectedItem', {
        editedItem: this.editedItem,
        selectedItem: this.selectedItem
      });
      this.editedItem = {}; // обнуляем выбор ячейки
      this.selectedItem = ''; // и редактируемое значение
      this.createJSON(); // создаем json
      this.showLink = true; // показываем кнопку загрузки json
    },
    createJSON() { // создаем json
      const file = new Blob([JSON.stringify(this.$store.state.data)], {
        type: 'text/json'
      });
      this.href = URL.createObjectURL(file);
      this.download = name;
    }
  },
  computed: mapState({ // подтягиваем из стора объект
    colsOrder: state => state.colsOrder,
  }),
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
table {
    border-collapse: collapse;
    background-color: white;
    box-shadow: 0 2px 1px -1px rgba(0,0,0,0.2), 0 1px 1px 0 rgba(0,0,0,0.14), 0 1px 3px 0 rgba(0,0,0,0.12) !important;

    td,
    th {
        width: 150px;
    }
    tr:nth-child(even) {
        background-color: #eeeeee;
    }
}
td,
th {
    padding: 5px;
}
th {
    height: 75px;
}
#selected-item:focus {
    outline: none;
    box-shadow: 0 0 0 1px #1976d2;
}

.icon-move {
    font-size: 15px;
}
.unselected-item {
    user-select: none;
}
h1 {
    margin-top: 30px;
    margin-bottom: 15px;
}
h2 {
    margin-bottom: 15px;
}
.icon-wrap {
    display: flex;
    justify-content: space-between;
    margin-bottom: 5px;
}
.text-center {
    text-align: center;
}
.scroll-wrapper {
    overflow: scroll;
}
.table-data {
    font-size: 12px;
}
.btn-download {
    margin-top: 30px;
}
</style>
