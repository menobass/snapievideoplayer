# Embedding Guide for 3speak Video Player

This guide shows how to embed videos from play.3speak.tv into your website or application.

## Quick Start

### Standard Embed (Recommended)

For embedding in blog posts, articles, or any webpage:

```html
<iframe 
  src="https://play.3speak.tv/watch?v=meno/1czchhmr&mode=iframe" 
  width="854" 
  height="480"
  frameborder="0"
  allowfullscreen
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  title="3speak Video Player">
</iframe>
```

### Responsive Embed

For responsive designs that adapt to screen size:

```html
<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <iframe 
    src="https://play.3speak.tv/watch?v=meno/1czchhmr&mode=iframe" 
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"
    frameborder="0"
    allowfullscreen
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    title="3speak Video Player">
  </iframe>
</div>
```

## URL Formats

### Legacy Videos (from videos collection)
```
https://play.3speak.tv/watch?v=owner/permlink&mode=iframe
```

### Direct Upload Videos (from embed-video collection)
```
https://play.3speak.tv/embed?v=owner/permlink&mode=iframe
```

### Full Player Page (without iframe mode)
```
https://play.3speak.tv/watch?v=owner/permlink
https://play.3speak.tv/embed?v=owner/permlink
```

## Parameters

- `v` - **Required**: Video identifier in format `owner/permlink`
- `mode=iframe` - **Optional**: Enables minimal UI for embedding (hides header and info panel)

## For Frontend Developers (PeakD, Ecency, etc.)

### Auto-Embed Detection

To automatically convert 3speak URLs into embedded players in your platform:

1. **Detect URLs** matching patterns:
   - `https://play.3speak.tv/watch?v=*`
   - `https://play.3speak.tv/embed?v=*`
   - `https://3speak.tv/watch?v=*` (legacy)

2. **Transform to iframe** by adding `&mode=iframe` parameter:
   ```javascript
   function embedVideo(url) {
     const embedUrl = url.includes('?') 
       ? `${url}&mode=iframe` 
       : `${url}?mode=iframe`;
     
     return `<iframe src="${embedUrl}" width="854" height="480" frameborder="0" allowfullscreen></iframe>`;
   }
   ```

3. **Example Implementation** (JavaScript):
   ```javascript
   // Detect 3speak URLs in markdown or content
   const videoRegex = /https:\/\/(play\.)?3speak\.tv\/(watch|embed)\?v=([^&\s]+)/g;
   
   const content = "Check out this video: https://play.3speak.tv/watch?v=meno/1czchhmr";
   
   const embeddedContent = content.replace(videoRegex, (match, subdomain, route, videoId) => {
     return `<iframe src="https://play.3speak.tv/${route}?v=${videoId}&mode=iframe" width="854" height="480" frameborder="0" allowfullscreen></iframe>`;
   });
   ```

### API Access (Alternative Method)

If you prefer to use your own video player, you can fetch video data via API:

```javascript
// For legacy videos
const response = await fetch('https://play.3speak.tv/api/watch?v=meno/1czchhmr');
const videoData = await response.json();
// Returns: { videoUrl, thumbnail, title, owner, permlink, etc. }

// For direct upload videos
const response = await fetch('https://play.3speak.tv/api/embed?v=meno/1czchhmr');
const videoData = await response.json();
```

Then use the `videoUrl` (HLS manifest) in your own Video.js or other HLS-compatible player.

## Features

- ✅ HLS streaming with adaptive quality
- ✅ Manual quality selector
- ✅ Automatic IPFS gateway fallback
- ✅ Thumbnail/poster support
- ✅ View counter integration
- ✅ Mobile responsive
- ✅ Fullscreen support
- ✅ Keyboard shortcuts
- ✅ Playback speed control

## Browser Compatibility

Works in all modern browsers:
- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest)
- Mobile browsers (iOS Safari, Chrome Mobile)

## Examples

### Full-width Embed
```html
<iframe 
  src="https://play.3speak.tv/watch?v=meno/1czchhmr&mode=iframe" 
  style="width: 100%; height: 500px;"
  frameborder="0"
  allowfullscreen>
</iframe>
```

### Fixed Size Embed
```html
<iframe 
  src="https://play.3speak.tv/watch?v=meno/1czchhmr&mode=iframe" 
  width="640" 
  height="360"
  frameborder="0"
  allowfullscreen>
</iframe>
```

### In a Blog Post (with caption)
```html
<figure>
  <iframe 
    src="https://play.3speak.tv/watch?v=meno/1czchhmr&mode=iframe" 
    width="854" 
    height="480"
    frameborder="0"
    allowfullscreen>
  </iframe>
  <figcaption>My awesome video on 3speak</figcaption>
</figure>
```

## Need Help?

- API Documentation: `https://play.3speak.tv/api`
- Issues: Open an issue on GitHub
- Contact: @meno on Hive

## License

This player is open source and free to use for embedding 3speak videos.
