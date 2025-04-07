<template>
    <div class="file-list">
      <!-- <h3>上传的文件列表</h3> -->
       <div v-if="!fileNames || fileNames.length === 0" class="empty-message">
        <br><br><br><br>
        <p class="empty_content">请上传模型导出结果</p>
       </div>
       <div v-else>
      <table>
        <thead>
          <tr>
            <th>序号</th>
            <th>可选特征变量</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(name, index) in fileNames" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ name }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    </div>
  </template>
  
<script setup lang="ts">
  defineProps<{
    fileNames: string[];
  }>();
</script>
  
<style scoped>
.empty_content {
    text-align: center;
    font-size: 30px;
}

  .file-list {
    padding: 20px;
    font-size: 16px;
    height: 90%;
    overflow-y: auto; /* 添加滚动条 */
  }
  
  table {
    width: 100%;
    border-collapse: collapse;
  }
  
  th,
  td {
    border: 1px solid #ccc;
    padding: 8px 12px;
    text-align: left;
  }
  
  th {
    background-color: #f2f2f2;
    color: black;
  }
</style>
  