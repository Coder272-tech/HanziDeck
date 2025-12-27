<script>
  import { onMount } from 'svelte';
  import vocab from '$lib/data/vocab.json';

  // Vocab data
  /*
  const vocab = [
    { hanzi: '爱', pinyin: 'ài', meaning: 'to love; affection; to be fond of; to like' },
    { hanzi: '八', pinyin: 'bā', meaning: 'eight; 8' },
    { hanzi: '爸爸', pinyin: 'bà ba', meaning: 'father (informal)' },
    { hanzi: '杯子', pinyin: 'bēi zi', meaning: 'cup; glass' },
    { hanzi: '茶', pinyin: 'chá', meaning: 'tea; tea plant' },
    { hanzi: '出租车', pinyin: 'chū zū chē', meaning: 'taxi' },
    { hanzi: '电脑', pinyin: 'diàn nǎo', meaning: 'computer' },
    { hanzi: '喜欢', pinyin: 'xǐ huan', meaning: 'to like; to be fond of' }
  ];
  */

  let currentIndex = 0;
  let questionStage = 1; // 1: choose pinyin, 2: choose meaning
  let right = 0;
  let wrong = 0;
  let feedback = '';
  let shuffledVocab = [];

  let options = [];

  function shuffle(array) {
    return array.sort(() => Math.random() - 0.5);
  }

  function generateOptions(stage) {
    let correct = stage === 1 ? vocab[currentIndex].pinyin : vocab[currentIndex].meaning;
    let choices = [correct];

    while (choices.length < 4) {
      let candidate = stage === 1
        ? vocab[Math.floor(Math.random() * vocab.length)].pinyin
        : vocab[Math.floor(Math.random() * vocab.length)].meaning;
      if (!choices.includes(candidate)) choices.push(candidate);
    }
    options = shuffle(choices);
  }

  function selectOption(choice) {
    let correct = questionStage === 1 ? vocab[currentIndex].pinyin : vocab[currentIndex].meaning;
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
      if (questionStage === 1) nextCard(); 
      else nextCard();
    }
  }

  function nextCard() {
    feedback = '';
    questionStage = 1;
    currentIndex++;
    if (currentIndex >= 10) {
      alert(`You got ${right} right and ${wrong} wrong out of 10!`);
      right = 0;
      wrong = 0;
      currentIndex = 0;
    }
    generateOptions(1);
  }

  onMount(() => {
  shuffledVocab = shuffle([...vocab]);
  generateOptions(1);
  });
</script>

<div class="min-h-screen flex flex-col items-center justify-center bg-gray-100 p-4">
  <div class="bg-white p-8 rounded-xl shadow-md w-full max-w-md text-center">
    <div class="text-5xl font-bold mb-6">{vocab[currentIndex].hanzi}</div>
    
    <div class="grid grid-cols-2 gap-4">
      {#each options as opt}
        <button 
          class="bg-blue-200 hover:bg-blue-300 p-4 rounded text-lg font-medium"
          on:click={() => selectOption(opt)}>
          {opt}
        </button>
      {/each}
    </div>

    {#if feedback}
      <div class="mt-4 text-3xl">{feedback}</div>
    {/if}
    
    <div class="mt-6 text-gray-700">
      Progress: {currentIndex + 1}/10 | ✅ {right} | ❌ {wrong}
    </div>
  </div>
</div>
