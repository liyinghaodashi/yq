<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { ElMessage } from 'element-plus'
import mainPage from './views/MainPage.vue'
const showMainPage = ref(false)
const fileToMainPage = ref({})
const dropZone = ref(null);

onMounted(() => {
    document.addEventListener('dragover', handleDocumentDragOver);
    document.addEventListener('dragleave', handleDocumentDragLeave);
    document.addEventListener('drop', handleDocumentDrop);
});

onBeforeUnmount(() => {
    document.removeEventListener('dragover', handleDocumentDragOver);
    document.removeEventListener('dragleave', handleDocumentDragLeave);
    document.removeEventListener('drop', handleDocumentDrop);
});


function loadFile(file) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (e) => {
            const newFile = {
                name: file.name,
                size: file.size,
                preview: e.target.result,
            };
            fileToMainPage.value.preview = newFile.preview;
            fileToMainPage.value.size = newFile.size;
            fileToMainPage.value.name = newFile.name;
            showMainPage.value = !showMainPage.value;
        };
        reader.readAsDataURL(file);
    })
}

const handleFiles = (droppedFiles) => {
    for (const file of droppedFiles) {
        if (file.type.startsWith('image/')) {
            loadFile(file)
        }
    }
};

const handleDocumentDrop = (event) => {
    event.preventDefault();
    dropZone.value.classList.remove('dragover');
    const droppedFiles = Array.from(event.dataTransfer.files);
    console.log(droppedFiles.length);
    if (droppedFiles.length > 1) {
        ElMessage({
            message: '目前只支持处理一张图片',
            type: 'warning',
        })
        return;
    }
    handleFiles(droppedFiles);
};

const handleDocumentDragOver = (event) => {
    event.preventDefault();
    dropZone.value.classList.add('dragover');
};

const handleDocumentDragLeave = () => {
    dropZone.value.classList.remove('dragover');
};
const addimg = () => {
    //生成一个input
    const input = document.createElement('input');
    input.type = 'file';
    input.accept = 'image/*';
    input.onchange = (e) => {
        const file = e.target.files[0];
        loadFile(file);
    };
    input.click();
    input.remove();
}
const changePage = () => {
    showMainPage.value = !showMainPage.value;
}
</script>

<template>
    <div class="loderPage" v-if="!showMainPage" @click="addimg()">

        <div ref="dropZone" class="drop-zone">
            拖拽或点击开始使用
        </div>
    </div>

    <div class="mainPage" v-if="showMainPage">
        <mainPage :fileToMainPage="fileToMainPage" @changePage="changePage()" />
    </div>
</template>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    /* ::-webkit-scrollbar {display:none} */
    user-select: none;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
}

.loderPage {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}

.drop-zone {
    border: 2px dashed #ccc;
    padding: 20px;
    text-align: center;
    transition: background-color 0.3s;
    margin: 20px;
}

.drop-zone.dragover {
    background-color: #f0f0f0;
}

.avatar-uploader .avatar {
    width: 178px;
    height: 178px;
    display: block;
}

.avatar-uploader .el-upload {
    border: 1px dashed var(--el-border-color);
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
    border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    text-align: center;
}
</style>
