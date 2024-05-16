<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="handleAdd">新增</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}

interface NewData {
  name: string;
  age: number;
}
const blockData = ref([
  {
    id: 0,
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});

function handleClickOption(btn, data) {
  if (btn.status === 'edit') {
    const updatedData = {
      id: data.id,
      name: tempData.value.name,
      age: + tempData.value.age,
    };

    axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest/', updatedData)
      .then(response => {
        console.log('編輯成功:', response.data);
        axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
          .then(response => {
            console.log(response.data);
            blockData.value = response.data;
          })
          .catch(error => {
            console.error('Error fetching data:', error);
        });

        const index = blockData.value.findIndex(item => item.id === data.id);
        if (index !== -1) {
          blockData.value.splice(index, 1, response.data);
        }
      })
      .catch(error => {
        console.error('編輯失敗:', error);
      });
  } else if (btn.status === 'delete') {
    if (confirm('確定要刪除嗎？')) {
      axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`)
        .then(response => {
          console.log(response.data);
          blockData.value = blockData.value.filter(item => item.id !== data.id);
        })
        .catch(error => {
          console.error('Error deleting data:', error);
        });
    }
  }
}

axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
  .then(response => {
    console.log(response.data);
    blockData.value = response.data;
  })
  .catch(error => {
    console.error('Error fetching data:', error);
});

function handleAdd() {
  const newData: NewData = {
  name: tempData.value.name,
  age: + tempData.value.age,
};

  blockData.value.push(newData);

  axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', newData)
    .then(response => {
      console.log('新增成功:', response.data);

      tempData.value = {
        name: '',
        age: '',
      };
    })
    .catch(error => {
      console.error('新增失敗:', error);
    });
}



</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
