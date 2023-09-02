<template>
  <div class="container">
    <div class="page-head">
      <div class="optear">
        <el-button
          type="info"
          :loading="formConfs.templateLoading"
          @click="downTemp"
          >下载导入模板</el-button
        >
        <el-button type="primary" @click="createTemplate">批量导入设备</el-button>
        <el-button type="primary" @click="createDev">新增设备</el-button>
        <el-button type="success" @click="testConnect_temp">测试连接</el-button>
        <el-button type="success" @click="deviceStart_temp">采集信息</el-button>

      </div>
      <div class="query">
        <el-form-item label="设备名" size="default">
          <el-input v-model.trim="formConfs.hostname"></el-input>
        </el-form-item>
        <el-button type="primary" @click="getDevHostData">查询</el-button>
        <el-button type="primary" @click="refreshHandle">刷新</el-button>
      </div>
    </div>
    <div class="page-table">
      <el-table :data="tableData" stripe border height="500px">
        <el-table-column prop="hostname" label="主机名" width="180" />
        <el-table-column prop="device_type" label="设备类型" width="180" />
        <el-table-column prop="templates_name" label="模板" width="180" />
        <el-table-column prop="username" label="用户名" />
        <el-table-column prop="protocol" label="协议" />
        <el-table-column prop="port" label="端口" />
        <el-table-column label="操作">
          <template #default="scope">
            <el-button
              type="primary"
              @click="editHandle(scope.row.hostname, scope.row.id)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              @click="devDeviceitem(scope.row.id, scope.$index)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="pagination">
      <div class="title">
        <span>消息</span>
        <el-button type="danger" @click="formConfs.message = []"
          >清空</el-button
        >
      </div>
      <el-pagination
        v-model:current-page="queryParams.pageNum"
        v-model:page-size="queryParams.pageSize"
        :page-sizes="[10, 20, 50, 100]"
        layout="total, sizes, prev, pager, next"
        :total="tableTotal"
        @size-change="getList"
        @current-change="getList"
      />
    </div>

    <div class="socket-msg">
      <div class="msg" v-for="(item, index) in formConfs.message" :key="index">
        {{ item }}
      </div>
    </div>

    <el-dialog
      title="新增设备"
      width="1000px"
      destroy-on-close
      v-model="formConfs.visible"
    >
      <el-form
        :model="createForm"
        ref="createFormRef"
        :rules="formRules"
        label-position="top"
      >
        <el-row :gutter="40">
          <el-col :span="12">
            <el-form-item label="主机名" prop="hostname">
              <el-input v-model="createForm.hostname" autocomplete="off" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="设备类型" prop="device_type">
              <el-input v-model="createForm.device_type" autocomplete="off" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="用户名" prop="username">
              <el-input v-model="createForm.username" autocomplete="off" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="协议" prop="protocol">
              <el-input v-model="createForm.protocol" autocomplete="off" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="端口" prop="port">
              <el-input v-model="createForm.port" autocomplete="off" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="模板名字" prop="templates_name">
              <el-input
                v-model="createForm.templates_name"
                autocomplete="off"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="密码" prop="password">
              <el-input v-model="createForm.password" autocomplete="off" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="enable密码" prop="super_pw">
              <el-input v-model="createForm.super_pw" autocomplete="off" />
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="cancelCreateForm">取消</el-button>
          <el-button type="primary" @click="confirmCreateForm">
            确定
          </el-button>
        </span>
      </template>
    </el-dialog>

    <el-dialog
      title="新增设备"
      width="1000px"
      destroy-on-close
      v-model="formConfs.visible_temp"
    >
      <el-upload
        class="upload-demo"
        :on-success="fileSuccess"
        drag
        :action="formConfs.up_file_url"
        multiple
      >
        <el-icon class="el-icon--upload"><upload-filled /></el-icon>
        <div class="el-upload__text">把文件放在这里或<em>点击上传</em></div>
        <template #tip>
          <div class="el-upload__tip">支持 xlsx 文件</div>
        </template>
      </el-upload>
    </el-dialog>

    <el-dialog
      title="测试连接"
      width="500px"
      destroy-on-close
      v-model="formConfs.testConnect_temp"
    >
      <el-form
        :model="testDeviceForm"
        ref="createTestFormRef"
        :rules="testDeviceFormRules"
        label-position="top"
      >
        <el-row :gutter="12">
          <el-col :span="12">
            <el-form-item label="模式" prop="pattern">
              <el-select v-model="testDeviceForm.pattern" placeholder="请选择模式">
                <el-option label="自动" :value="true"></el-option>
                <el-option label="自定义" :value="false"></el-option>
              </el-select>
            </el-form-item>
          </el-col>

          <el-col :span="12" v-if="!testDeviceForm.pattern">
            <el-form-item label="主机地址" prop="hostname" >
              <el-input v-model="testDeviceForm.hostname" autocomplete="off" />
            </el-form-item>
          </el-col>

          <el-col :span="12"  v-if="!testDeviceForm.pattern">
            <el-form-item label="端口号" prop="port" >
              <el-input v-model="testDeviceForm.port" autocomplete="off" />
            </el-form-item>
           </el-col>
          </el-row>
        </el-form>

     <template #footer>
       <span class="dialog-footer">
         <el-button @click="cancelTestCreateForm">取消</el-button>
         <el-button type="primary" @click="confirmTestCreateForm">
           确定
         </el-button>
       </span>
     </template>
    </el-dialog>

    <el-dialog
      title="采集信息"
      width="500px"
      destroy-on-close
      v-model="formConfs.deviceStart_temp"
    >
      <el-form
        :model="deviceStartForm"
        ref="deviceStartFormRef"
        :rules="deviceStartFormRules"
        label-position="top"
      >
        <el-row :gutter="12">
          <el-col :span="12">
            <el-form-item label="模式" prop="execute_all">
              <el-select v-model="deviceStartForm.execute_all" placeholder="请选择模式">
                <el-option label="自动" :value="true"></el-option>
                <el-option label="自定义" :value="false"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12" >
            <el-form-item label="采集数据保存方式" prop="content_process" >
              <el-select v-model="deviceStartForm.content_process" placeholder="请选择模式">
                <el-option label="备份配置文件" :value="contentProcessEnum.backup_txt"></el-option>
                <el-option label="保存文本格式" :value="contentProcessEnum.txt"></el-option>
                <el-option label="保存内容单表excel" :value="contentProcessEnum.excel"></el-option>
                <el-option label="保存多表excel" :value="contentProcessEnum.excel_sheet"></el-option>
                <el-option label="自定义保存格式" :value="contentProcessEnum.custom"></el-option>
              </el-select>
            </el-form-item>
           </el-col>
          <el-col :span="12" v-if="!deviceStartForm.execute_all">
            <el-form-item label="执行方法名称" prop="target_func_name" >
              <el-input v-model="deviceStartForm.target_func_name" autocomplete="off" />
            </el-form-item>
          </el-col>
          </el-row>
        </el-form>

     <template #footer>
       <span class="dialog-footer">
         <el-button @click="canceldeviceStartForm">取消</el-button>
         <el-button type="primary" @click="confirmdeviceStart">
           确定
         </el-button>
       </span>
     </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ElMessage } from 'element-plus'
