<script setup>
import { onMounted, onUnmounted, reactive, ref, watch } from "vue";
import Cropper from "cropperjs";
import 'cropperjs/dist/cropper.css';
const emits = defineEmits(['changePage']);
const props = defineProps({
    fileToMainPage: {}
});

const Shuju = reactive({
    afterImg: '', // 裁剪后的图片
    image: null,  // 裁剪的图片
    myCropper: null // 进行裁剪
});
const coutOverList = ref([]);
const image = ref();
const mainBox = ref();
const isInputChange = ref(false);
const radio1 = ref('5');
const cropData = ref({
    x: 0,
    y: 0,
    width: 0,
    height: 0
});

watch(radio1, (newVal) => {
    switch (newVal) {
        case '1':
            Shuju.myCropper.setAspectRatio(1);
            break;
        case '2':
            Shuju.myCropper.setAspectRatio(3 / 2);
            break;
        case '3':
            Shuju.myCropper.setAspectRatio(16 / 9);
            break;
        case '4':
            Shuju.myCropper.setAspectRatio(16 / 10);
            break;
        case '5':
            Shuju.myCropper.setAspectRatio(NaN);
            break;
    }
});

const sureSava = () => {
    const cutOver = Shuju.myCropper.getCroppedCanvas({
        imageSmoothingQuality: 'high'
    }).toDataURL('image/png'); // 设置图片格式
    coutOverList.value.push(cutOver);
    console.log(typeof coutOverList.value);
    ElMessage({
        message: '剪切成功',
        type: 'success',
    });
    console.log(coutOverList.value.length);
};

function downloadImage(index) {
    // 创建下载链接并点击下载
    var now = new Date();
    const link = document.createElement('a');
    link.href = coutOverList.value[index];
    link.download = `易切${now.getHours()}点${now.getMinutes()}分${now.getSeconds()}秒生成.png`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}


// 传递img的DOM进来
const initPhoto = (photo) => {
    return new Promise((resolve) => {
        Shuju.image = photo;
        Shuju.myCropper = new Cropper(Shuju.image, {
            viewMode: 0,
            aspectRatio: null,
            dragMode: 'move',
            background: true,
            rotatable: true, // 是否允许旋转图片
            preview: '.before',
            autoCropArea: 0.8,
            zoomOnWheel: true,
            center: true,
            ready() {
                const cropBoxData = Shuju.myCropper.getCropBoxData();
                cropData.value = {
                    x: Math.round(cropBoxData.left),
                    y: Math.round(cropBoxData.top),
                    width: Math.round(cropBoxData.width),
                    height: Math.round(cropBoxData.height),
                };
                resolve();
            },
            crop(event) {// 裁剪框改变时触发
                if (!isInputChange.value) {
                    cropData.value = {
                        x: Math.round(event.detail.x),
                        y: Math.round(event.detail.y),
                        width: Math.round(event.detail.width),
                        height: Math.round(event.detail.height),
                    };
                }
            }
        });
    });
};

const imgWidth = ref(0);
const imgHeight = ref(0);

function handleKeyPress(event) {
    const key = event.key;
    const canvasData = Shuju.myCropper.getCanvasData();
    const imageData = Shuju.myCropper.getImageData();
    const scaleX = imageData.naturalWidth / imageData.width;
    const scaleTop = imageData.naturalHeight / imageData.height;
    switch (key) {
        case 'w' || 'W'://上键
            ElMessage({
                message: '已将选择框靠上',
                type: 'success',
            });
            const newTop = 0 / scaleTop + canvasData.top;
            Shuju.myCropper.setCropBoxData({
                top: newTop,
            });
            break;
        case 'a' || 'A'://左键
            ElMessage({
                message: '已将选择框靠左',
                type: 'success',
            });
            const newLeft = 0 / scaleX + canvasData.left;
            Shuju.myCropper.setCropBoxData({
                left: newLeft,
            });
            break;
        case 's' || 'S'://下键
            ElMessage({
                message: '已将选择框靠下',
                type: 'success',
            });
            const newbottom = (0 / scaleTop + canvasData.top) + imageData.height - Shuju.myCropper.getCropBoxData().height;
            Shuju.myCropper.setCropBoxData({
                top: newbottom,
            });
            break;
        case 'd' || 'D'://右键
            ElMessage({
                message: '已将选择框靠右',
                type: 'success',
            });
            const newright = (0 / scaleX + canvasData.left) + imageData.width - Shuju.myCropper.getCropBoxData().width;
            Shuju.myCropper.setCropBoxData({
                left: newright,
            });
            break;
        case 'v' || 'V'://复位
            ElMessage({
                message: '已复位',
                type: 'success',
            });
            Shuju.myCropper.reset();
            break;
    }
}

