# icke.berlin

A personal website built with SvelteKit and configured for GitHub Pages deployment.

## Development

Once you've cloned the project and installed dependencies with `npm install`, start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

## Deployment

This project is configured to automatically deploy to GitHub Pages when changes are pushed to the main branch. The GitHub Actions workflow in `.github/workflows/deploy.yml` handles the build and deployment process.

### Manual Deployment

If you want to deploy manually, you can:

1. Build the site locally: `npm run build`
2. The output will be in the `build` directory 
3. Push the changes to GitHub, and the GitHub Actions workflow will deploy the site

### GitHub Pages Setup

To set up GitHub Pages for this repository:

1. Go to the repository settings on GitHub
2. Navigate to "Pages"
3. Select "GitHub Actions" as the source
4. The site will be available at `https://[username].github.io/icke.berlin/`

## Customization

- Edit `src/routes/+page.svelte` to modify the main page
- Add additional pages in the `src/routes` directory
- Add static assets to the `static` directory
