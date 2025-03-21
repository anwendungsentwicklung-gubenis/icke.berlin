<script>
  import { onMount, onDestroy } from 'svelte';
  
  // Props
  export let text = '';
  export let phrases = null;
  export let speed = 1;
  export let delay = 0;
  export let cursor = true;
  export let loop = false;
  export let loopDelay = 1500;
  export let startDelay = 0;
  export let deleteSpeed = null;
  export let pauseDelay = 500;
  
  let typingElement;
  let interval;
  let timeout;
  let originalContent = '';
  let currentPhrase = 0;
  let allPhrases = [];
  let currentText = '';

  let i = 0;
  let deleting = false;
  let waiting = false;

  onMount(() => {
    if (typingElement) {
      // Initialize phrases
      originalContent = text || '';
      allPhrases = phrases || (text ? [text] : []);
      currentText = allPhrases[currentPhrase] || '';
      
      // Hide original content until animation starts
      typingElement.textContent = '';
      
      // Add cursor if enabled
      if (cursor) {
        typingElement.classList.add('flowertype-cursor');
      }
      
      // Start the animation
      startTyping();
    }
  });
  
  // Start typing animation
  function startTyping() {
    clearInterval(interval);
    
    interval = setInterval(() => {
      if (waiting) return;
      
      if (!deleting && i < currentText.length) {
        // Typing forward
        typingElement.textContent = currentText.substring(0, i + 1);
        i++;
        
        if (i === currentText.length) {
          if (allPhrases.length > 1 && loop) {
            // If multiple phrases, prepare to delete after a pause
            waiting = true;
            timeout = setTimeout(() => {
              deleting = true;
              waiting = false;
            }, pauseDelay);
          } else if (!loop) {
            // If not looping, stop here
            clearInterval(interval);
          }
        }
      } else if (deleting && i > 0) {
        // Deleting
        typingElement.textContent = currentText.substring(0, i - 1);
        i--;
        
        if (i === 0) {
          deleting = false;
          
          // Move to next phrase
          currentPhrase = (currentPhrase + 1) % allPhrases.length;
          currentText = allPhrases[currentPhrase];
          
          // Add delay before starting next phrase
          if (loop) {
            waiting = true;
            timeout = setTimeout(() => {
              waiting = false;
            }, loopDelay);
          }
        }
      }
    }, deleting ? 1000 / ((deleteSpeed || speed) * 10) : 1000 / (speed * 10));
  }
  
  onDestroy(() => {
    clearInterval(interval);
    clearTimeout(timeout);
    
    // Restore original content
    if (typingElement) {
      typingElement.textContent = originalContent;
      
      if (cursor) {
        typingElement.classList.remove('flowertype-cursor');
      }
    }
  });
  
  // Update when props change
  $: if (typingElement && (text || phrases)) {
    clearInterval(interval);
    clearTimeout(timeout);
    
    // Reset animation state
    i = 0;
    deleting = false;
    waiting = false;
    currentPhrase = 0;
    
    // Update content
    originalContent = text || '';
    allPhrases = phrases || (text ? [text] : []);
    currentText = allPhrases[currentPhrase] || '';
    
    // Restart animation
    timeout = setTimeout(startTyping, startDelay + delay);
  }
</script>

<span 
  bind:this={typingElement} 
  class="flowertyper {$$props.class || ''}"
  aria-label={phrases ? phrases.join(', ') : text}
  {...$$restProps}
>
  {text || (phrases && phrases[0]) || ''}
</span>

<style>
  .flowertyper {
    display: inline-block;
    min-height: 1em;
    min-width: 1px;
  }
  
  :global(.flowertype-cursor) {
    border-right: 0.1em solid;
    animation: flowertype-cursor-blink 1.1s step-end infinite;
  }
  
  @keyframes flowertype-cursor-blink {
    from, to { border-color: transparent; }
    50% { border-color: currentColor; }
  }
</style> 