<template>
  <div id="main">
    <div class="email-header">
      <div id="title">{{ detailData.subject }}</div>
      
      <div class="email-meta">
        <div class="meta-row">
          <div class="meta-label">{{ lang.sender }}:</div>
          <div class="meta-content">
            <el-tooltip class="box-item" effect="dark" :content="detailData.from_address" placement="top">
              <el-tag size="small" type="info">
                {{ detailData.from_name !== '' ? detailData.from_name : detailData.from_address }}
              </el-tag>
            </el-tooltip>
          </div>
        </div>
        
        <div class="meta-row">
          <div class="meta-label">{{ lang.to }}:</div>
          <div class="meta-content">
            <el-tooltip v-for="to in tos" :key.prop="to" class="box-item" effect="dark" :content="to.EmailAddress" placement="top">
              <el-tag size="small" type="info">{{ to.Name !== '' ? to.Name : to.EmailAddress }}</el-tag>
            </el-tooltip>
          </div>
        </div>
        
        <div class="meta-row" v-if="showCC">
          <div class="meta-label">{{ lang.cc }}:</div>
          <div class="meta-content">
            <el-tooltip v-for="item in ccs" :key="item" class="box-item" effect="dark" :content="item.EmailAddress" placement="top">
              <el-tag size="small" type="info">{{ item.Name !== '' ? item.Name : item.EmailAddress }}</el-tag>
            </el-tooltip>
          </div>
        </div>
        
        <div class="meta-row">
          <div class="meta-label">{{ lang.date }}:</div>
          <div class="meta-content date-content">{{ detailData.send_date }}</div>
        </div>
      </div>
    </div>
    
    <el-divider class="content-divider"/>
    
    <div class="email-body">
      <div class="content" id="text" v-if="detailData.html === ''">
        {{ detailData.text }}
      </div>

      <div class="content" id="html" v-else v-html="detailData.html">
      </div>
    </div>

    <div v-if="detailData.attachments.length > 0" class="email-attachments">
      <el-divider/>
      <div class="attachments-header">{{ lang.attachment }}</div>
      <div class="attachments-list">
        <a class="att" v-for="item in detailData.attachments" :key="item"
           :href="'/attachments/download/' + detailData.id + '/' + item.Index">
          <el-icon>
            <Document/>
          </el-icon>
          <span class="attachment-name">{{ item.Filename }}</span>
        </a>
      </div>
    </div>
  </div>
</template>

<script setup>

import {ref} from 'vue'
import {useRoute} from 'vue-router'
import {Document} from '@element-plus/icons-vue';
import lang from '../i18n/i18n';
import {http} from "@/utils/axios";

const route = useRoute()
const detailData = ref({
  attachments: []
})

const tos = ref()
const ccs = ref()
const showCC = ref(false)

http.post("/api/email/detail", {id: parseInt(route.params.id)}).then(res => {
  detailData.value = res.data
  if (res.data.to !== "" && res.data.to != null) {
    tos.value = JSON.parse(res.data.to)
  }
  if (res.data.cc !== "" && res.data.cc != null) {
    ccs.value = JSON.parse(res.data.cc)

  }

  if (ccs.value != null) {
    showCC.value = ccs.value.length > 0
  } else {
    showCC.value = false
  }
})
</script>

<style scoped>
#main {
  max-width: 850px;
  margin: 0 auto;
  padding: 24px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
}

.email-header {
  padding-bottom: 16px;
}

#title {
  font-size: 22px;
  font-weight: 600;
  margin-bottom: 20px;
  color: #202124;
  text-align: left;
  padding-bottom: 8px;
  border-bottom: 1px solid #f1f3f4;
}

.email-meta {
  text-align: left;
  font-size: 14px;
}

.meta-row {
  display: flex;
  margin-bottom: 8px;
  align-items: flex-start;
}

.meta-label {
  width: 70px;
  color: #5f6368;
  font-weight: 500;
  flex-shrink: 0;
}

.meta-content {
  flex: 1;
}

.date-content {
  color: #5f6368;
  font-size: 13px;
}

.content-divider {
  margin: 0 0 20px 0;
}

.email-body {
  background-color: #fafafa;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
  text-align: left;
  min-height: 200px;
}

.content {
  line-height: 1.7;
  font-size: 14px;
  color: #202124;
}

#html {
  min-height: 200px;
}

.email-attachments {
  text-align: left;
  padding-top: 8px;
}

.attachments-header {
  font-weight: 500;
  color: #5f6368;
  margin-bottom: 12px;
  font-size: 14px;
}

.attachments-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

a, a:link, a:visited, a:hover, a:active {
  text-decoration: none;
  color: #1a73e8;
}

.att {
  display: inline-flex;
  align-items: center;
  padding: 8px 12px;
  background: #f1f3f4;
  border-radius: 6px;
  transition: background-color 0.2s;
  border: 1px solid #e0e0e0;
}

.att:hover {
  background: #e8f0fe;
}

.attachment-name {
  margin-left: 6px;
  max-width: 200px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.el-tag {
  margin-right: 8px;
  margin-bottom: 4px;
  font-size: 12px;
  padding: 0 8px;
  height: 24px;
  line-height: 22px;
}

.el-tooltip {
  display: inline-block;
}

.el-divider {
  margin: 16px 0;
}
</style>