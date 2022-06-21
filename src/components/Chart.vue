<template>
  <v-chart class="chart" :option="option" />
</template>

<script lang="ts">
/* eslint-disable no-use-before-define */
import { use } from 'echarts/core';
import { CanvasRenderer } from 'echarts/renderers';
import { LineChart } from 'echarts/charts';
import {
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  ToolboxComponent,
  GridComponent,
  VisualMapComponent,
  MarkAreaComponent,
} from 'echarts/components';
import VChart, { THEME_KEY } from 'vue-echarts';
import { ref, defineComponent } from 'vue';

use([
  CanvasRenderer,
  LineChart,
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  ToolboxComponent,
  GridComponent,
  VisualMapComponent,
  MarkAreaComponent,
]);

interface DataItem {
  name: string;
  value: [string, number];
}

function randomData(): DataItem {
  now = new Date(+now + oneDay);
  value = value + Math.random() * 21 - 10;
  return {
    name: now.toString(),
    value: [
      [now.getFullYear(), now.getMonth() + 1, now.getDate()].join('/'),
      Math.round(value),
    ],
  };
}

const data: DataItem[] = [];
let now = new Date(1997, 9, 3);
const oneDay = 24 * 3600 * 1000;
let value = Math.random() * 1000;
// eslint-disable-next-line no-plusplus
for (let i = 0; i < 1000; i++) {
  data.push(randomData());
}

export default defineComponent({
  name: 'HelloWorld',
  components: {
    VChart,
  },
  provide: {
    [THEME_KEY]: 'light',
  },
  setup() {
    // const data = [10, 23, 234, 34, 323, 23]
    const option = ref(
      {
        title: {
          text: 'Dynamic Data & Time Axis',
        },
        xAxis: {
          type: 'time',
          splitLine: {
            show: false,
          },
        },
        yAxis: {
          type: 'value',
          boundaryGap: [0, '100%'],
          splitLine: {
            show: false,
          },
        },
        series: [
          {
            name: 'Fake Data',
            type: 'line',
            showSymbol: false,
            data,
            markArea: {
              itemStyle: {
                color: 'rgba(255, 173, 177, 0.4)',
              },
              data: [
                [
                  {
                    name: 'Morning Peak',
                    xAxis: new Date(1997, 10, 3),
                  },
                  {
                    xAxis: new Date(1997, 12, 3),
                  },
                ],
                [
                  {
                    name: 'Evening Peak',
                    xAxis: '17:30',
                  },
                  {
                    xAxis: '21:15',
                  },
                ],
              ],
            },
          },
        ],
      },
    );

    setInterval(() => {
    //   const { data } = option.value.series[0];
      data.shift();
      data.push(randomData());
      // console.log(data)
      // update0ptions()
    }, 100);

    return { option };
  },
});
</script>

<style scoped>
.chart {
  height: 800px;
}
</style>