onMounted(async () => {
    image.value.src = props.fileToMainPage.preview;
    await initPhoto(image.value);
    imgWidth.value = Shuju.myCropper.getImageData().naturalWidth;
    imgHeight.value = Shuju.myCropper.getImageData().naturalHeight;
    document.addEventListener('keydown', handleKeyPress);
});
onUnmounted(() => {
    document.removeEventListener('keydown', handleKeyPress);
})
const xMove = () => {
    isInputChange.value = true;
    const canvasData = Shuju.myCropper.getCanvasData();
    const imageData = Shuju.myCropper.getImageData();
    const scaleX = imageData.naturalWidth / imageData.width;

    console.log('Canvas Left:', canvasData.left);
    console.log('Crop Data X:', Number(cropData.value.x));

    // 计算新的left位置
    const newLeft = Number(cropData.value.x) / scaleX + canvasData.left;

    console.log('New Left:', newLeft);

    Shuju.myCropper.setCropBoxData({
        left: newLeft,
    });
    isInputChange.value = false;
}
const yMove = () => {
    isInputChange.value = true;

    const canvasData = Shuju.myCropper.getCanvasData();
    const imageData = Shuju.myCropper.getImageData();
    const scaleY = imageData.naturalHeight / imageData.height;

    console.log('Canvas Top:', canvasData.top);
    console.log('Crop Data Y:', Number(cropData.value.y));

    // 计算新的top位置
    const newTop = Number(cropData.value.y) / scaleY + canvasData.top;

    console.log('New Top:', newTop);

    Shuju.myCropper.setCropBoxData({
        top: newTop,
    });

    isInputChange.value = false;
}

const wMove = () => {
    isInputChange.value = true;
    const scaleX = Shuju.myCropper.getImageData().naturalWidth / Shuju.myCropper.getImageData().width;
    Shuju.myCropper.setCropBoxData({
        width: Number(cropData.value.width) / scaleX,
    });
    if (radio1.value === '1') {
        cropData.value.height = cropData.value.width;
    }
    if (radio1.value === '2') {
        cropData.value.height = Math.round(cropData.value.width * 2 / 3);
    }
    if (radio1.value === '3') {
        cropData.value.height = Math.round(cropData.value.width * 9 / 16);
    }
    if (radio1.value === '4') {
        cropData.value.height = Math.round(cropData.value.width * 10 / 16);
    }
    isInputChange.value = false;
}

const hMove = () => {
    isInputChange.value = true;
    const scaleY = Shuju.myCropper.getImageData().naturalHeight / Shuju.myCropper.getImageData().height;
    Shuju.myCropper.setCropBoxData({
        height: Number(cropData.value.height) / scaleY,
    });
    if (radio1.value === '1') {
        cropData.value.width = cropData.value.height;
    }
    if (radio1.value === '2') {
        cropData.value.width = Math.round(cropData.value.height * 3 / 2);
    }
    if (radio1.value === '3') {
        cropData.value.width = Math.round(cropData.value.height * 16 / 9);
    }
    if (radio1.value === '4') {
        cropData.value.width = Math.round(cropData.value.height * 16 / 10);
    }
    isInputChange.value = false;
}

const fuwei = () => {
    Shuju.myCropper.reset();
}

const rmOverImg = (index) => {
    console.log(index);
    coutOverList.value.splice(index, 1);
}
const changeImgToP0 = () => {
    emits('changePage');
}
const xaunzhuanValue = ref(0);
watch(xaunzhuanValue, (newVal, oldVal) => {
    Shuju.myCropper.rotate(newVal - oldVal);
})
const drawer = ref(false);
const showLeft = () => {
    drawer.value = true;
}
const direction = ref('ltr');
</script>

