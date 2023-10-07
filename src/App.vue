<template>
<div class="table-container">
  <b-button variant="success" @click="handleAdd()">Добавить</b-button>
  <b-editable-table disableDefaultEdit bordered class="editable-table" v-model="items" :fields="fields">
    <template #cell(edit)="data">
      <BIconPencil class="edit-icon" @click="handleEdit(data)"></BIconPencil>
    </template>
    <template #cell(delete)="data">
      <BIconTrash class="remove-icon" @click="handleDelete(data)"></BIconTrash>
    </template>
  </b-editable-table>
  <div>
    <b-modal ref="my-modal" hide-footer title="Добавить юридическое лицо">
      <div class="d-block text-center">
        <b-form-input class="input" id="input-1" v-model="data.address" type="text" placeholder="Адрес" required></b-form-input>
        <b-form-input class="input" id="input-2" v-model="data.ogrn" type="text" placeholder="Орган" required></b-form-input>
        <div class="input" style="display: flex">
          <b-form-input id="input-3" v-model="data.inn" type="text" placeholder="ИНН" required></b-form-input>
          <b-button @click="getINN" :disabled="data.inn.length !== 10" variant="primary">Загрузить</b-button>
        </div>
        <b-form-input class="input" id="input-4" v-model="data.registrationDate" type="text" placeholder="Дата регистрации" required></b-form-input>
        <b-button @click="onSubmit" variant="primary">Сохранить</b-button>
      </div>
    </b-modal>
  </div>
</div>
</template>

<script>
import {
  v4 as uuidv4
} from 'uuid';

import {
  url,
  token
} from './config';
import BEditableTable from "bootstrap-vue-editable-table";
import {
  BIconTrash,
  BIconPencil,
  BButton,
} from "bootstrap-vue";

export default {
  components: {
    BEditableTable,
    BIconTrash,
    BIconPencil,
    BButton,
  },
  data() {
    return {
      showModal: false,
      data: {
        id: uuidv4(),
        address: '',
        ogrn: '',
        inn: '',
        registrationDate: "",
      },
      fields: [{
          key: "delete",
          label: ""
        },
        {
          key: "address",
          label: "Адрес"
        },
        {
          key: "ogrn",
          label: "Орган"
        },
        {
          key: "inn",
          label: "Инн"
        },
        {
          key: "registrationDate",
          label: "Дата регистрации"
        },
        {
          key: "edit",
          label: ""
        },
      ],
      items: []
    };
  },
  methods: {
    checkLocalStorage() {
      this.items = JSON.parse(localStorage.getItem("items"))
      if(!this.items) this.items = []
    },
    async getINN() {
      const options = {
        method: "POST",
        mode: "cors",
        headers: {
          "Content-Type": "application/json",
          "Accept": "application/json",
          "Authorization": "Token " + token
        },
        body: JSON.stringify({
          query: this.data.inn
        })
      }
      let response = await fetch(url, options);
      if (response.status === 200) {
        let json = await response.json();
        if(json.suggestions.length){
          const data = json.suggestions[0].data;
          this.data = {
            id: uuidv4(),
            address: data.address.value,
            ogrn: data.ogrn,
            inn: data.inn,
            registrationDate: data.okved_type,
          }
        } else {
          alert("По такому ИНН ничего не найдено");
        }

      } else {
        alert("Ошибка HTTP: " + response.status);
      }
    },
    handleAdd() {
      this.data = {
        id: uuidv4(),
        address: '',
        ogrn: '',
        inn: '',
        registrationDate: "",
      };
      this.$refs['my-modal'].show();
    },
    handleEdit(data) {
      this.data = data.item;
      this.$refs['my-modal'].show();
    },
    handleDelete(data) {
      this.items = this.items.filter(element => element.id !== data.id);
      localStorage.setItem("items", JSON.stringify(this.items))
      this.items = JSON.parse(localStorage.getItem("items"))
    },
    onSubmit() {
      const findItem = this.items.find(el => el.id === this.data.id)
      if (!findItem) {
        this.items.push(this.data)
      }
      this.items = this.items.map(element => {
        if (element.id === this.data.id) {
          return {
            ...this.data
          }
        }
        return element
      });
      this.data = {
        id: uuidv4(),
        address: '',
        ogrn: '',
        inn: '',
        registrationDate: "",
      };
      localStorage.setItem("items", JSON.stringify(this.items))
      this.$refs['my-modal'].hide();
      this.items = JSON.parse(localStorage.getItem("items"))
    },
  },
  mounted() {
    this.checkLocalStorage();
  },
};
</script>

<style>
.table-container {
  margin: 10px;
}

table.editable-table {
  margin-top: 10px;
}

table.editable-table td {
  vertical-align: middle;
}

.editable-table .data-cell {
  padding: 8px;
  vertical-align: middle;
}

.editable-table .custom-checkbox {
  width: 50px;
}

.remove-icon {
  color: red;
  cursor: pointer;
  font-size: 20px;
}

.edit-icon {
  color: rgb(4, 83, 158);
  cursor: pointer;
  font-size: 20px;
}

.name-col {
  width: 120px;
}

.department-col {
  width: 150px;
}

.age-col {
  width: 100px;
}

.date-col {
  width: 200px;
}

.is-active-col {
  width: 100px;
}

.input {
  margin-bottom: 15px;
}
</style>
