<template>
  <div class="email-container">
    <div class="email-header">
      <div class="compose-btn">
        <RouterLink to="/editer" class="compose-link">+{{ lang.compose }}</RouterLink>
      </div>
      <div class="email-title">{{ groupStore.name }}</div>
      <div class="email-actions">
        <el-button @click="del" size="small" type="danger">{{ lang.del_btn }}</el-button>
        <el-button @click="markRead" size="small" type="primary">{{ lang.read_btn }}</el-button>
        <el-dropdown style="margin-left: 12px;">
          <el-button size="small" type="success">
            {{ lang.move_btn }}
            <el-icon class="el-icon--right">
              <EpArrowDownBold />
            </el-icon>
          </el-button>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item @click="move(group.id)" v-for="group in groupList" :key="group.id">{{
                  group.name
                }}
              </el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </div>
    </div>
    
    <div class="email-list">
      <el-table 
        ref="taskTableDataRef" 
        :data="data" 
        :show-header="false"
        :border="false" 
        @row-click="rowClick" 
        :row-style="rowStyle"
        class="email-table"
      >
        <el-table-column type="selection" width="30"/>
        <el-table-column prop="is_read" label="" width="50">
          <template #default="scope">
            <div>
                            <span v-if="!scope.row.is_read">
                                {{ lang.new }}
                            </span>
              <span style="font-weight: 900;color: #FF0000;" v-if="scope.row.dangerous">
                                <el-tooltip effect="dark" :content="lang.dangerous" placement="top-start">
                                    !
                                </el-tooltip>

                            </span>
              <span style="font-weight: 900;color: #FF0000;" v-if="scope.row.error !== ''">
                                <el-tooltip effect="dark" :content="scope.row.error" placement="top-start">
                                    !
                                </el-tooltip>

                            </span>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="title" :label="lang.sender" width="150">
          <template #default="scope">
            <el-tooltip class="box-item" effect="dark" :content="scope.row.sender.EmailAddress" placement="top">
              <el-tag size="small" type="info">
                {{ scope.row.sender.Name !== '' ? scope.row.sender.Name : scope.row.sender.EmailAddress }}
              </el-tag>
            </el-tooltip>
          </template>
        </el-table-column>

        <el-table-column prop="title" :label="lang.to" width="150">
          <template #default="scope">
            <el-tooltip v-for="toInfo in scope.row.to" :key="toInfo" class="box-item" effect="dark" :content="toInfo.EmailAddress"
                        placement="top">
              <el-tag size="small" type="info">{{ toInfo.Name !== '' ? toInfo.Name : toInfo.EmailAddress }}</el-tag>
            </el-tooltip>
          </template>
        </el-table-column>

        <el-table-column prop="desc" :label="lang.title">
          <template #default="scope">
            <div v-if="scope.row.is_read">{{ scope.row.title }}</div>
            <div v-else style="font-weight:bolder;">{{ scope.row.title }}</div>

            <div style="font-size: 12px;height: 24px;">{{ scope.row.desc }}</div>

          </template>
        </el-table-column>
        <el-table-column prop="datetime" :label="lang.date" width="180">
          <template #default="scope">
            <span v-if="scope.row.is_read">{{ scope.row.datetime }}</span>
            <span v-else style="font-weight:bolder;">{{ scope.row.datetime }}</span>
          </template>
        </el-table-column>
      </el-table>
    </div>
    
    <div class="email-pagination">
      <el-pagination 
        background 
        layout="prev, pager, next" 
        :page-count="totalPage" 
        @current-change="pageChange"
      />
    </div>
  </div>
</template>

<script setup>

import {EpArrowDownBold} from "vue-icons-plus/ep";
import {RouterLink, useRouter} from 'vue-router'
import {ref, watch} from 'vue'
import useGroupStore from '../stores/group'
import lang from '../i18n/i18n';
import {http} from "@/utils/axios";
import {ElMessage, ElMessageBox} from "element-plus";


const router = useRouter();
const groupStore = useGroupStore()
const groupList = ref([])
const taskTableDataRef = ref(null)
let tag = groupStore.tag;

if (tag === "") {
  tag = '{"type":0,"status":-1}'
}


watch(groupStore, async (newV) => {
  tag = newV.tag;
  if (tag === "") {
    tag = '{"type":0,"status":-1}'
  }
  data.value = []
  http.post("/api/email/list", {tag: tag, page_size: 10}).then(res => {
    data.value = res.data.list
    totalPage.value = res.data.total_page
  })
})


const data = ref([])
const totalPage = ref(0)

const updateList = function () {
  http.post("/api/email/list", {tag: tag, page_size: 10}).then(res => {
    data.value = res.data.list
    totalPage.value = res.data.total_page
  })
}

const updateGroupList = function () {
  http.post("/api/group/list").then(res => {
    groupList.value = res.data
  })
}

updateList()
updateGroupList()

const rowClick = function (row) {
  router.push("/detail/" + row.id)
}

