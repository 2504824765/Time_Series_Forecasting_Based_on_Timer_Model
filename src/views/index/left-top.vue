<template>
  <div class="content">
    <br><br>
    <h3>请上传模型运行结果文件夹</h3>
    <br><br><br>
    <!-- <input class="content_input" type="file" webkitdirectory @change="handleFolderUpload" /> -->
  </div>
  <div class="custom-file-upload">
    <!-- 隐藏原生 input -->
    <input 
      type="file" 
      id="fileInput" 
      webkitdirectory
      @change="handleFolderUpload"
      style="display: none"
    >
    <!-- 使用 label 作为自定义按钮 -->
    <label for="fileInput" class="custom-button">
      选择文件
    </label>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';

export default {
  name: 'LeftTop',

  methods: {
    handleFolderUpload(event) {
      const files = Array.from(event.target.files).filter(file =>
        file.name.endsWith('.xlsx')
      );

      const fileMap = {};
      const readPromises = files.map(file => {
        return new Promise(resolve => {
          const reader = new FileReader();
          reader.onload = e => {
            const data = new Uint8Array(e.target.result);
            const workbook = XLSX.read(data, { type: 'array' });
            const sheetName = workbook.SheetNames[0];
            const worksheet = workbook.Sheets[sheetName];
            const jsonData = XLSX.utils.sheet_to_json(worksheet);
            fileMap[file.name] = jsonData;
            resolve();
          };
          reader.readAsArrayBuffer(file);
        });
      });

      Promise.all(readPromises).then(() => {
        this.$emit('folder-data-loaded', {
          fileNames: Object.keys(fileMap),
          fileDataMap: fileMap
        });
      });
    }
  }
};
</script>

<style>
.custom-file-upload {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 10px;
  align-items: center;
}

.custom-button {
  padding: 15px 30px; /* 调整按钮大小 */
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 30px;
  transition: background-color 0.3s;
}

.custom-button:hover {
  background-color: #45a049;
}

.file-name {
  margin-top: 5px;
  font-size: 14px;
  color: #ffffff;
}

  .content {
    display: flex;
    justify-content: center;
    flex-direction: column;
    align-items: center;

    h3 {
      text-align: center;
      font-size: 30px;
    }

    .content_input {
      /* margin-left: 100px; */
      font-size: 20px;
    }
  }
</style>