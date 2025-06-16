<template>
  <div>
    <h1>Web Audio API con Vue</h1>
    <section>
      <h2>Oscilador</h2>
      <button @click="startOscillator">Iniciar Oscilador</button>
      <button @click="stopOscillator">Detener Oscilador</button>
      <div>
        <label for="waveform">Tipo de Onda:</label>
        <select id="waveform" v-model="waveform" @change="changeWaveform">
          <option value="sine">Senoidal (sine)</option>
          <option value="square">Cuadrada (square)</option>
          <option value="sawtooth">Dientes de Sierra (sawtooth)</option>
          <option value="triangle">Triangular (triangle)</option>
        </select>
      </div>
      <div>
        <label for="pan">Panorámica (Izquierda/Derecha): </label>
        <input
          type="range"
          id="panner"
          min="-1"
          max="1"
          step="0.01"
          v-model="panValue"
        />
        <span>{{ panValue }}</span>
      </div>
    </section>

    <hr />

    <section>
      <h2>Archivo de Audio</h2>
      <div>
        <label for="file-input">Selecciona un archivo de audio: </label>
        <input
          type="file"
          id="file-input"
          accept="audio/*"
          @change="handleFileChange"
        />
      </div>
      <div v-if="audioFileUrl">
        <audio
          controls
          :src="audioFileUrl"
          style="margin-top: 10px; width: 100%"
        ></audio>
      </div>
    </section>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // Contexto de audio
      audioCtx: null,
      // Oscilador para el tono
      oscillator: null,
      // Nodo para la panorámica
      panner: null,
      // Valor de la panorámica, vinculado al slider con v-model
      panValue: 0,
      // URL para el archivo de audio seleccionado
      audioFileUrl: null,
      // Tipo de onda para el oscilador
      waveform: "sine",
    };
  },
  mounted() {
    // Inicializamos el AudioContext al montar el componente.
    try {
      this.audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      console.log("AudioContext inicializado");

      // Creamos el nodo de panorámica y lo conectamos a la salida final
      this.panner = this.audioCtx.createStereoPanner();
      this.panner.connect(this.audioCtx.destination);
      console.log("Panner node inicializado y conectado.");
    } catch (e) {
      console.error("Web Audio API no es soportado en este navegador", e);
    }
  },
  methods: {
    /* Funciones para el oscilador */
    startOscillator() {
      if (this.oscillator || !this.audioCtx) return; // Evitar múltiples instancias
      this.oscillator = this.audioCtx.createOscillator();
      this.oscillator.type = this.waveform;
      this.oscillator.frequency.setValueAtTime(
        440,
        this.audioCtx.currentTime
      ); // Frecuencia de 440Hz (nota La)

      // Conectamos el oscilador AL PANNER, no a la salida directamente
      this.oscillator.connect(this.panner);
      this.oscillator.start();
      console.log("Oscilador iniciado y conectado al panner.");
    },
    stopOscillator() {
      if (this.oscillator) {
        this.oscillator.stop();
        this.oscillator.disconnect(); // Desconecta del panner
        this.oscillator = null;
        console.log("Oscilador detenido");
      }
    },
    changeWaveform() {
      if (this.oscillator) {
        this.oscillator.type = this.waveform; // Cambia el tipo de onda en tiempo real
      }
    },

    /* Función para cargar archivo de audio y mostrarlo en el tag <audio> */
    handleFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        // Si hay una URL anterior, la revocamos para liberar memoria
        if (this.audioFileUrl) {
          URL.revokeObjectURL(this.audioFileUrl);
        }
        // Creamos una URL para el archivo local
        this.audioFileUrl = URL.createObjectURL(file);
        console.log(
          "Archivo de audio listo para reproducir:",
          this.audioFileUrl
        );
      }
    },
  },
  watch: {
    // Este watcher observa cambios en panValue (movido por el slider)
    // y actualiza el nodo de audio correspondiente.
    panValue(newPan) {
      if (this.panner) {
        // Usamos setValueAtTime para una transición suave
        this.panner.pan.setValueAtTime(newPan, this.audioCtx.currentTime);
      }
    },
  },
};
</script>

<style scoped>

button {
  margin: 10px;
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
  border-radius: 3px;
  background-color: antiquewhite;
}
section {
  background-color: rgba(0,0,0,0.1);
  margin-bottom: 20px;
}
label {
  margin-right: 10px;
}
select{
  background-color: rgba(0,0,0,0.2);
}
</style>