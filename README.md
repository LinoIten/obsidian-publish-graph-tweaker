# Obsidian Publish Graph Tweaker

A script for customizing the graph visualization in Obsidian Publish sites by directly manipulating the canvas renderer state. This provides customization capabilities that aren't normally available in the Publish version of Obsidian's graph view.

## Prerequisites

1. An Obsidian Publish site
2. Custom domain setup ([See Obsidian's guide on custom domain setup](https://help.obsidian.md/Obsidian+Publish/Custom+domains))

## Installation

1. Create a file named `publish.js` in your vault's root directory
2. Copy the following code into your `publish.js`:

```javascript
/**
 * Obsidian Publish Graph Tweaker
 * Customizes the graph view by manipulating the canvas renderer state
 */

// Configuration - Customize these values
const config = {
    // Base colors for graph elements
    baseColors: {
        line: { a: 0.6, rgb: 0x5ab873 },
        fill: { a: 1, rgb: 0x6bd385 },
        fillUnresolved: { a: 0.4, rgb: 0x6bd385 },
        text: { a: 1, rgb: 0xF4FFBF },
        fillHighlight: { a: 1, rgb: 0x5ab873 },
        lineHighlight: { a: 0.8, rgb: 0x5ab873 },
        circle: { a: 1, rgb: 0x6bd385 },
        fillFocused: { a: 1, rgb: 0x00FF00 }
    },
    // Color patterns for different types of nodes
    nodeColors: [
        { regex: /^Item\//, color: { a: 1, rgb: 0x00FF00 } },
        { regex: /^Spell\//, color: { a: 1, rgb: 0xFF8C00 } },
        { regex: /^World Boss/, color: { a: 1, rgb: 0xFF0040 } },
        { regex: /^Monster/, color: { a: 1, rgb: 0x6200FF } },
        { regex: /^NPC/, color: { a: 1, rgb: 0x8E00FF } },
        { regex: /^Locations\//, color: { a: 1, rgb: 0xFFDA00 } },
        { regex: /^Ore/, color: { a: 1, rgb: 0x00FFD2 } }
    ]
};

// State tracking
let lastProcessedState = '';
let persistenceInterval;

// Main update function
const applyGraphChanges = (force = false) => {
    if (!app?.graph?.renderer) return;

    // Track state changes
    const currentState = JSON.stringify({
        isExpanded: document.querySelector('.expanded-graph') !== null,
        navigationPath: window.location.pathname,
        hasHanger: !!app.graph.renderer.hanger,
        hasNodeLookup: !!app.graph.renderer.nodeLookup
    });

    if (!force && currentState === lastProcessedState) return;
    lastProcessedState = currentState;

    // Clear existing interval
    if (persistenceInterval) {
        clearInterval(persistenceInterval);
    }

    const applySettings = () => {
        app.graph.renderer.hidePowerTag = true;

        // Apply base colors
        if (app.graph.renderer.colors) {
            Object.assign(app.graph.renderer.colors, config.baseColors);
        }

        // Apply node colors
        if (app.graph.renderer.nodeLookup) {
            const lookup = app.graph.renderer.nodeLookup;
            const keys = Object.keys(lookup);
            
            for (let i = 0; i < keys.length; i++) {
                const key = keys[i];
                const node = lookup[key];
                
                for (const { regex, color } of config.nodeColors) {
                    if (regex.test(key)) {
                        const currentColor = node.color;
                        if (!currentColor || currentColor.a !== color.a || currentColor.rgb !== color.rgb) {
                            node.color = color;
                        }
                        break;
                    }
                }
            }
        }

        app.graph.renderer.renderCallback();
    };

    // Apply immediately and persist
    applySettings();
    persistenceInterval = setInterval(applySettings, 300);

    // Stop persistence after 1.5 seconds
    setTimeout(() => {
        if (persistenceInterval) {
            clearInterval(persistenceInterval);
            persistenceInterval = null;
        }
    }, 1500);
};

// Set up observers and event listeners
const observer = new MutationObserver((mutations) => {
    clearTimeout(debounceTimeout);
    debounceTimeout = setTimeout(() => applyGraphChanges(), 50);
});

observer.observe(document.body, {
    childList: true,
    subtree: true,
    attributes: true,
    characterData: true
});

window.addEventListener('popstate', () => applyGraphChanges(true));
document.addEventListener('click', () => {
    setTimeout(() => applyGraphChanges(true), 100);
});
```

3. Publish the `publish.js` file through Obsidian Publish settings

## Understanding and Customizing the Graph Renderer

The script works by manipulating `app.graph.renderer`, which is the core object controlling the graph's appearance and behavior. You can explore its capabilities by:

1. Opening your browser's developer console
2. Inspecting `app.graph.renderer` to discover available properties and methods
3. Using this knowledge to extend the script with new customizations

However, remember that any direct modifications to the renderer will be reset by Obsidian Publish. This is why the script uses an observer pattern and periodic reapplication - to ensure your customizations persist.

### Node Color Patterns

The `nodeColors` patterns work by matching against the full path of notes in your vault. For example:
- A note at `Item/Sword.md` would match the pattern `/^Item\//`
- A note at `Spells/Fire/Fireball.md` would match the pattern `/^Spells\//`

Currently, the coloring system relies on folder structures, so you'll need to organize your notes into folders to take advantage of the coloring patterns.

### Available Customizations

The script provides several ways to customize the graph:

1. **Base Colors**: Modify the `baseColors` object to change the default appearance of graph elements
2. **Node Colors**: Add or modify patterns in the `nodeColors` array to color specific types of nodes
3. **Renderer Properties**: Explore and modify other renderer properties you discover through console inspection

For example, you might find properties controlling:
- Node sizes and shapes
- Line thicknesses
- Animation behaviors
- Layout algorithms

Just remember to add any new customizations to the `applySettings` function to ensure they persist.

## Troubleshooting

- Script not loading: Verify custom domain setup and that `publish.js` is published
- Styles resetting: This is normal - the script automatically reapplies styles
- Performance issues: Remove unused node color patterns or increase debounce timeout

## Contributing

Found a way to improve the graph customization? Feel free to submit issues and pull requests!

## License

MIT License - See LICENSE file for details