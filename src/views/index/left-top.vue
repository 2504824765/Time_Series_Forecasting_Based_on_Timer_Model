<script setup lang="ts">
import { ref } from 'vue';
import * as XLSX from 'xlsx';

const emit = defineEmits(['file-loaded']);

const fileData = ref<any[]>([]);
const headers = ref<string[]>([]);
const errorMessage = ref<string | null>(null);

const handleFileChange = (e: Event) => {
  const input = e.target as HTMLInputElement;
  const files = input.files;
  
  // 重置状态
  fileData.value = [];
  headers.value = [];
  errorMessage.value = null;

  if (!files || files.length === 0) return;

  const file = files[0];
  const reader = new FileReader();
  
  reader.onload = (e) => {
    try {
      const data = new Uint8Array(e.target?.result as ArrayBuffer);
      const workbook = XLSX.read(data, { type: 'array' });
      
      const firstSheetName = workbook.SheetNames[0];
      const worksheet = workbook.Sheets[firstSheetName];
      const jsonData = XLSX.utils.sheet_to_json(worksheet);
      
      if (jsonData.length > 0) {
        headers.value = Object.keys(jsonData[0] as Record<string, unknown>);
        fileData.value = jsonData;
        emit('file-loaded', { headers: headers.value, data: jsonData });
      }
    } catch (error) {
      errorMessage.value = '解析Excel文件时出错';
      console.error(error);
    }
  };
  
  reader.readAsArrayBuffer(file);
};
</script>

<template>
  <div class="left-top-container">
    <!-- <h3>Excel 数据导入</h3> -->
    <input 
      type="file" 
      accept=".xlsx, .xls" 
      @change="handleFileChange"
      class="excel-input"
    />
    
    <div v-if="errorMessage" class="error-message">
      {{ errorMessage }}
    </div>
    
    <div v-if="fileData.length > 0" class="data-preview">
      <h4>数据预览 (前20行):</h4>
      <table>
        <thead>
          <tr>
            <th v-for="header in headers" :key="header">{{ header }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, index) in fileData.slice(0, 20)" :key="index">
            <td v-for="header in headers" :key="header">
              {{ row[header] || '-' }}
            </td>
          </tr>
        </tbody>
      </table>
      <p>共 {{ fileData.length }} 行数据</p>
    </div>
  </div>
</template>

<style scoped lang="scss">
.left-top-container {
  padding: 16px;
  height: 100%;
  overflow: hidden; /* 防止内容溢出 */
  display: flex;
  flex-direction: column;
  
  h3 {
    margin-bottom: 12px;
    font-size: 16px;
  }
  
  .excel-input {
    margin-bottom: 16px;
    width: 100%;
  }
  
  .error-message {
    color: red;
    margin-bottom: 16px;
  }
  
  .data-preview {
    /* 表格容器关键样式 */
    flex: 1;
    overflow: auto; /* 允许内容滚动 */
    margin-top: 16px;
    max-height: calc(100% - 60px); /* 减去标题和上传按钮的高度 */
    
    table {
      width: 100%;
      border-collapse: collapse;
      table-layout: fixed; /* 新增：固定表格布局 */

      th, td {
        border: 1px solid #ddd;
        padding: 6px;
        font-size: 12px;
        white-space: nowrap; /* 不换行 */
        overflow: hidden; /* 隐藏溢出内容 */
        text-overflow: ellipsis; /* 显示省略号 */
      }
      
      th {
        background-color:#414141;
        top: 0; /* 固定表头 */
      }
    }
    
    p {
      font-size: 12px;
      color: #666;
      text-align: right;
    }
  }
}
</style>