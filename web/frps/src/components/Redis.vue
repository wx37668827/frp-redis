<template>

    <div>
        <el-page-header :icon="null" style="width: 100%; margin-left: 30px; margin-bottom: 20px">
            <template #title>
                <span>IP白名单</span>
            </template>
            <template #content>frpc.exe客户端连接不受白名单IP影响</template>
            <template #extra>
                <div class="flex items-center" style="margin-right: 30px">

                    <el-button @click="add">新增</el-button>
                </div>
            </template>
        </el-page-header>

        <el-table :data="data" :default-sort="{ prop: 'name', order: 'ascending' }" style="width: 100%">

            <el-table-column label="IP" prop="ip" sortable> </el-table-column>
            <el-table-column label="过期时间" prop="expire_at" sortable> </el-table-column>


            <el-table-column label="操作" width="100">
      <template #default="scope">
        <el-button type="danger" size="small" @click="deleteIp(scope.row.ip)">
          删除
        </el-button>
      </template>
    </el-table-column>
        </el-table>

    </div>


    <el-dialog title="添加 IP 白名单" v-model="visible" width="400px" @close="resetForm">
        <el-form :model="form" label-width="80px">
            <el-form-item label="IP地址" :rules="[{ required: true, message: '请输入IP' }]">
                <el-input v-model="form.ip" placeholder="请输入 IP 地址" />
            </el-form-item>

            <el-form-item label="过期时间">
                <el-select v-model="form.expireOption" placeholder="请选择">
                    <el-option label="永不过期" value="0" />
                    <el-option label="2 天" value="2" />
                    <el-option label="30 天" value="30" />
                    <el-option label="自定义" value="custom" />
                </el-select>
            </el-form-item>

            <el-form-item v-if="form.expireOption === 'custom'" label="自定义天数">
                <el-input v-model="form.customDays" placeholder="请输入天数" type="number" min="1" />
            </el-form-item>
        </el-form>

        <template #footer>
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="submit">确定</el-button>
        </template>
    </el-dialog>

</template>

<script setup lang="ts">
import { ref } from 'vue'
import { ElMessage ,ElMessageBox} from 'element-plus'

let data = ref([])
const visible = ref(false)
const form = ref({
    ip: '',
    expireOption: '2',
    customDays: ''
})

const add = () => {

    
    visible.value = true;

}




function resetForm() {
    form.value = {
        ip: '',
        expireOption: '2',
        customDays: ''
    }
}



function submit() {
    if (!form.value.ip.trim()) {
        ElMessage.error('请输入 IP 地址')
        return
    }

    let expireDays = 0
    if (form.value.expireOption === 'custom') {
        const days = parseInt(form.value.customDays)
        if (isNaN(days) || days <= 0) {
            ElMessage.error('请输入有效的自定义天数')
            return
        }
        expireDays = days
    } else {
        expireDays = parseInt(form.value.expireOption)
    }

    fetch('../api/redis/addip', {
        method: 'POST',
        credentials: 'include',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            ip: form.value.ip.trim(),
            expire_days: expireDays
        })
    })
        .then(res => res.json())
        .then(json => {
            ElMessage.success('添加成功')
            visible.value = false
            // 例如刷新列表：
            // emit("refresh")
            fetchData();
        })
        .catch(err => {
            ElMessage.error('添加失败: ' + err.message)
        })
}

function deleteIp(ip: string) {
  ElMessageBox.confirm(`确认删除 IP：${ip}？`, '警告', {
    type: 'warning'
  }).then(() => {
    fetch('../api/redis/delip', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ ip })
    }).then(res => res.json())
      .then(() => {
        ElMessage.success('已删除')
        fetchData()
      })
      .catch(err => {
        ElMessage.error('删除失败: ' + err.message)
      })
  }).catch(() => {
    // 用户取消
  })
}


const fetchData = () => {
    fetch('../api/redis/whitelist', { credentials: 'include' })
        .then((res) => {
            return res.json()
        })
        .then((json) => {

            console.log(json)

            if (json.whitelist?.length > 0) {
                data.value = json.whitelist
            }
            else
            {
                data.value=[]
            }

        })
}
fetchData()
</script>

<style></style>
