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
        <!-- <div>
            5. 动起来动起来:
            <button @click="handleAnimation">动感光波！</button>
        </div> -->

        <hr />
        <!-- <svg width="1000" height="200" viewBox="0 0 1000 200">
            <path
                v-for="(stroke, index) in strokes"
                ref="pathRef"
                :d="stroke.path"
                fill="white"
                stroke="red"
                strokeWidth="2"
                :chip-path="`url(#clip-path-${index})`)"
            />
        </svg> -->
        <draw-svg
            v-if="fontData"
            :font="fontData"
            :text="text"
            :fontSize="48"
            :height="200"
            stroke="white"
            :strokeWidth="1"
            :animationDuration="0.5"
        >
        </draw-svg>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, nextTick } from "vue";
import opentype from "opentype.js";
import drawSvg from "./components/drawSvg.vue";

// 提示信息
const tips = ref("");
// 所有本地字体
const localFontList = ref([]);
// 选中的字体
const selectedFont = ref("");
// 输入的文字
const text = ref("前端小海");
// 笔画
const paths = ref();
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

const fontData = ref();

// 处理输入的文字
const handleFont = async () => {
    fontData.value = null
    // paths.value = null;
    // 找到字体
    const currentFont = localFontList.value.find(
        (item) => item.fullName === selectedFont.value
    );
    // 转换数据格式
    const blob = await currentFont.blob();
    const arrayBuffer = await blob.arrayBuffer();
    // opentype 读取 字体
    fontData.value = await opentype.parse(arrayBuffer);
};
</script>

<style scoped>
@keyframes draw {
    to {
        stroke-dashoffset: 0;
    }
}
</style>