<template>
    <div class="mainBox" ref="mainBox" @dragover.prevent="noPreventDefault" @drop.prevent="uploadFile">

        <el-drawer v-model="drawer" title="高级设置" :direction="direction" :size=380>
            <div>
                <h3 style="margin-bottom: 10px;">快捷键</h3>
                <span>选择框靠上: <span style="margin-left:98px;">W</span></span><br>
                <span>选择框靠左: <span style="margin-left:100px;">A</span></span><br>
                <span>选择框靠下: <span style="margin-left:100px;">S</span></span><br>
                <span>选择框靠右: <span style="margin-left:100px;">D</span></span><br>
                <span>快速复位: <span style="margin-left:115px;">V</span></span><br>
                <div style="position: fixed;bottom: 10px; left: 6px; ">
                    作者：liyinghao<br>
                    作者gitee链接：<a style="color: black;" target="_blank"
                        href="https://gitee.com/hyl19">https://gitee.com/hyl19</a><br>
                    主站友情链接：<a style="color: black;" target="_blank"
                        href="https://www.liyinghao.cc">https://www.liyinghao.cc</a>
                </div>
            </div>
        </el-drawer>

        <div class="PreviewBox">
            <div style="position: absolute; margin: 5px; width: 30px; height: 30px; cursor: pointer;"
                @click="showLeft()">
                <svg t="1722070347596" class="icon" viewBox="0 0 1024 1024" version="1.1"
                    xmlns="http://www.w3.org/2000/svg" p-id="941" width="30" height="30">
                    <path
                        d="M684.544 174.08a81.92 81.92 0 0 1 74.56 48l116.352 256a81.92 81.92 0 0 1 0 67.84l-116.352 256a81.92 81.92 0 0 1-74.56 48H339.456a81.92 81.92 0 0 1-74.56-48l-116.352-256a81.92 81.92 0 0 1 0-67.84l116.352-256a81.92 81.92 0 0 1 74.56-48h345.088z m0 61.44H339.456a20.48 20.48 0 0 0-18.624 11.968l-116.352 256a20.48 20.48 0 0 0 0 16.96l116.352 256a20.48 20.48 0 0 0 18.624 12.032h345.088a20.48 20.48 0 0 0 18.624-12.032l116.416-256a20.48 20.48 0 0 0 0-16.96l-116.416-256a20.48 20.48 0 0 0-18.56-11.968h-0.064zM512 358.4a153.6 153.6 0 1 1 0 307.2 153.6 153.6 0 0 1 0-307.2z m0 61.44a92.16 92.16 0 1 0 0 184.32 92.16 92.16 0 0 0 0-184.32z"
                        fill="#707070" p-id="942"></path>
                </svg>
            </div>
            <!-- 在preview中指定了预览容器 -->
            <div class="before" style="background-color: transparent;"></div>
        </div>

        <div class="toolBox">
            <el-button class="toolBoxMainBoxButton" type="primary" @click="sureSava">裁剪</el-button>
            <div class="toolBoxMainBox">
                <div class="toolBoxMainBoxP0">
                    <h3>原图参数</h3>
                    <div style="display: flex;">
                        <div style="text-align:left;margin-right: 20px;">
                            <p>名称：</p>
                            <p>大小(MB):</p>
                            <p>原图宽度(px)：</p>
                            <p>原图高度(px)：</p>
                        </div>
                        <div style="text-align: right; width: 222px;">
                            <p style="overflow: hidden; text-overflow: ellipsis;">{{ props.fileToMainPage.name }}</p>
                            <p> {{ (props.fileToMainPage.size / 1024 / 1024).toFixed(3) }}</p>
                            <p>{{ imgWidth }}</p>
                            <p>{{ imgHeight }}</p>
                        </div>
                    </div>
                </div>
                <div class="toolBoxMainBoxP1">
                    <h3>裁切比例</h3>
                    <el-radio-group class="radioBox" v-model="radio1" size="large">
                        <el-radio-button label="1/1" value="1" />
                        <el-radio-button label="3/2" value="2" />
                        <el-radio-button label="16/9" value="3" />
                        <el-radio-button label="16/10" value="4" />
                        <el-radio-button label="自由" value="5" />
                    </el-radio-group>
                </div>
                <div class="toolBoxMainBoxP2">
                    <h3>裁切参数</h3>
                    <div style="display: flex;">
                        <div style="text-align: left; white-space: nowrap; line-height: 32px;width: 250px;">
                            <p>x坐标(px):</p>
                            <p>y坐标(px):</p>
                            <p>宽度(px): </p>
                            <p>高度(px): </p>
                        </div>
                        <div style="text-align: right;">
                            <el-input class="right0" v-on:input="xMove()" v-model="cropData.x" style="width: 240px"
                                :placeholder="`${cropData.x}`" />
                            <el-input class="right0" v-on:input="yMove()" v-model="cropData.y" style="width: 240px"
                                :placeholder="`${cropData.y}`" />
                            <el-input class="right0" v-on:input="wMove()" v-model="cropData.width" style="width: 240px"
                                :placeholder="`${cropData.width}`" />
                            <el-input class="right0" v-on:input="hMove()" v-model="cropData.height" style="width: 240px"
                                :placeholder="`${cropData.height}`" />
                        </div>
                    </div>
                    <div style="margin-top: 5px;">
                        <p>旋转(degrees):</p>
                        <el-slider style="width: 90%; margin-top: 10px; margin-left: 10px; overflow: visible;"
                            v-model="xaunzhuanValue" :min="0" :max="359" />
                    </div>
                </div>
            </div>
            <el-button class="toolBoxMainBoxButton" style="bottom: 0px;" type="warning"
                @click="changeImgToP0()">更改图片</el-button>
        </div>

        <div class="topBox">
            <div class="topBoxItem" v-for="(item, index) in coutOverList" :key="index">
                <el-tooltip class="box-item" effect="dark" content="点击可以预览" placement="right" :hide-after=100
                    :show-after=100>
                    <el-image class="topBoxItemImg" :src="item" :zoom-rate="1.2" :max-scale="7" :min-scale="0.2"
                        :preview-src-list="coutOverList" :initial-index=index fit="cover" />
                </el-tooltip>
                <div class="topBoxItemBtn">
                    <el-button style="width: 80px;margin-left: 10px; margin-right: 10px;" @click="downloadImage(index)"
                        type="primary" plain>保存</el-button>
                    <el-button style="width: 80px;margin-left: 9px;" @click="rmOverImg(index)" type="primary"
                        plain>移除</el-button>
                </div>
            </div>
        </div>

        <div class="photoBox">
            <div class="fuweiBtn" @click="fuwei()">
                复位
            </div>
            <div class="box_1">
                <img class="Mainimage" ref="image" draggable="false">
            </div>
        </div>
    </div>
