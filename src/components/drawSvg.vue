<template>
    <svg :width="width" :height="height" :viewBox="viewBox">
      <defs>
        <clipPath v-for="(stroke, index) in strokes" :key="`clip-${index}`" :id="`clip-path-${index}`">
          <rect :width="stroke.width" :height="stroke.height" :x="stroke.x" :y="stroke.y">
            <animate 
              attributeName="width" 
              :from="0" 
              :to="stroke.width" 
              :begin="`${index * animationDuration}s`" 
              :dur="`${animationDuration}s`" 
              fill="freeze" 
            />
          </rect>
        </clipPath>
      </defs>
      
      <path 
        v-for="(stroke, index) in strokes" 
        :key="`stroke-${index}`"
        :d="stroke.path" 
        fill="none"
        :stroke="stroke.color"
        :stroke-width="strokeWidth"
        :clip-path="`url(#clip-path-${index})`"
      />
    </svg>
  </template>
  
  <script>
  import { computed, ref, onMounted } from 'vue';
  
  export default {
    name: 'HandwritingSimulationSvg',
    props: {
      font: {
        type: Object,
        required: true
      },
      text: {
        type: String,
        required: true
      },
      fontSize: {
        type: Number,
        default: 72
      },
      width: {
        type: Number,
        default: 300
      },
      height: {
        type: Number,
        default: 200
      },
      stroke: {
        type: String,
        default: 'black'
      },
      strokeWidth: {
        type: Number,
        default: 2
      },
      animationDuration: {
        type: Number,
        default: 0.5 // seconds per stroke
      },
      kerning: {
        type: Boolean,
        default: true
      }
    },
    setup(props) {
      const strokes = ref([]);
  
      const viewBox = computed(() => `0 0 ${props.width} ${props.height}`);
  
      const fontScale = computed(() => props.fontSize / props.font.unitsPerEm);
      const fontAscent = computed(() => props.font.ascender * fontScale.value);
      const fontDescent = computed(() => props.font.descender * fontScale.value);
      const fontHeight = computed(() => fontAscent.value - fontDescent.value);
  
      const baselineY = computed(() => {
        return (props.height + fontHeight.value) / 2 - fontDescent.value;
      });
  
      const formatPathCommand = (cmd) => {
        switch (cmd.type) {
          case 'M':
          case 'L':
            return `${cmd.type}${cmd.x.toFixed(2)},${cmd.y.toFixed(2)}`;
          case 'Q':
            return `${cmd.type}${cmd.x1.toFixed(2)},${cmd.y1.toFixed(2)} ${cmd.x.toFixed(2)},${cmd.y.toFixed(2)}`;
          case 'C':
            return `${cmd.type}${cmd.x1.toFixed(2)},${cmd.y1.toFixed(2)} ${cmd.x2.toFixed(2)},${cmd.y2.toFixed(2)} ${cmd.x.toFixed(2)},${cmd.y.toFixed(2)}`;
          case 'Z':
            return 'Z';
          default:
            console.warn('Unknown path command:', cmd.type);
            return '';
        }
      };
  
      const extractStrokes = () => {
        const x = 0;
        const path = props.font.getPath(props.text, x, baselineY.value, props.fontSize, {
          kerning: props.kerning
        });
  
        let currentStroke = { path: '', color: props.stroke, width: 0, height: 0, x: Infinity, y: Infinity };
        let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity;
  
        path.commands.forEach((cmd, index) => {
          if (cmd.type === 'M' && index !== 0) {
            finishStroke();
          }
  
          currentStroke.path += formatPathCommand(cmd) + ' ';
          
          if (cmd.x !== undefined) {
            minX = Math.min(minX, cmd.x);
            minY = Math.min(minY, cmd.y);
            maxX = Math.max(maxX, cmd.x);
            maxY = Math.max(maxY, cmd.y);
          }
  
          if (cmd.x1 !== undefined) {
            minX = Math.min(minX, cmd.x1);
            minY = Math.min(minY, cmd.y1);
            maxX = Math.max(maxX, cmd.x1);
            maxY = Math.max(maxY, cmd.y1);
          }
  
          if (cmd.x2 !== undefined) {
            minX = Math.min(minX, cmd.x2);
            minY = Math.min(minY, cmd.y2);
            maxX = Math.max(maxX, cmd.x2);
            maxY = Math.max(maxY, cmd.y2);
          }
        });
  
        finishStroke();
  
        function finishStroke() {
          if (currentStroke.path) {
            currentStroke.width = maxX - minX;
            currentStroke.height = maxY - minY;
            currentStroke.x = minX;
            currentStroke.y = minY;
            strokes.value.push(currentStroke);
            currentStroke = { path: '', color: props.stroke, width: 0, height: 0, x: Infinity, y: Infinity };
            minX = Infinity; minY = Infinity; maxX = -Infinity; maxY = -Infinity;
          }
        }
      };
  
      onMounted(() => {
        extractStrokes();
      });
  
      return {
        viewBox,
        strokes
      };
    }
  }
  </script>