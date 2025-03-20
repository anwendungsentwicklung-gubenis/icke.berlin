<script>
  import { onMount } from 'svelte';
  let title = "icke.berlin";
  let imageWidth = 450; // Larger default size for SSR
  let showBubble = false;
  
  // More direct resize handling
  function handleResize() {
    if (typeof window !== 'undefined') {
      const width = window.innerWidth;
      
      // Show speech bubble when viewport is small - increased threshold
      showBubble = width < 800;
      
      // Adjusted scaling with larger initial sizes
      if (width > 1200) {
        imageWidth = Math.min(500, width / 3); // Larger for big screens - 33% of width
      } else if (width > 768) {
        imageWidth = Math.min(450, width / 2.5); // Medium screens - 40% of width
      } else {
        imageWidth = Math.min(350, width / 2); // Small screens - 50% of width
      }
      
      // Ensure integer value
      imageWidth = Math.floor(imageWidth);
      
      // Debug for testing
      console.log('Window width:', width, 'Image width:', imageWidth);
    }
  }
  
  onMount(() => {
    // Initial sizing
    handleResize();
    
    // Set up resize listener directly
    window.addEventListener('resize', handleResize);
    
    // Initialize canvas for noise
    const canvas = document.getElementById('noise-canvas');
    const ctx = canvas.getContext('2d');
    
    // Set canvas size to window size
    function resizeCanvas() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      generateNoise();
    }
    
    // Generate the noise pattern
    function generateNoise() {
      // Clear canvas and set background color
      ctx.fillStyle = '#fcb766';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      
      // Create more irregular, less circular flakes
      const flakeCount = Math.floor(canvas.width * canvas.height / 12000);
      
      // BLACK FLAKES
      for (let i = 0; i < flakeCount; i++) {
        const x = Math.random() * canvas.width;
        const y = Math.random() * canvas.height;
        
        // Creating irregular shapes
        const opacity = Math.random() * 0.3 + 0.5; // 0.5 to 0.8 opacity
        const size = Math.random() * 1.2 + 0.3; // 0.3 to 1.5px base size
        
        // Mainly use irregular shapes and lines
        const shapeType = Math.random() < 0.1 ? 0 : Math.floor(Math.random() * 3) + 1; // Rarely use circles
        
        ctx.fillStyle = `rgba(0, 0, 0, ${opacity})`;
        
        if (shapeType === 0) {
          // Small irregular dot (rare)
          const radius = size * 0.8;
          ctx.beginPath();
          ctx.arc(x, y, radius, 0, Math.PI * 2);
          ctx.fill();
        } else if (shapeType === 1) {
          // Irregular flake
          ctx.beginPath();
          ctx.moveTo(x, y);
          
          // Create small irregular polygon
          const points = Math.floor(Math.random() * 3) + 3; // 3-5 points
          const baseRadius = size;
          
          for (let j = 0; j <= points; j++) {
            const angle = (j / points) * Math.PI * 2;
            const radius = baseRadius * (0.7 + Math.random() * 0.6); // Vary radius
            const ptX = x + Math.cos(angle) * radius;
            const ptY = y + Math.sin(angle) * radius;
            
            if (j === 0) {
              ctx.moveTo(ptX, ptY);
            } else {
              ctx.lineTo(ptX, ptY);
            }
          }
          
          ctx.fill();
        } else if (shapeType === 2) {
          // Very small, slightly elongated rectangle
          const width = size * (Math.random() * 0.6 + 0.4);
          const height = size * (Math.random() * 0.6 + 0.4);
          ctx.fillRect(x, y, width, height);
        } else {
          // Short line with varied thickness
          const angle = Math.random() * Math.PI * 2;
          const length = size * (Math.random() * 1.5 + 0.5);
          
          ctx.beginPath();
          ctx.moveTo(x, y);
          ctx.lineTo(x + Math.cos(angle) * length, y + Math.sin(angle) * length);
          ctx.lineWidth = Math.random() * 0.5 + 0.1; // Thinner lines
          ctx.strokeStyle = `rgba(0, 0, 0, ${opacity})`;
          ctx.stroke();
        }
      }
      
      // WHITE NOISE - simplified and more visible
      const whiteFlakeCount = Math.floor(canvas.width * canvas.height / 10000);
      for (let i = 0; i < whiteFlakeCount; i++) {
        const x = Math.random() * canvas.width;
        const y = Math.random() * canvas.height;
        
        // More visible opacity
        const opacity = Math.random() * 0.2 + 0.4; // 0.4 to 0.6 opacity (more visible)
        const size = Math.random() * 1.5 + 0.5; // 0.5 to 2px (larger)
        
        // Simple shape types - square or dot
        const shapeType = Math.floor(Math.random() * 3);
        
        ctx.fillStyle = `rgba(255, 255, 255, ${opacity})`;
        
        if (shapeType === 0) {
          // White dot
          ctx.beginPath();
          ctx.arc(x, y, size * 0.8, 0, Math.PI * 2);
          ctx.fill();
        } else if (shapeType === 1) {
          // Small white square
          ctx.fillRect(x, y, size, size);
        } else {
          // Short white line
          const angle = Math.random() * Math.PI * 2;
          const length = size * 1.5;
          
          ctx.beginPath();
          ctx.moveTo(x, y);
          ctx.lineTo(x + Math.cos(angle) * length, y + Math.sin(angle) * length);
          ctx.lineWidth = size * 0.4;
          ctx.strokeStyle = `rgba(255, 255, 255, ${opacity})`;
          ctx.stroke();
        }
      }
    }
    
    // Initial setup
    resizeCanvas();
    
    // Handle window resize for canvas
    window.addEventListener('resize', resizeCanvas);
    
    // Clean up on component unmount
    return () => {
      window.removeEventListener('resize', handleResize);
      window.removeEventListener('resize', resizeCanvas);
    };
  });