import { UploadFilled } from '@element-plus/icons-vue'
import { onMounted, reactive, ref,onBeforeUnmount } from '@vue/runtime-core'

import {
  addDev,
  delDevice,
  getDevHost,
  editDevDetail,
  getDevList,
  getDevDetail,
  downTemplate,
  getDevListTotal,
  testConnect,
  deviceStart,
} from '../request/index'


let baseurl=process.env.NODE_ENV === 'development' ? '127.0.0.1:18888' : location.host
let tableTotal = ref(0)
let tableData = reactive([])
let queryParams = reactive({
  pageSize: 9,
  pageNum: 0,
})

let createFormRef = ref()
let createForm = reactive({
  hostname: 'ip/域名',
  device_type: 'huawei',
  username: new Date().getTime(),
  protocol: 'ssh',
  port: 22,
  templates_name: 'templates_name',
  password: 'password',
  super_pw: '',
})

let createTestFormRef = ref()
let testDeviceForm = reactive({
    pattern: true,
    hostname: 'ip/域名',
    port: 22,
})
let testDeviceFormRules=reactive({
  pattern:[
    {
      required: true,
      message: '该值不能为空',
      trigger: 'change',
    }
  ],
  hostname:[
    {
      required: false,
      message: '',
      trigger: 'change',
    }
  ],
  port:[
    {
      required: false,
      message: '',
      trigger: 'change',
    }
  ],
})

