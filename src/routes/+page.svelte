<script>
  let title = "icke.berlin";
  let description = "Personal website";
</script>

<svelte:head>
  <title>{title}</title>
  <meta name="description" content={description} />
</svelte:head>

<main>
  <section class="hero">
    <h1>{title}</h1>
    <p class="subtitle">Welcome to my personal website</p>
  </section>

  <section class="content">
    <h2>About</h2>
    <p>
      This is a static site built with SvelteKit and hosted on GitHub Pages.
    </p>
    
    <h2>Contact</h2>
    <p>
      You can reach me at <a href="mailto:contact@example.com">contact@example.com</a>
    </p>
  </section>
</main>

<style>
  main {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  .hero {
    text-align: center;
    margin: 4rem 0;
  }

  h1 {
    font-size: 3rem;
    margin-bottom: 0.5rem;
  }

  .subtitle {
    font-size: 1.5rem;
    color: #555;
  }

  .content {
    max-width: 800px;
    margin: 0 auto;
  }

  h2 {
    margin-top: 2rem;
    margin-bottom: 1rem;
  }

  a {
    color: #0066cc;
    text-decoration: none;
  }

  a:hover {
    text-decoration: underline;
  }
</style>