const markRead = function () {
  let rows = taskTableDataRef.value?.getSelectionRows()
  let ids = []
  rows.forEach(element => {
    ids.push(element.id)
  });

  http.post("/api/email/read", {"ids": ids}).then(res => {
    if (res.errorNo === 0) {
      updateList()
    } else {
      ElMessage({
        type: 'error',
        message: res.errorMsg,
      })
    }
  })
}


const move = function (group_id) {
  let rows = taskTableDataRef.value?.getSelectionRows()
  let ids = []
  rows.forEach(element => {
    ids.push(element.id)
  });

  ElMessageBox.confirm(
      lang.move_email_confirm,
      'Warning',
      {
        confirmButtonText: 'OK',
        cancelButtonText: 'Cancel',
        type: 'warning',
      }
  )
      .then(() => {
        http.post("/api/email/move", {"group_id": group_id, "ids": ids}).then(res => {
          if (res.errorNo === 0) {
            updateList()
            ElMessage({
              type: 'success',
              message: 'Move completed',
            })
          } else {
            ElMessage({
              type: 'error',
              message: res.errorMsg,
            })
          }
        })


      })
}


const del = function () {
  let rows = taskTableDataRef.value?.getSelectionRows()
  let ids = []
  rows.forEach(element => {
    ids.push(element.id)
  });

  let groupTag = JSON.parse(tag)

  ElMessageBox.confirm(
    lang.del_email_confirm,
    '删除确认',
    {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning',
      center: true,
      customClass: 'delete-confirm-box',
      confirmButtonClass: 'confirm-delete-btn',
      cancelButtonClass: 'cancel-delete-btn'
    }
  ).then(() => {
    http.post("/api/email/del", {"ids": ids, "forcedDel": groupTag.status === 3}).then(res => {
      if (res.errorNo === 0) {
        updateList()
        ElMessage({
          type: 'success',
          message: '删除成功',
          customClass: 'success-message'
        })
      } else {
        ElMessage({
          type: 'error',
          message: res.errorMsg,
          customClass: 'error-message'
        })
      }
    })
  })
}
</script>

<style scoped>
.email-container {
  height: 100%;
  display: flex;
  flex-direction: column;
  background-color: #f5f7fa;
  padding: 20px;
}

.email-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
  padding: 10px 20px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.compose-btn {
  margin-right: auto;
}

.compose-link {
  display: inline-block;
  padding: 8px 16px;
  background-color: #409eff;
  color: white;
  border-radius: 4px;
  font-weight: bold;
  text-decoration: none;
  transition: all 0.3s;
}

.compose-link:hover {
  background-color: #66b1ff;
}

.email-title {
  font-size: 20px;
  font-weight: bold;
  color: #303133;
  margin: 0 20px;
}

.email-actions {
  display: flex;
  align-items: center;
}

.email-list {
  flex: 1;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.email-table {
  width: 100%;
}

.email-table :deep(.el-table__row) {
  transition: all 0.3s;
}

.email-table :deep(.el-table__row:hover) {
  background-color: #f5f7fa;
}

.email-pagination {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}

/* 删除弹窗样式 */
:global(.delete-confirm-box) {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: #2d3748;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  padding: 28px 32px;
  width: 420px !important;
  border: none !important;
  color: #e2e8f0;
}

:global(.delete-confirm-box .el-message-box__header) {
  position: relative;
  padding: 0 0 20px 0;
}

:global(.delete-confirm-box .el-message-box__headerbtn) {
  position: absolute;
  right: 0;
  top: 0;
}

:global(.delete-confirm-box .el-message-box__title) {
  font-size: 20px;
  font-weight: 600;
  color: #e2e8f0;
}

:global(.delete-confirm-box .el-message-box__content) {
  padding: 20px 0;
  font-size: 16px;
  color: #a0aec0;
}

:global(.delete-confirm-box .el-message-box__btns) {
  padding: 20px 0 0 0;
  text-align: right;
}

:global(.delete-confirm-box .el-message-box__btns button) {
  min-width: 100px;
  margin-left: 12px;
  padding: 10px 20px;
  font-size: 14px;
  border-radius: 6px;
  transition: all 0.2s;
}

:global(.confirm-delete-btn) {
  background-color: #ed64a6 !important;
  border-color: #ed64a6 !important;
  color: white !important;
}

:global(.confirm-delete-btn:hover) {
  background-color: #d53f8c !important;
  border-color: #d53f8c !important;
  transform: translateY(-1px);
}

:global(.cancel-delete-btn) {
  border: 1px solid #4a5568 !important;
  color: #e2e8f0 !important;
  background: transparent !important;
}

:global(.cancel-delete-btn:hover) {
  background-color: #4a5568 !important;
  transform: translateY(-1px);
}

:global(.delete-confirm-box .el-message-box__status) {
  font-size: 24px !important;
  color: #ed64a6 !important;
  margin-right: 10px !important;
}

/* 添加遮罩层样式 */
:global(.v-modal) {
  background-color: rgba(0, 0, 0, 0.5) !important;
  backdrop-filter: blur(2px);
}
</style>