let deviceStartFormRef=ref()
let deviceStartForm=reactive({
  execute_all: true,
  target_func_name: 'backup_config',
  content_process: '备份配置文件'
})
let deviceStartFormRules=reactive({
  execute_all:[
    {
      required: true,
      message: '该值不能为空',
      trigger: 'change',
    }
  ],
  target_func_name:[
    {
      required: false,
      message: '',
      trigger: 'change',
    }
  ],
  content_process:[
    {
      required: false,
      message: '',
      trigger: 'change',
    }
  ],
})

let contentProcessEnum={
  backup_txt:'备份配置文件',
  txt:'保存内容txt',
  excel:'保存内容单表excel',
  excel_sheet:'保存多表excel',
  custom:'自定义保存格式',
}

let formRules = reactive({
  hostname: [
    {
      required: true,
      message: '该值不能为空',
      trigger: 'change',
    },
  ],
  device_type: [
    {
      required: true,
      message: '该值不能为空',
      trigger: 'change',
    },
  ],
  username: [
    {
      required: true,
      message: '该值不能为空',
      trigger: 'change',
    },
  ],
  protocol: [
    {
      required: true,
      message: '该值不能为空',
      trigger: 'change',
    },
  ],
  port: [
    {
      required: true,
      message: '该值不能为空',
      trigger: 'change',
    },
  ],
  templates_name: [
    {
      required: false,
      message: '',
      trigger: 'change',
    },
  ],
  password: [
    {
      required: true,
      message: '该值不能为空',
      trigger: 'change',
    },
  ],
  super_pw: [
    {
      required: false,
      message: '',
      trigger: 'change',
    },
  ],
})

let formConfs = reactive({
  up_file_url:(process.env.NODE_ENV === 'development' ? '/api':'/API')+'/upload_dev_profile',
  type: 'create',
  hostname: '',
  visible: false,
  visible_temp: false,
  startLoading: false,
  templateLoading: false,
  testStartLoading: false,
  getIpStatus:false,
  getBackupConfig:false,
  testConnect_temp:false,
  deviceStart_temp:false,
  message: [],
})

onMounted(() => {
  console.log(location.hostname)
  getListTotal()
  getList()
  connectWebSocket()
})

const createDev = () => {
  formConfs.visible = true
  formConfs.type = 'create'
}
const createTemplate = () => {
  formConfs.visible_temp = true
}
const testConnect_temp = () => {
  formConfs.testConnect_temp = true
}
const deviceStart_temp = () =>{
  formConfs.deviceStart_temp=true
}

const fileSuccess = (data) => {
  formConfs.visible_temp = false
  if (data.msg === 'ok') {
    ElMessage.success('新增成功')
  } else {
    ElMessage.error('新增失败')
  }
}

const refreshHandle = () => {
  formConfs.hostname = ''
  queryParams.pageNum = 0
  getList()
}

const cancelCreateForm = () => {
  formConfs.visible = false
  createFormRef.value.resetFields()
}
const confirmCreateForm = () => {
  createFormRef.value.validate((valid, fields) => {
    if (valid) {
      submitData().then(() => {
        cancelCreateForm()
      })
    }
  })
}
const cancelTestCreateForm = () => {
  formConfs.testConnect_temp = false
  if (createTestFormRef.value) {
    createTestFormRef.value.resetFields()
  }
}

const confirmTestCreateForm = () => {
  createTestFormRef.value.validate((valid, fields) => {
    if (valid) {
      createTestForm().then(() => {
        cancelTestCreateForm()
      })
    }
  })
}

const createTestForm = async () => {
  console.log(testDeviceForm)
  try {
    let data = await testConnect(testDeviceForm)
    if (data.msg === 'ok') {
      tableData.splice(index, 1)
      ElMessage.success('操作成功')
    } else {
      ElMessage.error(data.msg)
    }
  } catch (error) {
    console.log(error)
  }
}
const canceldeviceStartForm = () => {
  formConfs.deviceStart_temp = false
  if (deviceStartFormRef.value) {
    deviceStartFormRef.value.resetFields()
  }
}
const confirmdeviceStart = () => {
  deviceStartFormRef.value.validate((valid, fields) => {
    if (valid) {
      deviceStart_func().then(() => {
        canceldeviceStartForm()
      })
    }
  })
}

