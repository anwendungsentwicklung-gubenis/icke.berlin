<script>
  import { onMount, onDestroy } from 'svelte';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
  
  // Props
  export let text = 'V';
  export let color = '#2E7D32';
  
  let containerEl;
  let mounted = false;
  
  // Settings
  const fontName = 'Jost';
  const textureFontSize = 140;
  const fontScaleFactor = 0.085;
  
  // 3D scene related globals
  let scene, camera, renderer, textCanvas, textCtx, particleGeometry, dummy, clock;
  let flowerInstancedMesh, leafInstancedMesh, flowerMaterial, leafMaterial;
  
  // Coordinates data per 2D canvas and 3D scene
  let textureCoordinates = [];
  
  // 1d-array of data objects to store and change params of each instance
  let particles = [];
  
  // Parameters of whole string per 2D canvas and 3D scene
  let stringBox = {
    wTexture: 0,
    wScene: 0,
    hTexture: 0,
    hScene: 0
  };
  
  // Animation frame ID for cleanup
  let animationFrameId;
  
  // Particle constructors from index.js
  function Flower([x, y]) {
    this.type = 0;
    this.x = x + .2 * (Math.random() - .5);
    this.y = y + .2 * (Math.random() - .5);
    this.z = 0;

    this.color = Math.random() * 60;

    this.isGrowing = true;
    this.toDelete = false;

    this.scale = 0;
    this.maxScale = .9 * Math.pow(Math.random(), 20);
    this.deltaScale = .03 + .1 * Math.random();
    this.age = Math.PI * Math.random();
    this.ageDelta = .01 + .02 * Math.random();
    this.rotationZ = .5 * Math.random() * Math.PI;

    this.grow = function () {
      this.age += this.ageDelta;
      if (this.isGrowing) {
        this.deltaScale *= .99;
        this.scale += this.deltaScale;
        if (this.scale >= this.maxScale) {
          this.isGrowing = false;
        }
      } else if (this.toDelete) {
        this.deltaScale *= 1.1;
        this.scale -= this.deltaScale;
        if (this.scale <= 0) {
          this.scale = 0;
          this.deltaScale = 0;
        }
      } else {
        this.scale = this.maxScale + .2 * Math.sin(this.age);
        this.rotationZ += .001 * Math.cos(this.age);
      }
    }
  }

  function Leaf([x, y]) {
    this.type = 1;
    this.x = x;
    this.y = y;
    this.z = 0;

    this.rotationZ = .6 * (Math.random() - .5) * Math.PI;

    this.color = 100 + Math.random() * 50;

    this.isGrowing = true;
    this.toDelete = false;

    this.scale = 0;
    this.maxScale = .1 + .7 * Math.pow(Math.random(), 7);
    this.deltaScale = .03 + .03 * Math.random();
    this.age = Math.PI * Math.random();

    this.grow = function () {
      if (this.isGrowing) {
        this.deltaScale *= .99;
        this.scale += this.deltaScale;
        if (this.scale >= this.maxScale) {
          this.isGrowing = false;
        }
      }
      if (this.toDelete) {
        this.deltaScale *= 1.1;
        this.scale -= this.deltaScale;
        if (this.scale <= 0) {
          this.scale = 0;
        }
      }
    }
  }
  
  // Initialize the scene
  function init() {
    if (!containerEl || !mounted) return;
    
    console.log("FlowerLogo: Initializing scene");
    console.log("FlowerLogo: Container size:", containerEl.clientWidth, containerEl.clientHeight);
    
    // Fix for zero width container
    const containerWidth = containerEl.clientWidth || 200; 
    const containerHeight = containerEl.clientHeight || 200;
    
    console.log("FlowerLogo: Using adjusted container size:", containerWidth, containerHeight);
    
    // Create a hidden canvas to render the text
    textCanvas = document.createElement('canvas');
    textCanvas.width = 1024;
    textCanvas.height = 1024;
    textCtx = textCanvas.getContext('2d');
    
    if (!textCtx) {
      console.error("FlowerLogo: Failed to get 2D context for text canvas");
      return;
    }
    
    // Scene setup
    scene = new THREE.Scene();
    
    // Camera setup
    const fov = 40;
    const aspect = containerWidth / containerHeight;
    const near = 0.1;
    const far = 1000;
    camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
    camera.position.z = 4.5;
    
    // Renderer setup
    try {
      renderer = new THREE.WebGLRenderer({ 
        antialias: true, 
        alpha: true
      });
      console.log("FlowerLogo: WebGL renderer created successfully");
    } catch (e) {
      console.error("FlowerLogo: Failed to create WebGL renderer:", e);
      return;
    }
    
    renderer.setSize(containerWidth, containerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.setClearColor(0x000000, 0); // Clear to transparent
    containerEl.appendChild(renderer.domElement);
    
    // Create plane geometry for the particles (matching index.js)
    particleGeometry = new THREE.PlaneGeometry(1.2, 1.2);
    
    // Load textures from URLs as in index.js
    const textureLoader = new THREE.TextureLoader();
    
    // For local development, we'll define fallback images if the URLs fail
    const flowerTexture = textureLoader.load('https://ksenia-k.com/img/threejs/flower.png', 
      undefined, 
      undefined, 
      (err) => {
        console.error("FlowerLogo: Failed to load flower texture:", err);
        
        // Create a fallback texture if loading fails
        const canvas = document.createElement('canvas');
        canvas.width = canvas.height = 64;
        const ctx = canvas.getContext('2d');
        
        // Draw a simple flower shape
        ctx.fillStyle = 'white';
        ctx.beginPath();
        ctx.arc(32, 32, 10, 0, Math.PI * 2);
        ctx.fill();
        
        for (let i = 0; i < 5; i++) {
          const angle = (i / 5) * Math.PI * 2;
          ctx.beginPath();
          ctx.ellipse(
            32 + Math.cos(angle) * 20, 
            32 + Math.sin(angle) * 20, 
            10, 15, 
            angle, 0, Math.PI * 2
          );
          ctx.fill();
        }
        
        const fallbackTexture = new THREE.CanvasTexture(canvas);
        flowerMaterial.alphaMap = fallbackTexture;
        flowerMaterial.needsUpdate = true;
      }
    );
    
    const leafTexture = textureLoader.load('https://ksenia-k.com/img/threejs/leaf.png',
      undefined, 
      undefined, 
      (err) => {
        console.error("FlowerLogo: Failed to load leaf texture:", err);
        
        // Create a fallback texture if loading fails
        const canvas = document.createElement('canvas');
        canvas.width = canvas.height = 64;
        const ctx = canvas.getContext('2d');
        
        // Draw a simple leaf shape
        ctx.fillStyle = 'white';
        ctx.beginPath();
        ctx.ellipse(32, 40, 10, 20, 0, 0, Math.PI * 2);
        ctx.fill();
        
        const fallbackTexture = new THREE.CanvasTexture(canvas);
        leafMaterial.alphaMap = fallbackTexture;
        leafMaterial.needsUpdate = true;
      }
    );
    
    // Materials with textures (matching index.js approach)
    flowerMaterial = new THREE.MeshBasicMaterial({
      alphaMap: flowerTexture,
      opacity: 0.3,
      depthTest: false,
      transparent: true,
    });
    
    leafMaterial = new THREE.MeshBasicMaterial({
      alphaMap: leafTexture,
      opacity: 0.35,
      depthTest: false,
      transparent: true,
    });
    
    console.log("FlowerLogo: Materials created with textures");
    
    // Initialize dummy for matrix transformations
    dummy = new THREE.Object3D();
    
    // Initialize clock for animations
    clock = new THREE.Clock();
    
    // Setup controls
    try {
      const controls = new OrbitControls(camera, renderer.domElement);
      controls.enablePan = false;
      controls.enableDamping = true;
      controls.dampingFactor = 0.05;
      controls.rotateSpeed = 0.5;
      console.log("FlowerLogo: OrbitControls initialized");
    } catch (e) {
      console.error("FlowerLogo: Error initializing OrbitControls:", e);
    }
    
    // Sample text coordinates and create particles
    sampleCoordinates();
    
    // Add meshes to scene
    recreateInstancedMesh();
    
    // Start animation
    animate();
    
    // Handle window resize
    window.addEventListener('resize', onWindowResize);
    
    console.log("FlowerLogo: Initialization complete");
  }
  
  function sampleCoordinates() {
    console.log("FlowerLogo: Sampling coordinates for text:", text);
    
    // Draw text
    textCanvas.width = 1024;
    textCanvas.height = 1024;
    textCtx.font = `600 ${textureFontSize}px ${fontName}`;
    textCtx.fillStyle = 'white';
    textCtx.clearRect(0, 0, textCanvas.width, textCanvas.height);
    textCtx.fillStyle = '#2a9d8f';
    
    // Calculate centering position
    const metrics = textCtx.measureText(text);
    const textWidth = metrics.width;
    // Calculate text height better (approx 70% of font size)
    const textHeight = textureFontSize * 0.7;
    const centerX = (textCanvas.width - textWidth) / 2;
    const centerY = textCanvas.height / 2 + textHeight/3; // Add offset to center vertically
    
    // Draw text centered
    textCtx.fillText(text, centerX, centerY);
    
    // Sample coordinates if canvas has valid size
    stringBox.wTexture = textWidth;
    stringBox.hTexture = textureFontSize;
    stringBox.wScene = stringBox.wTexture * fontScaleFactor;
    stringBox.hScene = stringBox.hTexture * fontScaleFactor;
    
    // Image data to 2d array
    const imageData = textCtx.getImageData(0, 0, textCanvas.width, textCanvas.height);
    
    // Reset particles and coordinates
    textureCoordinates = [];
    particles = [];
    
    // Sample coordinates - using a step to reduce density
    const step = 6;
    for (let y = 0; y < textCanvas.height; y += step) {
      for (let x = 0; x < textCanvas.width; x += step) {
        const index = (y * textCanvas.width + x) * 4;
        const alpha = imageData.data[index + 0] + imageData.data[index + 1] + imageData.data[index + 2];
        
        // Only create particles for non-transparent pixels
        if (alpha > 30) {
          // Convert to scene coordinates
          const tx = x / textCanvas.width;
          const ty = y / textCanvas.height;
          const sx = (tx - 0.5) * stringBox.wScene * 2;
          const sy = (0.5 - ty) * stringBox.hScene * 2;
          
          // Store coordinates
          textureCoordinates.push({x: sx, y: sy});
          
          // Create particle (Flower or Leaf)
          const p = Math.random() > 0.2 ? new Flower([sx, sy]) : new Leaf([sx, sy]);
          particles.push(p);
          
          // Limit particles to avoid performance issues
          if (particles.length >= 500) break;
        }
      }
      if (particles.length >= 500) break;
    }
    
    console.log(`FlowerLogo: Created ${particles.length} particles`);
  }
  
  function recreateInstancedMesh() {
    // Remove existing meshes
    if (flowerInstancedMesh) scene.remove(flowerInstancedMesh);
    if (leafInstancedMesh) scene.remove(leafInstancedMesh);
    
    // Count particles by type
    const totalNumberOfFlowers = particles.filter(v => v.type === 0).length;
    const totalNumberOfLeafs = particles.filter(v => v.type === 1).length;
    
    console.log("FlowerLogo: Creating instanced meshes with", totalNumberOfFlowers, "flowers and", totalNumberOfLeafs, "leaves");
    
    // Create new instanced meshes
    flowerInstancedMesh = new THREE.InstancedMesh(particleGeometry, flowerMaterial, Math.max(1, totalNumberOfFlowers));
    leafInstancedMesh = new THREE.InstancedMesh(particleGeometry, leafMaterial, Math.max(1, totalNumberOfLeafs));
    
    // Set colors for each instance
    let flowerIdx = 0;
    let leafIdx = 0;
    particles.forEach(p => {
      if (p.type === 0) {
        flowerInstancedMesh.setColorAt(flowerIdx, new THREE.Color("hsl(" + p.color + ", 100%, 50%)"));
        flowerIdx++;
      } else {
        leafInstancedMesh.setColorAt(leafIdx, new THREE.Color("hsl(" + p.color + ", 100%, 20%)"));
        leafIdx++;
      }
    });
    
    // Center the meshes in the scene
    leafInstancedMesh.position.x = flowerInstancedMesh.position.x = 0;
    leafInstancedMesh.position.y = flowerInstancedMesh.position.y = 0;
    
    // Add to scene
    scene.add(flowerInstancedMesh, leafInstancedMesh);
  }
  
  function onWindowResize() {
    if (!containerEl || !camera || !renderer) return;
    
    const containerWidth = containerEl.clientWidth || 200;
    const containerHeight = containerEl.clientHeight || 200;
    
    camera.aspect = containerWidth / containerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(containerWidth, containerHeight);
  }
  
  function animate() {
    if (!mounted) {
      console.log("FlowerLogo: Animation stopped - component not mounted");
      return;
    }
    
    animationFrameId = requestAnimationFrame(animate);
    
    if (!particles.length || !flowerInstancedMesh || !leafInstancedMesh || !dummy) {
      console.warn("FlowerLogo: Animation - missing required objects");
      return;
    }
    
    // Update particles
    updateParticlesMatrices();
    
    // Render scene
    renderer.render(scene, camera);
  }
  
  function updateParticlesMatrices() {
    let flowerIdx = 0;
    let leafIdx = 0;
    
    particles.forEach(p => {
      // Grow the particle according to its rules
      p.grow();
      
      // Match rotation to camera (billboarding)
      dummy.quaternion.copy(camera.quaternion);
      
      // Apply particle properties
      dummy.rotation.z += p.rotationZ;
      dummy.scale.set(p.scale, p.scale, p.scale);
      dummy.position.set(p.x, p.y, p.z);
      
      // Add offset for leaves
      if (p.type === 1) {
        dummy.position.y += 0.5 * p.scale;
      }
      
      // Update the matrix
      dummy.updateMatrix();
      
      // Apply to the correct instanced mesh
      if (p.type === 0 && flowerIdx < flowerInstancedMesh.count) {
        flowerInstancedMesh.setMatrixAt(flowerIdx, dummy.matrix);
        flowerIdx++;
      } else if (p.type === 1 && leafIdx < leafInstancedMesh.count) {
        leafInstancedMesh.setMatrixAt(leafIdx, dummy.matrix);
        leafIdx++;
      }
    });
    
    // Update matrices in GPU
    flowerInstancedMesh.instanceMatrix.needsUpdate = true;
    leafInstancedMesh.instanceMatrix.needsUpdate = true;
  }
  
  onMount(() => {
    if (typeof window !== 'undefined') {
      console.log("FlowerLogo: Component mounted");
      mounted = true;
      // Wait a bit for the container to be properly sized
      setTimeout(() => {
        console.log("FlowerLogo: Starting initialization");
        init();
      }, 100);
    } else {
      console.log("FlowerLogo: Not in browser environment, skipping initialization");
    }
  });
  
  onDestroy(() => {
    console.log("FlowerLogo: Component being destroyed");
    mounted = false;
    
    if (typeof window === 'undefined') return;
    
    if (animationFrameId) {
      cancelAnimationFrame(animationFrameId);
      console.log("FlowerLogo: Animation frame canceled");
    }
    
    if (renderer && containerEl) {
      try {
        containerEl.removeChild(renderer.domElement);
      } catch (e) {
        console.error("Could not remove renderer:", e);
      }
    }
    
    if (typeof window !== 'undefined') {
      window.removeEventListener('resize', onWindowResize);
    }
    
    // Dispose resources
    if (flowerInstancedMesh) {
      flowerInstancedMesh.geometry.dispose();
      flowerInstancedMesh.material.dispose();
    }
    
    if (leafInstancedMesh) {
      leafInstancedMesh.geometry.dispose();
      leafInstancedMesh.material.dispose();
    }
  });
</script>

<div class="flower-logo-container" bind:this={containerEl} style="width: 100%; height: 100%;">
  <!-- Canvas will be added here by Three.js -->
</div>

<style>
  .flower-logo-container {
    width: 100%;
    height: 100%;
    min-width: 200px;
    min-height: 220px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: transparent;
    position: relative;
  }
  
  :global(.flower-logo-container canvas) {
    position: absolute;
    top: 0;
    left: 0;
    width: 100% !important;
    height: 100% !important;
    z-index: 1;
  }
</style> 