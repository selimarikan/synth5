<template>
  <div class="container">
    <div class="keyboard"></div>

    <div class="settingsBar">
      <div class="left">
        <span>Volume: </span>
        <input
          type="range"
          min="0.0"
          max="1.0"
          step="0.01"
          value="0.5"
          list="volumes"
          name="volume"
        />
        <datalist id="volumes">
          <option value="0.0" label="Mute"></option>
          <option value="1.0" label="100%"></option>
        </datalist>
      </div>
      <div class="right">
        <span>Current waveform: </span>
        <select name="waveform">
          <option value="sine">Sine</option>
          <option value="square" selected>Square</option>
          <option value="sawtooth">Sawtooth</option>
          <option value="triangle">Triangle</option>
          <option value="custom">Custom</option>
        </select>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Keyboard",
  methods: {
      
  },
  mounted: function() {
      setup();
  }
};

import FrequencyCalculator from 'frequency-calculator';

let audioContext = new (window.AudioContext || window.webkitAudioContext)();
let oscList = [];
let mainGainNode = null;
let keyboard = document.querySelector(".keyboard");
let wavePicker = document.querySelector("select[name='waveform']");
let volumeControl = document.querySelector("input[name='volume']");

let noteFreq = null;
let customWaveform = null;
let sineTerms = null;
let cosineTerms = null;

function createNoteTable() {
  let noteFreq = [];
  // 2-dim array to keep [octave][note]
  for (let i = 0; i < 7; i++) {
    noteFreq[i] = [];
  }
  console.log(noteFreq)
  for (let iOctave = 0; iOctave < 7; iOctave++) {
      noteFreq[iOctave]["C"] = FrequencyCalculator.calculateFrequencyByNote('C', iOctave);
      noteFreq[iOctave]["C#"] = FrequencyCalculator.calculateFrequencyByNote('CSharp', iOctave);
      noteFreq[iOctave]["D"] = FrequencyCalculator.calculateFrequencyByNote('D', iOctave);
      noteFreq[iOctave]["D#"] = FrequencyCalculator.calculateFrequencyByNote('DSharp', iOctave);
      noteFreq[iOctave]["E"] = FrequencyCalculator.calculateFrequencyByNote('E', iOctave);
      noteFreq[iOctave]["F"] = FrequencyCalculator.calculateFrequencyByNote('F', iOctave);
      noteFreq[iOctave]["F#"] = FrequencyCalculator.calculateFrequencyByNote('FSharp', iOctave);
      noteFreq[iOctave]["G"] = FrequencyCalculator.calculateFrequencyByNote('G', iOctave);
      noteFreq[iOctave]["G#"] = FrequencyCalculator.calculateFrequencyByNote('GSharp', iOctave);
      noteFreq[iOctave]["A"] = FrequencyCalculator.calculateFrequencyByNote('A', iOctave);
      noteFreq[iOctave]["A#"] = FrequencyCalculator.calculateFrequencyByNote('ASharp', iOctave);
      noteFreq[iOctave]["B"] = FrequencyCalculator.calculateFrequencyByNote('B', iOctave);
  }

//   noteFreq[0]["A"] = 27.5;
//   noteFreq[0]["A#"] = 29.135235094880619;
//   noteFreq[0]["B"] = 30.867706328507756;

//   noteFreq[1]["C"] = 32.703195662574829;
//   noteFreq[1]["C#"] = 34.647828872109012;
//   noteFreq[1]["D"] = 36.708095989675945;
//   noteFreq[1]["D#"] = 38.890872965260113;
//   noteFreq[1]["E"] = 41.203444614108741;
//   noteFreq[1]["F"] = 43.653528929125485;
//   noteFreq[1]["F#"] = 46.249302838954299;
//   noteFreq[1]["G"] = 48.999429497718661;
//   noteFreq[1]["G#"] = 51.913087197493142;
//   noteFreq[1]["A"] = 55.0;
//   noteFreq[1]["A#"] = 58.270470189761239;
//   noteFreq[1]["B"] = 561.735412657015513;

  return noteFreq;
}
function setup() {
    keyboard = document.querySelector(".keyboard");
    wavePicker = document.querySelector("select[name='waveform']");
    volumeControl = document.querySelector("input[name='volume']");
    console.log(volumeControl);
    noteFreq = createNoteTable();

  volumeControl.addEventListener("change", changeVolume, false);

  mainGainNode = audioContext.createGain();
  mainGainNode.connect(audioContext.destination);
  mainGainNode.gain.value = volumeControl.value;

  // Create the keys; skip any that are sharp or flat; for
  // our purposes we don't need them. Each octave is inserted
  // into a <div> of class "octave".

  noteFreq.forEach(function (keys, idx) {
    let keyList = Object.entries(keys);
    let octaveElem = document.createElement("div");
    octaveElem.className = "octave";

    keyList.forEach(function (key) {
      if (key[0].length == 1) {
        octaveElem.appendChild(createKey(key[0], idx, key[1]));
      }
    });

    keyboard.appendChild(octaveElem);
  });

  document
    .querySelector("div[data-note='B'][data-octave='1']")
    .scrollIntoView(false);

  sineTerms = new Float32Array([0, 0, 1, 0, 1]);
  cosineTerms = new Float32Array(sineTerms.length);
  customWaveform = audioContext.createPeriodicWave(cosineTerms, sineTerms);

  for (let i = 0; i < 9; i++) {
    oscList[i] = {};
  }
}
function createKey(note, octave, freq) {
  let keyElement = document.createElement("div");
  let labelElement = document.createElement("div");

  keyElement.className = "key";
  keyElement.dataset["octave"] = octave;
  keyElement.dataset["note"] = note;
  keyElement.dataset["frequency"] = freq;

  labelElement.innerHTML = note + "<sub>" + octave + "</sub>";
  keyElement.appendChild(labelElement);

  keyElement.addEventListener("mousedown", notePressed, false);
  keyElement.addEventListener("mouseup", noteReleased, false);
  keyElement.addEventListener("mouseover", notePressed, false);
  keyElement.addEventListener("mouseleave", noteReleased, false);

  return keyElement;
}