</template>


<style scoped lang="less">
/* 自定义滚动条的整体样式 */
::-webkit-scrollbar {
    width: 6px;
    /* 滚动条的宽度（竖向滚动条）或高度（横向滚动条） */
    height: 6px;
    /* 横向滚动条的高度 */
}

/* 自定义滚动条轨道 */
::-webkit-scrollbar-track {
    background: #f1f1f1;
    /* 轨道背景色 */
    border-radius: 10px;
    /* 轨道圆角 */
}

/* 自定义滚动条滑块 */
::-webkit-scrollbar-thumb {
    background: #79bbff;
    /* 滑块背景色 */
    border-radius: 10px;
    /* 滑块圆角 */
}

/* 当滚动条滑块被悬停时的样式 */
::-webkit-scrollbar-thumb:hover {
    background: #337ecc;
    /* 悬停时滑块的背景色 */
}

* {
    overflow: hidden;
}

html {
    min-width: 600px;
}

.mainBox {
    width: 100vw;
    height: 100vh;
}

.PreviewBox {
    position: relative;
    top: 0px;
    left: 0px;
    width: 380px;
    height: 30vh;
    overflow: hidden;

    .before {
        position: relative;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        width: 380px;
        height: 30vh;
        border: 4px solid #b1b3b8;
        overflow: hidden;
    }
}

.toolBox {
    position: absolute;
    top: 30vh;
    left: 0px;
    width: 380px;
    height: 70vh;
    padding: 6px;

    .toolBoxMainBox {
        position: relative;
        background-color: #d6dce6;
        padding: 6px;
        height: 80%;
        border-radius: 6px;
        overflow: auto;

        h3 {
            margin-top: 6px;
            margin-bottom: 6px;
            text-align: center;
        }

        .radioBox {
            position: relative;
            left: 50%;
            transform: translateX(-50%);
        }

        .toolBoxMainBoxP0 {
            white-space: nowrap;
            overflow: hidden;
        }
    }

    .toolBoxMainBoxButton {
        width: 100%;
        height: 9%;
        margin-top: 5px;
        margin-bottom: 5px;
    }
}

.topBox {
    position: absolute;
    top: 0px;
    right: 0px;
    width: calc(100vw - 380px);
    height: 30vh;
    background-color: #f5f5f7;
    display: flex;
    overflow-x: scroll;

    .topBoxItem {
        position: relative;
        background-color: #f3d19e;
        height: 28vh;
        width: 200px;
        margin: 1vh;
        min-width: 200px;

        .topBoxItemImg {
            position: absolute;
            top: 20px;
            left: 10px;
            background-color: white;
            width: 180px;
            height: 20vh;
            cursor: pointer;
            object-fit: cover;
        }

        .topBoxItemBtn {
            position: absolute;
            width: 100%;
            bottom: 0px;
            display: flex;
            margin-bottom: 4px;
        }
    }
}



.photoBox {
    position: absolute;
    width: calc(100vw - 380px);
    height: 70vh;
    top: 30vh;
    right: 0px;
    overflow: hidden;

    .fuweiBtn {
        position: absolute;
        background-color: #95d475;
        color: white;
        z-index: 99;
        width: 36px;
        height: 36px;
        top: 10px;
        left: 10px;
        border-radius: 50%;
        text-align: center;
        line-height: 36px;
        font-size: 15px;
        transition: all 0.3s;
        cursor: pointer;
    }

    div {
        width: calc(100vw - 380px);
        height: 70vh;

        img {
            display: block;
            width: calc(100vw - 380px);
            height: 70vh;
        }
    }
}
</style>