</script>

<svelte:head>
  <title>{title}</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Fuzzy+Bubbles:wght@700&display=swap" rel="stylesheet">
</svelte:head>

<canvas id="noise-canvas"></canvas>
<div class="image-container">
  <div class="image-with-bubble">
    <img src="/icke.png" alt="icke" class="bottom-image" style="width: {imageWidth}px;" />
    {#if showBubble}
      <div class="comic-bubble">
        <span class="comic-text">Viewport</span>
        <span class="comic-text">size</span>
        <span class="comic-text">matters!</span>
      </div>
    {/if}
  </div>
</div>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    overflow: hidden;
    position: relative;
    background-color: #fcb766; /* Updated background color */
  }
  
  #noise-canvas {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 1;
  }

  .image-container {
    position: fixed;
    bottom: -1px;
    left: 0;
    width: 100%;
    display: flex;
    justify-content: flex-start;
    align-items: flex-end;
    z-index: 2; /* Above the noise layer */
  }
  
  .image-with-bubble {
    position: relative;
    display: inline-block;
  }

  .bottom-image {
    display: block;
    height: auto;
    max-width: 100%;
    transition: width 0.3s ease-in-out;
  }
  
  /* Enhanced comic style speech bubble */
  .comic-bubble {
    position: absolute;
    top: 55%; /* Adjusted to align with mouth */
    left: 100%; /* Position at the right edge of the container */
    transform: translateY(-50%) rotate(2deg); /* Center vertically and add slight rotation */
    min-width: 280px;
    background-color: white;
    border: 5px solid black;
    border-radius: 40px;
    padding: 15px 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    box-shadow: 6px 6px 0 rgba(0, 0, 0, 0.8);
    z-index: 3;
    margin-left: 5px; /* Small gap between image and bubble */
    animation: comic-pop 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.5) forwards;
  }
  
  .comic-text {
    font-family: 'Fuzzy Bubbles', cursive;
    font-size: 30px;
    font-weight: 700;
    line-height: 1.1;
    text-align: center;
    color: black;
    text-transform: none;
    letter-spacing: 1px;
    transform: skew(-3deg);
    display: inline-block;
    position: relative;
    z-index: 4;
  }
  
  .comic-text:nth-child(2) {
    font-size: 34px;
    transform: skew(3deg);
  }
  
  .comic-text:nth-child(3) {
    font-size: 32px;
    transform: skew(-2deg);
  }
  
  /* Triangle pointer */
  .comic-bubble::after {
    content: '';
    position: absolute;
    top: 50%;
    right: 99%; /* Positioned at the edge */
    width: 0;
    height: 0;
    border-top: 25px solid transparent;
    border-bottom: 25px solid transparent;
    border-right: 40px solid white; /* Point to the left */
    transform: translateY(-50%);
    left: auto; /* Ensure right positioning works */
  }
  
  /* Border for triangle */
  .comic-bubble::before {
    content: '';
    position: absolute;
    top: 50%;
    right: calc(99% + 2px); /* Adjusted to match the pointer */
    width: 0;
    height: 0;
    border-top: 28px solid transparent;
    border-bottom: 28px solid transparent;
    border-right: 44px solid black; /* Point to the left with black border */
    transform: translateY(-50%);
    z-index: -1;
    left: auto; /* Ensure right positioning works */
  }
  
  @keyframes comic-pop {
    0% {
      opacity: 0;
      transform: translateY(-50%) scale(0.1) rotate(2deg);
    }
    60% {
      opacity: 1;
      transform: translateY(-50%) scale(1.1) rotate(2deg);
    }
    75% {
      transform: translateY(-50%) scale(0.95) rotate(2deg);
    }
    90% {
      transform: translateY(-50%) scale(1.05) rotate(2deg);
    }
    100% {
      transform: translateY(-50%) scale(1) rotate(2deg);
    }
  }
</style>
