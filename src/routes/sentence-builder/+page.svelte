<script>
  import { onMount } from 'svelte';
  import vocab from '$lib/data/vocab.json';

  let topVocab = [];
  let sentence = [];

  let dragItem = null;
  let dragSource = null; // 'top' or 'bottom'
  let dragIndex = null;

  // GitHub Pages repository base path
  const BASE_PATH = '/HanziDeck';

  onMount(() => {
    topVocab = vocab;
  });

  function getAudioUrl(src) {
    if (!src) return null;

    // vocab.json contains paths like:
    // /audio/天.mp3
    //
    // GitHub Pages needs:
    // /HanziDeck/audio/天.mp3
    return `${BASE_PATH}${src}`;
  }

  function loadPreset1() {
    const presetString =
      '天上玄靈愛自生靈七思七召三現隨迎代予之形形隨物化應化而成熟急如律令';

    const chars = presetString.split('');

    const matched = chars
      .map(char => topVocab.find(v => v.hanzi === char))
      .filter(Boolean);

    sentence = matched;
  }

  function handleDragStart(item, source, index = null) {
    dragItem = item;
    dragSource = source;
    dragIndex = index;
  }

  function handleDrop() {
    if (!dragItem) return;

    if (dragSource === 'top') {
      // Add a COPY to sentence
      sentence = [...sentence, dragItem];
    }

    if (dragSource === 'bottom') {
      const updated = [...sentence];

      updated.splice(dragIndex, 1);
      updated.push(dragItem);

      sentence = updated;
    }

    dragItem = null;
    dragSource = null;
    dragIndex = null;
  }

  function handleReorder(targetIndex) {
    if (dragSource !== 'bottom') return;

    const updated = [...sentence];

    updated.splice(dragIndex, 1);
    updated.splice(targetIndex, 0, dragItem);

    sentence = updated;

    dragItem = null;
    dragSource = null;
    dragIndex = null;
  }

  function playAudio(src) {
    const audioUrl = getAudioUrl(src);

    if (!audioUrl) return;

    console.log('Playing audio:', audioUrl);

    const audio = new Audio(audioUrl);

    audio.play().catch(error => {
      console.error('Audio playback failed:', error);
    });
  }

  async function playSentence() {
    for (const item of sentence) {
      const audioUrl = getAudioUrl(item.audio);

      if (!audioUrl) continue;

      console.log('Playing sentence audio:', audioUrl);

      const audio = new Audio(audioUrl);

      try {
        await audio.play();

        await new Promise(resolve => {
          audio.onended = resolve;
          audio.onerror = resolve;
        });
      } catch (error) {
        console.error('Audio playback failed:', error);
        break;
      }
    }
  }
</script>

<div class="min-h-screen bg-gray-100 p-6 flex flex-col gap-8">

  <!-- TOP AREA -->
  <div class="bg-white p-6 rounded-xl shadow-md">
    <h2 class="text-xl font-bold mb-4">Characters</h2>

    <div class="flex flex-wrap gap-3">
      {#each topVocab as item}
        <button
          draggable="true"
          on:dragstart={() => handleDragStart(item, 'top')}
          on:click={() => playAudio(item.audio)}
          class="px-4 py-2 bg-blue-200 hover:bg-blue-300 rounded text-2xl font-bold cursor-grab active:cursor-grabbing"
        >
          {item.hanzi}
        </button>
      {/each}
    </div>
  </div>

  <!-- BOTTOM AREA -->
  <div
    class="bg-white p-6 rounded-xl shadow-md min-h-[120px]"
    on:dragover|preventDefault
    on:drop={handleDrop}
  >
    <h2 class="text-xl font-bold mb-4">Sentence Builder</h2>

    <div class="flex flex-wrap gap-3">
      {#each sentence as item, index}
        <button
          draggable="true"
          on:dragstart={() => handleDragStart(item, 'bottom', index)}
          on:dragover|preventDefault
          on:drop={() => handleReorder(index)}
          on:click={() => playAudio(item.audio)}
          class="px-4 py-2 bg-green-200 hover:bg-green-300 rounded text-2xl font-bold cursor-move"
        >
          {item.hanzi}
        </button>
      {/each}
    </div>
  </div>

  <!-- CONTROLS -->
  <div class="flex gap-4">

    <button
      on:click={playSentence}
      class="px-6 py-3 bg-purple-400 hover:bg-purple-500 rounded text-white font-bold"
    >
      ▶ Play Sentence
    </button>

    <button
      on:click={() => (sentence = [])}
      class="px-6 py-3 bg-red-400 hover:bg-red-500 rounded text-white font-bold"
    >
      Clear
    </button>

    <button
      on:click={loadPreset1}
      class="px-6 py-3 bg-blue-500 hover:bg-blue-600 rounded text-white font-bold"
    >
      1
    </button>

  </div>
</div>