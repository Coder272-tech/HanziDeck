<script>
  import { onMount } from 'svelte';
  import { afterUpdate } from 'svelte';
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
  let audio;
  const TOTAL_QUESTIONS = 15;


  let options = [];

  function shuffle(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
}




	function playAudio() {
	  const src = shuffledVocab[currentIndex]?.audio;
	  if (!src) return;

	  if (!audio || audio.src !== location.origin + src) {
		audio = new Audio(src);
	  }

	  audio.currentTime = 0;
	  audio.play();
	}


  function generateOptions(stage) {
    let correct = stage === 1 ? shuffledVocab[currentIndex].pinyin : shuffledVocab[currentIndex].meaning;
    let choices = [correct];

   // while (choices.length < 4) {
	while (choices.length < 4 && choices.length < shuffledVocab.length) {
      let candidate = stage === 1
        ? shuffledVocab[Math.floor(Math.random() * shuffledVocab.length)].pinyin
        : shuffledVocab[Math.floor(Math.random() * shuffledVocab.length)].meaning;
      if (!choices.includes(candidate)) choices.push(candidate);
    }
    options = shuffle(choices);
  }

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
      if (questionStage === 1) nextCard(); 
      else nextCard();
    }
  }

  function nextCard() {
    feedback = '';
    questionStage = 1;
    currentIndex++;
	
	if (currentIndex >= TOTAL_QUESTIONS) {
	  alert(`You got ${right} right and ${wrong} wrong out of ${TOTAL_QUESTIONS}!`);
	  right = 0;
	  wrong = 0;
	  currentIndex = 0;
	}

    generateOptions(1);
  }

  onMount(() => {
	  // If you want entries 11–23 inclusive (humans count from 1):
	  // const selectedVocab = vocab.slice(10, 23);
	  const selectedVocab = vocab;
	  shuffledVocab = shuffle([...selectedVocab]);
	  generateOptions(1);
  });
  


	afterUpdate(() => {
	  const card = shuffledVocab[currentIndex];
	  if (!card?.audio) return;

	  audio = new Audio(card.audio);
	});


</script>

<div class="min-h-screen flex flex-col items-center justify-center bg-gray-100 p-4">
  <div class="bg-white p-8 rounded-xl shadow-md w-full max-w-md text-center">
  
  
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
        class="bg-blue-200 hover:bg-blue-300 p-4 rounded text-lg font-medium"
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
