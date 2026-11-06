<script>
  import { onMount } from 'svelte';
  import { afterUpdate } from 'svelte';
  import vocabData from '$lib/data/vocab.json';
  import whiteapeData from '$lib/data/whiteape.json';
  import { base } from '$app/paths';

  // Dataset selection state ('vocab' or 'whiteape')
  let selectedFile = 'vocab';

  let currentIndex = 0;
  let questionStage = 1; // 1: choose pinyin, 2: choose meaning
  let right = 0;
  let wrong = 0;
  let feedback = '';
  let shuffledVocab = [];
  let audio;

  // Computed active dataset based on dropdown choice
  $: activeVocab = selectedFile === 'vocab' ? vocabData : whiteapeData;
  $: TOTAL_QUESTIONS = shuffledVocab.length;

  function shuffle(array) {
    const arr = [...array];
    for (let i = arr.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
    return arr;
  }

  function getAudioUrl(src) {
    if (!src) return null;
    return `${base}${src}`;
  }

  function playAudio() {
    const src = shuffledVocab[currentIndex]?.audio;
    if (!src) return;

    const audioSrc = getAudioUrl(src);

    if (!audio || audio.src !== new URL(audioSrc, window.location.origin).href) {
      audio = new Audio(audioSrc);
    }

    audio.currentTime = 0;
    audio.play();
  }

  function generateOptions(stage) {
    if (!shuffledVocab[currentIndex]) return;
    let correct = stage === 1 ? shuffledVocab[currentIndex].pinyin : shuffledVocab[currentIndex].meaning;
    let choices = [correct];

    while (choices.length < 4 && choices.length < shuffledVocab.length) {
      let candidate = stage === 1
        ? shuffledVocab[Math.floor(Math.random() * shuffledVocab.length)].pinyin
        : shuffledVocab[Math.floor(Math.random() * shuffledVocab.length)].meaning;
      if (!choices.includes(candidate)) choices.push(candidate);
    }
    options = shuffle(choices);
  }

  let options = [];

  function selectOption(choice) {
    let correct = questionStage === 1 ? shuffledVocab[currentIndex].pinyin : shuffledVocab[currentIndex].meaning;
    if (choice === correct) {
      feedback = '✔️';
      if (questionStage === 1) {
        questionStage = 2;
        generateOptions(2);
      } else {
        right++;
        nextCard();
      }
    } else {
      feedback = '❌';
      wrong++;
      nextCard();
    }
  }

  function nextCard() {
    feedback = '';
    questionStage = 1;
    currentIndex++;
    
    if (currentIndex >= shuffledVocab.length) {
      alert(`You got ${right} right and ${wrong} wrong out of ${shuffledVocab.length}!`);
      resetGame();
      return;
    }

    generateOptions(1);
  }

  function resetGame() {
    right = 0;
    wrong = 0;
    currentIndex = 0;
    questionStage = 1;
    feedback = '';
    shuffledVocab = shuffle(activeVocab);
    generateOptions(1);
  }

  // Watch for changes to the dropdown selection and reset/reload the game state
  $: if (activeVocab) {
    resetGame();
  }

  onMount(() => {
    resetGame();
  });

  afterUpdate(() => {
    const card = shuffledVocab[currentIndex];
    if (!card?.audio) return;
    audio = new Audio(getAudioUrl(card.audio));
  });
</script>

<div class="min-h-screen flex flex-col items-center justify-center bg-gray-100 p-4">
  <div class="bg-white p-8 rounded-xl shadow-md w-full max-w-md text-center">
    
    <!-- Dataset Selector Dropdown -->
    <div class="mb-6">
      <label for="dataset-select" class="block text-sm font-medium text-gray-700 mb-1">Select Dataset:</label>
      <select 
        id="dataset-select" 
        bind:value={selectedFile} 
        class="w-full p-2 border border-gray-300 rounded-lg shadow-sm focus:ring-blue-500 focus:border-blue-500 text-gray-800 bg-white"
      >
        <option value="vocab">vocab.json</option>
        <option value="whiteape">whiteape.json</option>
      </select>
    </div>

    {#if shuffledVocab[currentIndex]}
      <div class="relative text-5xl font-bold mb-6 flex items-center justify-center gap-4">
        <span>{shuffledVocab[currentIndex].hanzi}</span>

        <button
          on:click={playAudio}
          class="text-3xl hover:scale-110 transition"
          aria-label="Play pronunciation"
        >
          🔊
        </button>
      </div>

      <div class="grid grid-cols-2 gap-4">
        {#each options as opt}
          <button 
            class="bg-blue-200 hover:bg-blue-300 p-4 rounded text-lg font-medium transition"
            on:click={() => selectOption(opt)}>
            {opt}
          </button>
        {/each}
      </div>
    {:else}
      <div class="text-gray-400">Loading cards…</div>
    {/if}

    {#if feedback}
      <div class="mt-4 text-3xl">{feedback}</div>
    {/if}
    
    <div class="mt-6 text-gray-700">
      Progress: {currentIndex + 1}/{TOTAL_QUESTIONS} | ✅ {right} | ❌ {wrong}
    </div>
  </div>
</div>