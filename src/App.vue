<template>
  <div>
    <button @click="connect">Connect</button>
  </div>
  <div>
    <table>
      <tr>
        <th>Raw value</th>
        <th>Classical algorithm</th>
      </tr>
      <tr>
        <td>{{current.raw}}</td>
        <td>{{current.standard_algorithm}}</td>
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
        <td>{{current.confidence?.push}}%</td>
        <td>{{current.confidence?.release}}%</td>
        <td>{{current.confidence?.idle}}%</td>
      </tr>
    </table>
  </div>
  <!-- <label for="console">
    <textarea name="console" id="console" v-model="log" cols="30" rows="10"></textarea>
  </label> -->
</template>

<script lang="ts">

/* eslint-disable no-await-in-loop */
/* eslint-disable @typescript-eslint/no-unused-vars */

import { Options, Vue } from 'vue-class-component';
import HelloWorld from './components/HelloWorld.vue';

@Options({
  components: {
    HelloWorld,
  },
})
export default class App extends Vue {
  tmp = 10;

  port: unknown = undefined;

  log = '';

  buffer = '';

  current: object = {};

  static mounted() {
    if ('serial' in navigator) {
      console.log('Awesome, The serial port is supported.');
      // The Web Serial API is supported.
    }
  }

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
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
