<template>
  <div>
    <button @click="connect">Connect</button>
  </div>
  <div>
    <table>
      <tr>
        <th>Raw value</th>
        <th>Classical algorithm</th>
        <th>AI</th>
      </tr>
      <tr>
        <td>{{current.raw}}</td>
        <td :class="current.standard_algorithm === '1' ? 'highlight' : ''">
          {{current.standard_algorithm}}
        </td>
        <td :class="currentState === '1' ? 'highlight' : ''">
          {{currentState}}
        </td>
      </tr>
    </table>
  </div>
  <div>
    <table>
      <tr>
        <th>Press</th>
        <th>Release</th>
        <th>Idle</th>
      </tr>
      <tr>
        <td :class="current.confidence?.push > 90 ? 'highlight' : ''">
          {{current.confidence?.push}}%
        </td>
        <td :class="current.confidence?.release > 90 ? 'highlight' : ''">
          {{current.confidence?.release}}%
        </td>
        <td>{{current.confidence?.idle}}%</td>
      </tr>
    </table>
  </div>
  <!-- <label for="console">
    <textarea name="console" id="console" v-model="log" cols="30" rows="10"></textarea>
  </label> -->
  <!-- <Gauge v-model:value="pushConfidence"></Gauge>
  <Chart></Chart> -->
</template>

<script lang="ts">

/* eslint-disable no-await-in-loop */
/* eslint-disable @typescript-eslint/no-unused-vars */

import { Options, Vue } from 'vue-class-component';
import HelloWorld from './components/HelloWorld.vue';
import Chart from './components/Chart.vue';
import Gauge from './components/Gauge.vue';

@Options({
  components: {
    HelloWorld,
    Chart,
    Gauge,
  },
})
export default class App extends Vue {
  tmp = 10;

  port: unknown = undefined;

  log = '';

  buffer = '';

  current: object = {
    raw: '-',
    standard_algorithm: '-',
    confidence: {
      push: '-',
      release: '-',
      idle: '-',
    },
  };

  currentState = '-';

  pushConfidence = 0;

  static mounted() {
    if ('serial' in navigator) {
      console.log('Awesome, The serial port is supported.');
      // The Web Serial API is supported.
    }
  }

  pushHighlight = '';

  async connect() {
    console.log(`connect button pressed ${this.tmp}`);
    navigator.serial
      .requestPort({ filters: [] })
      .then(async (port) => {
        // Initialize the list of available ports with `ports` on page load.
        this.port = port;
        console.log('saving port');
        await port.open({ baudRate: 115200 });

        while (port.readable) {
          const reader = port.readable.getReader();
          try {
            // eslint-disable-next-line no-constant-condition
            while (true) {
              const { value, done } = await reader.read();
              if (done) {
                // |reader| has been canceled.
                break;
              }
              // Do something with |value|...
              const chunk :string = new TextDecoder().decode(value);
              this.buffer += chunk;
              const position = this.buffer.indexOf('\n');
              if (position !== -1) {
                const line = this.buffer.substring(0, position).replaceAll('\r', '');
                this.buffer = this.buffer.substring(position + 1);
                // console.log(line);
                const parts = line.split(',');
                const data = {
                  raw: parts[0],
                  standard_algorithm: parts[1],
                  confidence: {
                    push: parts[2],
                    release: parts[3],
                    idle: parts[4],
                  },
                };
                this.current = data;
                if (parseInt(data.confidence?.push, 10) > 1) {
                  this.currentState = '1';
                }
                if (parseInt(data.confidence?.release, 10) > 1) {
                  this.currentState = '0';
                }
                // this.pushConfidence = parseInt(data.confidence?.push, 10);
                // if (this.pushConfidence > 90) {
                //   this.pushHighlight = 'highlight';
                // }
                // console.log(data);
                // this.log += line;
              }
            }
          } catch (error) {
            // Handle |error|...
          } finally {
            reader.releaseLock();
          }
        }
      })
      .catch((e: Error) => {
        // The user didn't select a port.
        console.log(e);
      });
  }
}
</script>

<style>
table {
  width: 100%;
}
tr, td {
  text-align: center;
  transition: background-color 0.5s ease-out;
}

tr.highlight, td.highlight {
  transition: background-color 0s ease-out;
}

.highlight {
  background-color: yellow;
}
</style>
