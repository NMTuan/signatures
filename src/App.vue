<template>
    <div>
        <div v-if="tips">{{ tips }}</div>
        <div>
            1. 读取本地字体:
            <button @click="refreshLoadFontList">刷新字体</button>
        </div>
        <div>
            2. 选择本地字体:
            <select v-model="selectedFont">
                <option
                    v-for="font in localFontList_cn"
                    :key="font.fullName"
                    :value="font.fullName"
                >
                    {{ font.fullName }}
                </option></select
            >{{ selectedFont }}
        </div>
        <div>
            3. 输入文字内容:
            <input type="text" v-model="text" />
        </div>
        <div>
            4. 处理成 svg 图:
            <button @click="handleFont">处理</button>
        </div>
        <div>
            5. 动起来动起来:
            <button @click="handleAnimation">动感光波！</button>
        </div>

        <hr />
        <svg width="1000" height="200" viewBox="0 0 1000 200">
            <path
                ref="pathRef"
                :d="paths"
                fill="none"
                stroke="red"
                strokeWidth="2"
                style="animation: 'draw 2s linear forwards';"
                :style="{
                  animation: 'draw 2s linear forwards'
                }"
            />
        </svg>
    </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import opentype from "opentype.js";

// 提示信息
const tips = ref("");
// 所有本地字体
const localFontList = ref([]);
// 选中的字体
const selectedFont = ref("");
// 输入的文字
const text = ref("前端小海");
const paths = ref();
const pathRef = ref();
// 中文字体
const localFontList_cn = computed(() => {
    const reg = /[\u4e00-\u9fa5]/;
    return localFontList.value.filter((item) => reg.test(item.fullName));
});
// 读取本地字体
const loadLocalFontList = async () => {
    const fonts = await queryLocalFonts();
    if (!fonts.length) {
        tips.value = "没有读到本地字体";
        return;
    }
    localFontList.value = fonts;
};

// 刷新字体
const refreshLoadFontList = () => {
    loadLocalFontList();
};

// 处理输入的文字
const handleFont = async () => {
    paths.value = null;
    // 找到字体
    const currentFont = localFontList.value.find(
        (item) => item.fullName === selectedFont.value
    );
    // 转换数据格式
    const blob = await currentFont.blob();
    const arrayBuffer = await blob.arrayBuffer();
    // opentype 读取 字体
    const fontData = await opentype.parse(arrayBuffer);
    console.log("fontData", fontData);
    // 循环处理 text
    // const y =
    //     ((fontData.ascender / 2 + fontData.descender / 2) /
    //         fontData.unitsPerEm) *
    //     200;
    const fontPath = fontData.getPath(text.value, 0, 100, 100);
    paths.value = fontPath.toPathData();
    // text.value.split("").forEach((item) => {
    //     const fontPath = fontData.getPath(item, -100, y, 200);
    //     console.log("path", fontPath);
    //     paths.value.push({
    //         label: item,
    //         path: fontPath.toPathData(),
    //     });
    // });
};

// 动起来
const handleAnimation = () => {
    const len = pathRef.value.getTotalLength();
    console.log("len", len);
    pathRef.value.style.strokeDasharray = len;
    pathRef.value.style.strokeDashoffset = len;
};
</script>

<style scoped>
@keyframes draw {
    to {
        stroke-dashoffset: 0;
    }
}
</style>