const deviceStart_func = async () => {
  console.log(deviceStartForm)
  try {
    let data = await deviceStart(deviceStartForm)
    if (data.msg === 'ok') {
      tableData.splice(index, 1)
      ElMessage.success('操作成功')
    } else {
      ElMessage.error(data.msg)
    }
  } catch (error) {
    console.log(error)
  }
}

const editHandle = async (hostname, id) => {
  formConfs.id = id
  formConfs.type = 'edit'
  formConfs.visible = true
  try {
    let data = await getDevDetail(hostname)
    console.log(data)
    if (data.id) {
      for (const i in data) {
        if (createForm.hasOwnProperty(i)) {
          createForm[i] = data[i]
        }
      }
    } else {
      ElMessage.error(data.msg)
    }
  } catch (error) {
    console.log(error)
  }
}

const getList = async () => {
  try {
    let data = await getDevList(queryParams.pageNum, queryParams.pageSize)
    if (data && Array.isArray(data)) {
      tableData.splice(0, tableData.length)
      tableData.push(...data)
    }
  } catch (error) {
    console.log(error)
  }
}
const getDevHostData = async () => {
  try {
    let data = await getDevHost(formConfs.hostname)
    if (data) {
      tableData.splice(0, tableData.length)
      tableData.push(data)
      tableTotal.value = tableData.length
    } else {
      ElMessage.error(data.detail)
    }
  } catch (error) {
    console.log(error)
  }
}

const getListTotal = async () => {
  try {
    let data = await getDevListTotal()
    tableTotal.value = data
  } catch (error) {
    console.log(error)
  }
}

const devDeviceitem = async (id, index) => {
  console.log(id, index)
  try {
    let data = await delDevice(id)
    if (data.msg === 'ok') {
      tableData.splice(index, 1)
      ElMessage.success('操作成功')
    } else {
      ElMessage.error(data.msg)
    }
  } catch (error) {
    console.log(error)
  }
}

const submitData = async () => {
  if (formConfs.type === 'create') {
    createDevice()
  } else {
    editDevice()
  }
}

const createDevice = async () => {
  try {
    let data = await addDev(createForm)
    if (data.id) {
      ElMessage.success('新增成功')
    } else {
      ElMessage.error(data.msg)
    }
  } catch (error) {
    console.log(error)
  }
}
const editDevice = async () => {
  try {
    let data = await editDevDetail({ ...createForm, id: formConfs.id })
    if (data.id) {
      ElMessage.success('新增成功')
    } else {
      ElMessage.error(data.msg)
    }
  } catch (error) {
    console.log(error)
  }
}


// const startDevice = async () => {
//   try {
//     formConfs.startLoading = true
//     let data = await startDev()
//     if (data.msg === 'ok') {
//       ElMessage.success('启动成功')
//       initSocket()
//     } else {
//       ElMessage.error(data.msg)
//     }
//   } catch (error) {
//     console.log(error)
//   } finally {
//     // formConfs.startLoading = false
//   }
// }


// const startBackupConfig = async () => {
//   try {
//     formConfs.getBackupConfig = true
//     let data = await getBackupConfig()
//     if (data.msg === 'ok') {
//       ElMessage.success('启动备份配置文件成功')
//       backupConfig()
//     } else {
//       ElMessage.error(data.msg)
//     }
//   } catch (error) {
//     console.log(error)
//   } finally {
//     // formConfs.startLoading = false
//   }
// }

const downTemp = async () => {
  try {
    formConfs.templateLoading = true
    let res = await downTemplate()
    if (res) {
      const blob = new Blob([res.data])
      let filename = res.headers['content-disposition']
        .split('=')[1]
        .split('"')[1]
      console.log(filename)
      let elink = document.createElement('a')
      elink.download = filename
      elink.style.display = 'none'
      elink.href = URL.createObjectURL(blob)
      document.body.appendChild(elink)
      elink.click()
      URL.revokeObjectURL(elink.href)
      document.body.removeChild(elink)
      ElMessage.success('下载成功')
    } else {
      ElMessage.error(res.data.msg)
    }
  } catch (error) {
    console.log(error)
  } finally {
    formConfs.templateLoading = false
  }
}