function playTone(freq) {
  let osc = audioContext.createOscillator();
  osc.connect(mainGainNode);

  let type = wavePicker.options[wavePicker.selectedIndex].value;

  if (type == "custom") {
    osc.setPeriodicWave(customWaveform);
  } else {
    osc.type = type;
  }
    console.log(freq);
  osc.frequency.value = freq;
  osc.start();

  return osc;
}
function notePressed(event) {
  if (event.buttons & 1) {
    let dataset = event.target.dataset;
    console.log("PRESSED");
    console.log(event);
    if (!dataset["pressed"]) {
      let octave = +dataset["octave"];
      oscList[octave][dataset["note"]] = playTone(dataset["frequency"]);
      dataset["pressed"] = "yes";
    }
  }
}
function noteReleased(event) {
  let dataset = event.target.dataset;

  if (dataset && dataset["pressed"]) {
    let octave = +dataset["octave"];
    oscList[octave][dataset["note"]].stop();
    delete oscList[octave][dataset["note"]];
    delete dataset["pressed"];
  }
}
function changeVolume() {
  mainGainNode.gain.value = volumeControl.value;
}

// setup();
</script>

<style>
.key {
  cursor: pointer;
  font: 16px "Open Sans", "Lucida Grande", "Arial", sans-serif;
  border: 1px solid black;
  border-radius: 2px;
  width: 32px;
  height: 100px;
  text-align: center;
  /* box-shadow: 2px 2px darkgray; */
  display: inline-block;
  position: relative;
  margin-right: 3px;
  user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
}

.key div {
  position: absolute;
  bottom: 0;
  text-align: center;
  width: 100%;
  pointer-events: none;
}

.key div sub {
  font-size: 10px;
  pointer-events: none;
}

.key:hover {
  background-color: #AAAAFF;
}

.key:active {
  background-color: #000077;
  color: #fff;
}

.octave {
  display: inline-block;
  padding: 0 6px 0 0;
}

.settingsBar {
  padding-top: 8px;
  font: 14px "Open Sans", "Lucida Grande", "Arial", sans-serif;
  position: relative;
  vertical-align: middle;
  width: 100%;
  height: 30px;
}

.left {
  width: 50%;
  position: absolute;
  left: 0;
  display: table-cell;
  vertical-align: middle;
}

.left span, .left input {
  vertical-align: middle;
}

.right {
  width: 50%;
  position: absolute;
  right: 0;
  display: table-cell;
  vertical-align: middle;
}

.right span {
  vertical-align: middle;
}

.right input {
  vertical-align: baseline;
}
        
</style>