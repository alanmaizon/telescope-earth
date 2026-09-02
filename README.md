# Telescope

Telescope is a static, browser-based visualization about scale. The landing page presents a cinematic journey from Earth to the observable universe. The final frame links to an interactive Three.js solar-system model where visitors can orbit the camera, zoom, and select planets for facts, imagery, and narration.

## Demo

- Intro: `index.html`
- Interactive model: `solar-system.html`
- GitHub Pages: `https://alanmaizon.github.io/telescope-earth/`

## Run locally

No build tool or package installation is required. Serve the repository over HTTP so browser assets and audio load consistently:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000/>. Opening the HTML files directly can prevent some browser APIs and media assets from working as expected.

## Technical overview

The project is intentionally framework-free:

- `index.html` contains the hyperframe experience. Its scene timeline creates and animates DOM elements with small easing helpers.
- `solar-system.html` is the interactive entrypoint and owns the page structure, navigation, and accessible planet-details panel.
- `static/js/script.js` builds the Three.js scene, loads local textures, groups planets for rotation, raycasts pointer selections, and updates the detail panel.
- `static/css/style.css` provides the full-screen canvas shell, responsive header, and details panel styling.
- `.github/workflows/deploy-pages.yml` uploads the repository as a Pages artifact and deploys it whenever `main` changes.

The model is a visual scale study, not a physically accurate simulation. Planet sizes and distances are tuned for legibility in one viewport; rotation values provide motion and the camera controls provide exploration.

## Interaction flow

1. `OrbitControls` handles camera orbiting, damping, and zoom.
2. A pointer event converts screen coordinates into normalized device coordinates.
3. `THREE.Raycaster` tests those coordinates against the planet meshes.
4. The matching data object populates the sidebar and starts its optional narration.
5. The animation loop rotates planet groups, updates selected labels, and renders the frame.

## Representative code

### Creating a textured planet

```js
const createPlanet = (size, texturePath, distance) => {
    const geometry = new THREE.SphereGeometry(size, 32, 32);
    const texture = textureLoader.load(texturePath);
    const material = new THREE.MeshBasicMaterial({ map: texture });
    const planet = new THREE.Mesh(geometry, material);
    planet.position.x = distance;
    return planet;
};
```

### Selecting a planet with raycasting

```js
raycaster.setFromCamera(mouse, camera);
const intersects = raycaster.intersectObjects(planets.map(planet => planet.object));

if (intersects.length > 0) {
    selectedPlanet = planets.find(planet => planet.object === intersects[0].object);
    displayPlanetInfo(selectedPlanet);
}
```

### Keeping the renderer responsive

```js
const resize = () => {
    const width = window.innerWidth;
    const height = window.innerHeight;

    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.setSize(width, height);
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
};

window.addEventListener('resize', resize);
```

## Interview talking points

- Explain the separation between the DOM-based narrative and the WebGL exploration page.
- Call out the data-driven `planets` array: geometry, media, audio, and facts are connected through one object model.
- Discuss the deliberate compromise between scientific scale and visual communication.
- Mention static deployment: there is no runtime server, build pipeline, or framework dependency.
- Suggested next steps: move planet data to JSON, add keyboard focus selection, lazy-load media, and use physically based materials with lighting for richer surfaces.

## Deployment

The workflow in `.github/workflows/deploy-pages.yml` runs on pushes to `main` and can also be started manually from the Actions tab. In the repository settings, set **Pages → Build and deployment → Source** to **GitHub Actions**.