const testStartDevice = () => {
  formConfs.testStartLoading = true
  let socket = new WebSocket(`ws://${baseurl}/API/ws/ConnectTest`)
  socket.onclose = () => {
    formConfs.testStartLoading = false
    console.log('ipstatus onclose')
  }
  socket.onerror = () => {
    formConfs.testStartLoading = false
    console.log('ipstatus onerror')
  }
  socket.onopen = () => {
    console.log('ipstatus onopen')
  }
  socket.onmessage = (msg) => {
    console.log('ipstatus onmessage', msg)
    formConfs.message.push(msg.data)
  }
}

const initSocket = () => {
  let socket = new WebSocket(`ws://${baseurl}/API/ws/start`)
  socket.onclose = () => {
    formConfs.startLoading = false
    socket.close()
    console.log('onclose')
  }
  socket.onerror = () => {
    formConfs.startLoading = false
    console.log('onerror')
  }
  socket.onopen = () => {
    console.log('onopen')
  }
  socket.onmessage = (msg) => {
    // console.log('onmessage', msg)
    formConfs.message.push(msg.data)
  }
}

const backupConfig = () => {
  formConfs.getBackupConfig = true
  console.log(location.href);
  let socket = new WebSocket(`ws://${baseurl}/API/ws/BackupConfig`)
  socket.onclose = () => {
    formConfs.getBackupConfig = false
    console.log('backupconfig onclose')
  }
  socket.onerror = () => {
    formConfs.getBackupConfig = false
    console.log('backupconfig onerror')
  }
  socket.onopen = () => {
    console.log('backupconfig onopen')
  }
  socket.onmessage = (msg) => {
    // console.log('backupconfig onmessage', msg)
    formConfs.message.push(msg.data)
  }
}
const websocket = ref(null);
const websocketStatus = ref('Connecting...');
const connectWebSocket = () => {
  // 创建 WebSocket 连接
  websocket.value = new WebSocket('ws://127.0.0.1:18887/API/ws');

  // 监听 WebSocket 事件
  websocket.value.onopen = () => {
    websocketStatus.value = 'Connected';
    // websocket.value.send('连接成功啦！')
    // WebSocket 连接成功后，你可以在这里发送消息或执行其他操作
  };

  websocket.value.onerror = () => {
    websocketStatus.value = 'Error';
  };

  websocket.value.onclose = () => {
    websocketStatus.value = 'Disconnected';
  };
  websocket.value.onmessage = (event) => {
    const receivedData = JSON.parse(event.data);
    if (receivedData.msg === "ok") {
    console.log(event.data); // 如果是心跳消息，打印到控制台
  } else {
    // 否则将消息添加到 formConfs.message 数组
    formConfs.message.push(receivedData.msg);
  }
  };
};

// onMounted(() => {
//   // 在组件挂载时执行 WebSocket 连接
//   connectWebSocket();
// });
const closeWebSocket = () => {
  console.log(websocket.value)
  if (websocket.value && websocket.value.readyState === WebSocket.OPEN) {
    console.log('-------执行了关闭方法')
    // websocket.value.send('close')
    websocket.value.close();
  }
};

// onBeforeUnmount(() => {
//   // 在页面卸载前关闭 WebSocket 连接
//   console.log('刷新触发')
//   closeWebSocket();
// });
// window.addEventListener('beforeunload', () => {
//   // 在页面即将刷新时关闭 WebSocket 连接
//   closeWebSocket();
// });
window.addEventListener('unload', () => {
  // 在页面即将刷新时关闭 WebSocket 连接
  closeWebSocket();
});
</script>

<style lang="less" scoped>
.container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 20px 100px;
  background: #f6f6f6;

  .page-head {
    height: 100px;
    display: flex;
    align-items: center;
    justify-content: space-between;

    .query {
      display: flex;
      .el-button {
        margin-left: 20px;
      }
      .el-form-item {
        margin: 0;
      }
    }
  }

  .pagination {
    display: flex;
    align-items: center;
    justify-content: space-between;

    .title {
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: space-between;
      span {
        display: inline-block;
        font-size: 22px;
        line-height: 100%;
        letter-spacing: 2px;
        margin: 0 20px 0 0;
      }
    }
  }
  .el-pagination {
    margin: 20px 0;
  }

  .socket-msg {
    flex: 1;
    width: 100%;
    overflow: auto;
    background: white;
    border: 1px solid #c2bbbb;
    .msg {
      padding: 0 20px;
      font-size: 14px;
      line-height: 30px;
    }
    .msg:hover {
      background: #f6f6f6;
    }
  }
}
</style>
