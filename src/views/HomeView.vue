<template>
  <main>
    <form
      class="no-print"
      @submit.prevent
    >
      <label for="chordpro-input">Chord chart in ChordPro format:</label>
      <textarea
        ref="textareaRef"
        v-model="input"
        id="chordpro-input"
        rows="10"
      />

      <fieldset>
        <legend>Settings</legend>

        <div>
          <label for="chordpro-transpose">Transpose: </label>
          <input
            type="number"
            id="chordpro-transpose"
            v-model="transposition"
            max="12"
            min="-12"
            required
          >
        </div>

        <div>
          <label for="fontSizeInput">Font size:</label>
          <input
            id="fontSizeInput"
            min="1"
            type="number"
            v-model="fontSize"
          />
        </div>

        <div>
          <label for="columnSelect">Number of columns:</label>
          <select
            id="columnSelect"
            name="Columns"
            v-model="columns"
          >
            <option
              :value="1"
              default
            >
              One
            </option>
            <option
              :value="2"
            >
              Two
            </option>
          </select>
        </div>
        <button
          @click="onPrint()"
          type="button"
          class="print-button"
        >
          Print
        </button>
        <button
          @click="onSave()"
          type="button"
          class="save-button"
        >
          Save
        </button>		
        <button
          @click="onMidi()"
          type="button"
          class="midi-button"
        >
          Midi
        </button>	
        <button
          @click="onPlay()"
          type="button"
          class="play-button"
        >
          Play
        </button>			
      </fieldset>
    </form>
    <LoaderBars
      style="margin: 0 auto; "
      v-if="isLoading"
    />
    <div
      class="print-view"
      :style="{
        '--font-size': `${validatedFontSize}px`
      }"
    >
      <ChordChart
        :columns="columns"
        :transposition="validatedTransposition"
        @click="onChordChartClick"
      />
    </div>
  </main>
</template>

<script setup>
const DEFAULT_FONT_SIZE = 12;
import { computed, inject, defineAsyncComponent, provide, ref, watch } from 'vue';
const ChordChart = defineAsyncComponent(() => import('../components/ChordChart.vue'));
const LoaderBars = defineAsyncComponent(() => import('../components/LoaderBars.vue'));
const input = inject('chordProInput');
const isLoading = inject('isLoading');

const caretPosition = ref(0);
const columns = ref(1);
const fontSize = ref(DEFAULT_FONT_SIZE);
const textareaRef = ref(null);
const transposition = ref(0);

provide('caretPosition', caretPosition);

watch(input, () => {
  caretPosition.value = getCaretPosition();
});

const validatedFontSize = computed(() => {
  if (fontSize.value > 0) {
    return fontSize.value;
  }
  return DEFAULT_FONT_SIZE;
});

const validatedTransposition = computed(() => {
  const transpositionWithDefault = transposition.value || 0;
  return Math.min(12, Math.max(-12, transpositionWithDefault || 0));
});

function getCaretPosition() {
  return textareaRef.value.selectionStart;
}

function onChordChartClick(event) {
  const closestElementWithIndex = event.target.closest('[data-index]');
  if (closestElementWithIndex) {
    const index = parseInt(closestElementWithIndex.getAttribute('data-index'), 10);
    caretPosition.value = index;
    textareaRef.value.setSelectionRange(caretPosition.value, caretPosition.value);
  }
  textareaRef.value.focus();
};

function onPlay() {
	const song = document.getElementById("chordpro-input").value;
	console.debug("Convert to midi and play", song);
	window.parent.postMessage(song, "*");
}

function onSave() {
	console.debug("Save chordpro file");
	const choFilename = prompt("Enter filename");
	
	if (choFilename) {
		const blob = new Blob([document.getElementById("chordpro-input").value], { type: 'text/plain' }); 	
		const anchor = document.createElement('a');
		anchor.href = window.URL.createObjectURL(blob);
		anchor.style = "display: none;";
		anchor.download = choFilename + ".cho";
		document.body.appendChild(anchor);
		anchor.click();
		window.URL.revokeObjectURL(anchor.href); 
		document.body.removeChild(anchor);			
	}
  
}
	
async function onMidi() {
	console.debug("Generate Midi file");
	const midiFilename = prompt("Enter filename");
	
	if (midiFilename) {
		const url = "/orinayo/cp2midi";
		const response = await fetch(url, {method: "POST", body: document.getElementById("chordpro-input").value});
		const blob = await response.blob();		
		const anchor = document.createElement('a');
		anchor.href = window.URL.createObjectURL(blob);
		anchor.style = "display: none;";
		anchor.download = midiFilename + ".mid";
		document.body.appendChild(anchor);
		anchor.click();
		window.URL.revokeObjectURL(anchor.href);
		document.body.removeChild(anchor);		
	}
  
}

function onPrint() {
  window.print();
}
</script>

<style scoped>
main {
  font-size: 16px;

  @media screen and (max-width: 1280px) {
    font-size: 14px;
  }

  @media screen and (max-width: 1024px) {
    font-size: 13px;
  }

  @media screen and (max-width: 875px) {
    font-size: 12px;
  }

  @media screen and (max-width: 768px) {
    font-size: 11px;
  }

  @media screen and (max-width: 576px) {
    font-size: 10px;
  }

  @media screen and (max-width: 475px) {
    font-size: 9px;
  }
}

label {
  display: block;
  font-size: 12px;
  font-weight: bold;
  margin-bottom: 4px;
}

textarea {
  font-family: monospace;
  font-size: 14px;
  width: 100%;
}

input:invalid {
  border-color: red;
}

fieldset {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;

  > * {
    flex: 1;
  }
}

.print-view {
  font-size: var(--font-size, 16pt);

  @media print {
    outline: none;
    padding: 0;
    transform: none;
  }
}
</style>
