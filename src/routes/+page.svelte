<script>
  import { onMount } from 'svelte';
  import { fade, fly } from 'svelte/transition';
  import { projects } from '$lib/projects';
  import FlowerTyper from '$lib/flowertyper/FlowerTyper.svelte';
  import FlowerLogo from '$lib/flowertyper/FlowerLogo.svelte';
  
  let title = "icke.berlin";
  let imageWidth = 450; // Larger default size for SSR
  let showBubble = false;
  let isFirstLoad = true; // Track initial load
  let currentProjectIndex = 0; // For mobile swipe view
  let isMobile = false; // Track if we're on mobile
  
  // Example phrases for the FlowerTyper
  const welcomePhrases = [
    "Welcome to icke.berlin",
    "Explore my projects",
    "Check out what I build",
    "Stay a while and browse"
  ];
  
  // More direct resize handling
  function handleResize() {
    if (typeof window !== 'undefined') {
      const width = window.innerWidth;
      
      // Show speech bubble when viewport is small - increased threshold
      showBubble = width < 800;
      
      // Track if we're on mobile for swipe view
      isMobile = width < 768;
      
      let newWidth;
      // Adjusted scaling with larger initial sizes
      if (width > 1200) {
        newWidth = Math.min(500, width / 3); // Larger for big screens - 33% of width
      } else if (width > 768) {
        newWidth = Math.min(450, width / 2.5); // Medium screens - 40% of width
      } else {
        newWidth = Math.min(350, width / 2); // Small screens - 50% of width
      }
      
      // Ensure integer value
      newWidth = Math.floor(newWidth);
      
      // Only apply animation if not first load
      if (!isFirstLoad) {
        imageWidth = newWidth;
      } else {
        imageWidth = newWidth;
        isFirstLoad = false;
      }
    }
  }
  
  // Handle swipe functionality
  let touchStartX = 0;
  let touchEndX = 0;
  let swipeDirection = null; // 'left' or 'right'
  let isTransitioning = false;
  
  function handleTouchStart(e) {
    touchStartX = e.changedTouches[0].screenX;
  }
  
  function handleTouchMove(e) {
    // Don't process if already transitioning
    if (isTransitioning) return;
    
    const currentX = e.changedTouches[0].screenX;
    const diff = currentX - touchStartX;
    
    if (Math.abs(diff) > 20) {
      swipeDirection = diff > 0 ? 'right' : 'left';
    }
  }
  
  function handleTouchEnd(e) {
    if (isTransitioning) return;
    
    touchEndX = e.changedTouches[0].screenX;
    handleSwipe();
  }
  
  function handleSwipe() {
    const swipeThreshold = 50; // Minimum distance to register as swipe
    const diff = touchEndX - touchStartX;
    
    if (Math.abs(diff) < swipeThreshold) return; // Not a significant swipe
    
    // Set transitioning state
    isTransitioning = true;
    
    if (diff > 0) {
      // Swipe right - show previous project
      swipeDirection = 'right';
      prevProject();
    } else {
      // Swipe left - show next project
      swipeDirection = 'left';
      nextProject();
    }
    
    // Reset transitioning state after animation
    setTimeout(() => {
      isTransitioning = false;
    }, 450); // Slightly longer than the total animation duration
  }
  
  function nextProject() {
    if (currentProjectIndex < projects.length - 1) {
      currentProjectIndex++;
    } else {
      currentProjectIndex = 0; // Loop back to first
    }
  }
  
  function prevProject() {
    if (currentProjectIndex > 0) {
      currentProjectIndex--;
    } else {
      currentProjectIndex = projects.length - 1; // Loop to last
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
        
        // Creating irregular shapes with maximum opacity
        const opacity = Math.random() * 0.05 + 0.95; // 0.95 to 1.0 opacity (maximum visibility)
        const size = Math.random() * 1.5 + 0.5; // 0.5 to 2.0px (slightly larger)
        
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
      const whiteFlakeCount = Math.floor(canvas.width * canvas.height / 8000); // Increased density
      for (let i = 0; i < whiteFlakeCount; i++) {
        const x = Math.random() * canvas.width;
        const y = Math.random() * canvas.height;
        
        // Maximum opacity for white flakes
        const opacity = Math.random() * 0.05 + 0.95; // 0.95 to 1.0 opacity (maximum visibility)
        const size = Math.random() * 2.0 + 0.8; // 0.8 to 2.8px (larger)
        
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
  <link href="https://fonts.googleapis.com/css2?family=Fuzzy+Bubbles:wght@700&family=Jost:wght@300;400;600&display=swap" rel="stylesheet">
</svelte:head>

<canvas id="noise-canvas"></canvas>

<!-- Projects showcase -->
<div class="projects-container">
  <!-- Mobile swipe view -->
  {#if isMobile}
    <div 
      class="mobile-project-swipe {swipeDirection ? `swiping-${swipeDirection}` : ''}" 
      on:touchstart={handleTouchStart}
      on:touchmove={handleTouchMove} 
      on:touchend={handleTouchEnd}
    >
      <div class="swipe-indicator">
        <div class="indicator-dots">
          {#each projects as _, i}
            <div class="dot {i === currentProjectIndex ? 'active' : ''}"></div>
          {/each}
        </div>
        <div class="swipe-hint">Swipe to browse projects</div>
      </div>
      
      <div class="card-container">
        {#key currentProjectIndex}
          <div class="dictionary-card" 
            style="--accent-color: {projects[currentProjectIndex].color};"
            in:fly={{ x: swipeDirection === 'right' ? 300 : -300, duration: 400, delay: 50, opacity: 0.2 }}
            out:fly={{ x: swipeDirection === 'right' ? -300 : 300, duration: 300, opacity: 0.2 }}
          >
            <div class="mobile-card-content">
              <div class="content-area">
                <div class="term-container">
                  <h2 class="term">{projects[currentProjectIndex].name}</h2>
                  <div class="pronunciation-container">
                    <span class="pronunciation">/{projects[currentProjectIndex].pronunciation || projects[currentProjectIndex].name.toLowerCase()}/</span>
                    <span class="part-of-speech">noun</span>
                  </div>
                </div>
                
                <div class="definition-container">
                  <div class="definition">
                    <span class="definition-text">{projects[currentProjectIndex].description}</span>
                  </div>
                  {#if projects[currentProjectIndex].keywords && projects[currentProjectIndex].keywords.length > 0}
                    <div class="keywords-container">
                      {#each projects[currentProjectIndex].keywords as keyword}
                        <span class="keyword" style="background-color: {projects[currentProjectIndex].color}20; color: {projects[currentProjectIndex].color};">
                          {keyword}
                        </span>
                      {/each}
                    </div>
                  {/if}
                  <div class="usage">
                    <a href="https://{projects[currentProjectIndex].domain}" target="_blank" rel="noopener noreferrer" class="usage-example">
                      {projects[currentProjectIndex].domain}
                    </a>
                  </div>
                </div>
              </div>
              
              <div class="mobile-logo-area">
                <div class="logo-container">
                  {#if projects[currentProjectIndex].useTyper}
                    <div class="viveros-logo">
                      <FlowerLogo 
                        text="V" 
                        color={projects[currentProjectIndex].color}
                      />
                    </div>
                  {:else if projects[currentProjectIndex].logo}
                    <img src={projects[currentProjectIndex].logo} alt="{projects[currentProjectIndex].name} logo" class="project-logo" />
                  {/if}
                </div>
              </div>
            </div>
          </div>
        {/key}
      </div>
      
      <div class="mobile-navigation">
        <button class="nav-button prev" on:click={prevProject} aria-label="Previous project">←</button>
        <button class="nav-button next" on:click={nextProject} aria-label="Next project">→</button>
      </div>
    </div>
  {:else}
    <!-- Desktop grid view -->
    <div class="projects-grid">
      {#each projects as project}
        <div class="dictionary-card" style="--accent-color: {project.color};">
          <div class="content-area">
            <div class="term-container">
              <h2 class="term">{project.name}</h2>
              <div class="pronunciation-container">
                <span class="pronunciation">/{project.pronunciation || project.name.toLowerCase()}/</span>
                <span class="part-of-speech">noun</span>
              </div>
            </div>
            
            <div class="definition-container">
              <div class="definition">
                <span class="definition-text">{project.description}</span>
              </div>
              {#if project.keywords && project.keywords.length > 0}
                <div class="keywords-container">
                  {#each project.keywords as keyword}
                    <span class="keyword" style="background-color: {project.color}20; color: {project.color};">
                      {keyword}
                    </span>
                  {/each}
                </div>
              {/if}
              <div class="usage">
                <a href="https://{project.domain}" target="_blank" rel="noopener noreferrer" class="usage-example">
                  {project.domain}
                </a>
              </div>
            </div>
          </div>
          
          <div class="logo-area">
            <div class="logo-container">
              {#if project.useTyper}
                <div class="viveros-logo">
                  <FlowerLogo 
                    text="V" 
                    color={project.color}
                  />
                </div>
              {:else if project.logo}
                <img src={project.logo} alt="{project.name} logo" class="project-logo" />
              {/if}
            </div>
          </div>
        </div>
      {/each}
    </div>
  {/if}
</div>

<div class="image-container">
  <div class="image-with-bubble">
    <img 
      src="./icke.png" 
      alt="icke" 
      class="bottom-image" 
      style="width: {imageWidth}px; transition: {isFirstLoad ? 'none' : 'width 0.3s ease-in-out'};" 
    />
    {#if showBubble}
      <div class="comic-bubble">
        <span class="comic-text">Viewport</span>
        <span class="comic-text">size</span>
        <span class="comic-text">matters!</span>
      </div>
    {/if}
  </div>
</div>

<footer class="copyright">
  <span>Icke.Berlin</span> © {new Date().getFullYear()}
</footer>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    overflow-x: hidden;
    position: relative;
    background-color: #fcb766;
    font-family: 'Jost', sans-serif;
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
    bottom: -2px;
    left: 0;
    width: 100%;
    display: flex;
    justify-content: flex-start;
    align-items: flex-end;
    z-index: 2; /* Lower z-index by default */
    pointer-events: none; /* Let clicks go through to the projects below */
  }
  
  /* On small screens, increase z-index to show above projects */
  @media (max-width: 768px) {
    .image-container {
      z-index: 4; /* Higher z-index on small screens */
    }
  }
  
  .image-with-bubble {
    position: relative;
    display: inline-block;
    pointer-events: auto; /* Restore pointer events for the image and bubble */
  }

  .bottom-image {
    display: block;
    height: auto;
    max-width: 100%;
    border-radius: 2px;
  }
  
  .projects-container {
    position: fixed;
    top: 8px; /* Reduced from 60px */
    left: 8px; /* Reduced from 60px */
    right: 8px; /* Reduced from 60px */
    z-index: 3;
    width: auto;
    bottom: 60px; /* Keep bottom padding for the image */
    overflow-y: auto;
    overflow-x: hidden; /* Prevent horizontal scrolling while allowing shadows to be visible */
    scrollbar-width: thin;
    scrollbar-color: rgba(0, 0, 0, 0.2) transparent;
    padding: 5px; /* Add small padding to prevent shadow clipping */
  }
  
  /* Custom scrollbar for webkit browsers */
  .projects-container::-webkit-scrollbar {
    width: 8px;
  }
  
  .projects-container::-webkit-scrollbar-track {
    background: transparent;
  }
  
  .projects-container::-webkit-scrollbar-thumb {
    background-color: rgba(0, 0, 0, 0.2);
    border-radius: 10px;
  }
  
  /* On mobile screens, adjust the container to allow space for image and bubble */
  @media (max-width: 768px) {
    .projects-container {
      /* Set bottom to allow space for image and bubble */
      bottom: 120px; /* Reduced from 200px to show more of the project area */
      padding: 10px; /* Add padding to prevent shadow clipping */
    }
  }
  
  .projects-grid {
    display: grid;
    grid-template-columns: minmax(0, 1fr);
    gap: 20px;
    width: 100%;
    padding: 12px; /* Reduced from 16px */
  }
  
  /* Reduce padding on mobile screens */
  @media (max-width: 768px) {
    .projects-grid {
      padding: 6px; /* Reduced from 8px */
      gap: 10px; /* Reduced from 12px */
    }
  }
  
  /* Media query for 2-column layout with increasing gaps for larger screens */
  @media (min-width: 1000px) {
    .projects-grid {
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: clamp(24px, 4vw, 40px);
    }
  }
  
  @media (min-width: 1400px) {
    .projects-grid {
      gap: clamp(40px, 6vw, 80px);
    }
  }
  
  @media (min-width: 1800px) {
    .projects-grid {
      gap: clamp(80px, 10vw, 180px);
    }
  }
  
  @media (min-width: 2200px) {
    .projects-grid {
      gap: clamp(120px, 14vw, 250px);
    }
  }
  
  .dictionary-card {
    background: rgba(255, 255, 255, 0.95);
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 8px 8px 0 rgba(0, 0, 0, 0.25), 0 0 0 2px rgba(0, 0, 0, 0.1);
    padding: 0;
    position: relative;
    z-index: 1;
    display: flex;
    width: 100%;
    min-width: 0;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }
  
  .dictionary-card:hover {
    transform: translateY(-2px) translateX(-2px);
    box-shadow: 10px 10px 0 rgba(0, 0, 0, 0.2), 0 0 0 2px rgba(0, 0, 0, 0.1);
  }
  
  .content-area {
    flex: 1;
    padding: 20px; /* Reduced from 24px */
  }
  
  .logo-area {
    width: 256px;
    background-color: rgba(0, 0, 0, 0.03);
    display: none; /* Hide by default on small screens */
    align-items: center;
    justify-content: center;
    padding: 0px;
    border-left: 1px solid rgba(0, 0, 0, 0.08);
  }
  
  /* Show logos on medium screens and above */
  @media (min-width: 768px) {
    .logo-area {
      display: flex;
    }
  }
  
  .project-logo {
    width: 100%;
    height: auto;
    object-fit: contain;
  }
  
  .term-container {
    display: flex;
    flex-direction: column;
    gap: 4px;
    margin-bottom: 12px;
  }
  
  .term {
    font-family: 'Jost', sans-serif;
    font-weight: 600;
    font-size: 28px;
    margin: 0;
    color: #111;
  }
  
  .pronunciation-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
  }
  
  .pronunciation {
    font-family: 'Jost', sans-serif;
    font-weight: 400;
    font-size: 14px;
    color: #666;
    font-style: italic;
  }
  
  .part-of-speech {
    font-family: 'Jost', sans-serif;
    font-style: italic;
    font-size: 14px;
    color: #555;
    text-align: right;
  }
  
  .definition-container {
    padding-left: 5px;
    margin-bottom: 16px; /* Reduced from 18px */
  }
  
  .definition {
    display: flex;
    align-items: baseline;
    margin-bottom: 10px;
  }
  
  .definition-text {
    font-family: 'Jost', sans-serif;
    font-size: 16px;
    line-height: 1.5;
    color: #333;
  }
  
  .usage {
    font-family: 'Jost', sans-serif;
    font-size: 14px;
    margin-top: 10px;
    color: #555;
    display: flex;
    align-items: baseline;
    gap: 8px;
  }
  
  .usage-label {
    font-weight: 600;
  }
  
  .usage-example {
    color: var(--accent-color, #6E56CF);
    text-decoration: none;
    font-weight: 500;
    transition: opacity 0.2s;
  }
  
  .usage-example:hover {
    opacity: 0.8;
    text-decoration: underline;
  }
  
  /* Comic bubble styles */
  .comic-bubble {
    position: absolute;
    top: 55%; /* Adjusted to align with mouth */
    left: 100%; /* Position at the right edge of the container */
    transform: translateY(-50%) rotate(2deg); /* Center vertically and add slight rotation */
    min-width: 220px; /* Reduced from 280px */
    background-color: white;
    border: 5px solid black;
    border-radius: 40px;
    padding: 15px 12px; /* Reduced horizontal padding from 20px to 12px */
    display: flex;
    flex-direction: column;
    align-items: center;
    box-shadow: 6px 6px 0 rgba(0, 0, 0, 0.8);
    z-index: 3; /* Lower than the max z-index of projects */
    margin-left: 5px; /* Small gap between image and bubble */
    animation: comic-pop 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.5) forwards;
  }
  
  .comic-text {
    font-family: 'Fuzzy Bubbles', cursive;
    font-size: 24px; /* Reduced from 30px */
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
    font-size: 28px; /* Reduced from 34px */
    transform: skew(3deg);
  }
  
  .comic-text:nth-child(3) {
    font-size: 26px; /* Reduced from 32px */
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
  
  .copyright {
    position: fixed;
    bottom: 12px;
    right: 16px;
    font-family: 'Jost', sans-serif;
    color: rgba(0, 0, 0, 0.8);
    font-size: 12px;
    text-align: right;
    z-index: 10;
    font-weight: 600;
    letter-spacing: 1px;
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
  
  .keywords-container {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    margin: 10px 0;
  }
  
  .keyword {
    font-family: 'Jost', sans-serif;
    font-size: 10px;
    font-weight: 500;
    padding: 3px 8px;
    border-radius: 50px;
    display: inline-block;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }
  
  .welcome-text {
    font-family: 'Jost', sans-serif;
    font-size: 32px;
    font-weight: 600;
    color: rgba(0, 0, 0, 0.8);
    margin-bottom: 32px;
    text-align: center;
  }
  
  .fallback-logo, .viveros-logo {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }
  
  .viveros-logo {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 220px;
  }
  
  /* Adjust bubble position on very small screens */
  @media (max-width: 450px) {
    .comic-bubble {
      top: 40%; /* Move it higher up */
      min-width: 180px; /* Make it smaller */
      padding: 12px 10px; /* Reduce padding */
    }
    
    .comic-text {
      font-size: 20px; /* Smaller text */
    }
    
    .comic-text:nth-child(2) {
      font-size: 24px;
    }
    
    .comic-text:nth-child(3) {
      font-size: 22px;
    }
  }
  
  /* On small screens, increase z-index for bubble */
  @media (max-width: 768px) {
    .comic-bubble {
      z-index: 5; /* Higher z-index on small screens */
    }
  }
  
  /* Mobile swipe view styling */
  .mobile-project-swipe {
    width: 100%;
    padding: 16px; /* Increased from 8px to show shadows */
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    min-height: auto;
    position: relative;
    gap: 12px; /* Add gap between swipe indicator, card, and navigation */
  }
  
  /* Card container for transitions */
  .card-container {
    width: 100%;
    position: relative;
    min-height: 500px;
    display: flex;
    justify-content: center;
    overflow: visible; /* Changed from hidden to show shadows */
    border-radius: 8px;
    will-change: contents;
    transform: translateZ(0);
    box-sizing: border-box;
    padding: 8px; /* Added padding to show shadows */
  }
  
  /* Swipe direction indicators */
  .swiping-left .card-container::after,
  .swiping-right .card-container::before {
    content: '';
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    width: 30px;
    height: 30px;
    background-color: rgba(0, 0, 0, 0.1);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 10;
    animation: indicatorPulse 0.5s infinite alternate;
  }
  
  .swiping-left .card-container::after {
    right: -10px;
    clip-path: polygon(40% 0%, 100% 50%, 40% 100%, 40% 75%, 70% 50%, 40% 25%);
  }
  
  .swiping-right .card-container::before {
    left: -10px;
    clip-path: polygon(60% 0%, 60% 25%, 30% 50%, 60% 75%, 60% 100%, 0% 50%);
  }
  
  @keyframes indicatorPulse {
    from { opacity: 0.7; }
    to { opacity: 0.3; }
  }
  
  /* Mobile view card styling */
  .mobile-project-swipe .dictionary-card {
    width: 100%;
    position: absolute; /* Back to absolute positioning for smooth transitions */
    left: 0;
    top: 0;
    animation-fill-mode: forwards;
    transform-origin: center;
    display: flex;
    flex-direction: column;
    box-shadow: 8px 8px 0 rgba(0, 0, 0, 0.25), 0 0 0 2px rgba(0, 0, 0, 0.1);
    will-change: transform;
    min-width: 0;
    box-sizing: border-box;
  }
  
  .swipe-indicator {
    text-align: center;
    margin-bottom: 4px; /* Reduced from 8px */
    width: 100%;
  }
  
  .indicator-dots {
    display: flex;
    justify-content: center;
    gap: 6px; /* Reduced from 8px */
    margin-bottom: 4px; /* Reduced from 8px */
  }
  
  .dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background-color: rgba(0, 0, 0, 0.2);
    transition: background-color 0.3s, transform 0.3s;
  }
  
  .dot.active {
    background-color: rgba(0, 0, 0, 0.6);
    transform: scale(1.3);
  }
  
  .swipe-hint {
    font-size: 12px; /* Increased from 11px */
    color: rgba(0, 0, 0, 0.7); /* Increased contrast from 0.5 to 0.7 */
    margin-bottom: 4px;
    font-weight: 500; /* Added font weight for better visibility */
    animation: pulse 3s infinite;
  }
  
  @keyframes pulse {
    0%, 100% { opacity: 0.9; } /* Increased from 0.8 */
    50% { opacity: 0.7; } /* Increased from 0.5 */
  }
  
  /* Mobile navigation - position directly under the card */
  .mobile-navigation {
    display: flex;
    justify-content: center;
    gap: 20px; /* Reduced from 40px to bring arrows closer together */
    width: 100%;
    margin-top: 0;
    margin-bottom: 12px;
    padding: 0;
    position: relative;
    z-index: 5;
  }
  
  .nav-button {
    background-color: transparent;
    border: none;
    border-radius: 50%;
    width: 52px;
    height: 52px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 28px; /* Increased from 24px for better visibility */
    color: rgba(0, 0, 0, 0.8); /* Darker color for better contrast */
    cursor: pointer;
    -webkit-tap-highlight-color: transparent;
    box-shadow: none;
    transition: none;
  }
  
  .nav-button:hover {
    background-color: transparent;
    transform: none;
  }
  
  .nav-button:active {
    transform: none;
    background-color: transparent;
    box-shadow: none;
  }
  
  /* Mobile card layout - vertical for small screens */
  .mobile-card-content {
    display: flex;
    flex-direction: column;
    width: 100%;
    min-width: 0; /* Allow content to shrink if needed */
    overflow: hidden; /* Prevent content overflow */
  }
  
  .mobile-card-content .content-area {
    padding: 16px 16px 14px; /* Slightly reduced bottom padding */
  }
  
  .mobile-logo-area {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: rgba(0, 0, 0, 0.03);
    padding: 10px;
    padding-bottom: 8px;
    border-top: 1px solid rgba(0, 0, 0, 0.08);
    border-bottom-left-radius: 8px;
    border-bottom-right-radius: 8px;
    box-sizing: border-box;
    min-height: 240px; /* Increased from 180px */
  }
  
  .mobile-logo-area .logo-container {
    width: 100%;
    max-width: 220px;
    height: 220px; /* Increased from 160px */
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .mobile-logo-area .project-logo {
    max-width: 100%;
    max-height: 220px; /* Increased from 160px */
    object-fit: contain;
  }
  
  .mobile-project-swipe .viveros-logo {
    min-height: 220px; /* Increased from 160px */
  }
</style>
