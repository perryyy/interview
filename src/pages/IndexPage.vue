<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md q-mr-sm" v-if="editId === ''" @click="addUser()">新增</q-btn>
        <q-btn color="primary" class="q-mt-md q-mr-sm" v-if="editId.length > 0" @click="editUser()">更新</q-btn>
        <q-btn color="primary" class="q-mt-md q-mr-sm" v-if="editId.length > 0" @click="clearData()">清除</q-btn>
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
      <q-dialog v-model="alert" persistent >
        <q-card style="width: 300px">
          <q-card-section>
            <div class="text-h6">提醒</div>
          </q-card-section>

          <q-card-section class="q-pt-none">
            姓名與年齡為必填，年齡必須是數字
          </q-card-section>

          <q-card-actions align="right">
            <q-btn flat label="OK" color="primary" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>
      <q-dialog v-model="isShowDeleteDialog" persistent>
        <q-card style="width: 300px">
          <q-card-section>
            <div class="text-h6">提示</div>
          </q-card-section>

          <q-card-section class="q-pt-none">
            是否刪除該筆資料?
          </q-card-section>

          <q-card-actions align="right">
            <q-btn flat label="取消" color="primary" v-close-popup />
            <q-btn flat label="確定" color="primary" v-close-popup @click="deleteUser()"/>
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios'
import { QTableProps } from 'quasar'
import { ref, onMounted } from 'vue'

interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([
  {
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
    sortable: true
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
    sortable: true
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
])

const tempData = ref({
  name: '',
  age: '',
})

const editId = ref('')
const isShowDeleteDialog = ref(false)
function handleClickOption(btn, data) {
  console.log(data)
  switch (btn.status) {
    case 'edit':
      editId.value = data.id
      tempData.value.name = data.name
      tempData.value.age = data.age
    break
    case 'delete':
      editId.value = data.id
      isShowDeleteDialog.value = true
    break
  }
}
onMounted(() => {
  getUserData()
})
const clearData = () => {
  editId.value = ''
  tempData.value.name = ''
  tempData.value.age = 0
}
const alert = ref(false)
const getUserData = () => {
  return new Promise((resolve, reject) => {
    axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
      .then(res => {
        blockData.value = res.data
        resolve(res)
      })
      .catch(err => {
        console.log(err)
        reject(err)
      })
  })
};
const addUser = () => {
  return new Promise((resolve, reject) => {
    const age = Number(tempData.value.age)
    if (!tempData.value.name || !tempData.value.age || isNaN(age)) {
      alert.value = true
      reject(new Error('Missing required fields'))
    } else {
      axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value)
        .then(res => {
          getUserData()
            .then(data => resolve(data))
            .catch(err => reject(err))
        })
        .catch(err => {
          console.log(err)
          reject(err)
        })
    }
  })
}
const editUser = () => {
  return new Promise((resolve, reject) => {
    axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value)
      .then(res => {
        const userIndex = blockData.value.findIndex(user => user.id === editId.value)
        if (userIndex !== -1) {
          blockData.value[userIndex] = { ...blockData.value[userIndex], ...tempData.value }
        }
      })
      .catch(err => {
        console.log(err)
        reject(err)
      })
  })
}
const deleteUser = () => {
  return new Promise((resolve, reject) => {
    axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${editId.value}`)
      .then(res => {
        editId.value = ''
        getUserData()
          .then(data => resolve(data))
          .catch(err => reject(err))
        clearData()
      })
      .catch(err => {
        console.log(err)
        reject(err)
      })
  })
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
