# JW Player Implementation Guide for 3Speak

This document details how 3Speak implements JW Player for video playback, including CSS styling, responsive behavior, and vertical video handling.

## Table of Contents
1. [Player Setup](#player-setup)
2. [CSS Styling](#css-styling)
3. [Responsive Behavior](#responsive-behavior)
4. [Vertical Video Handling](#vertical-video-handling)
5. [Player Configuration](#player-configuration)
6. [Key Implementation Details](#key-implementation-details)

---

## Player Setup

### JW Player CDN
```html
<script src="https://cdn.jwplayer.com/libraries/j7Kz0Rae.js"></script>
```

### HTML Container
The player is initialized on a simple `<div>` or `<video>` element:

**Watch Page (views/new/watch.twig, line 138):**
```html
<video style="width: 100%" controls="controls"
       poster="{{ APP_IMAGE_CDN_DOMAIN }}/{{ video.permlink }}/poster.png"
       id="player"></video>
```

**Embed Page (views/embed/index.twig, line 28):**
```html
<video controls="controls" 
       poster="{{ APP_IMAGE_CDN_DOMAIN }}/{{ v.permlink }}/poster.png"
       id="player" 
       style="width: 100%; aspect-ratio: 16/9">
</video>
```

### JavaScript Initialization
```javascript
const player = jwplayer('player').setup({
  file: '{{video.playUrl}}',  // HLS manifest URL (default.m3u8)
  image: "{{ video.baseThumbUrl }}",  // Poster image
  playbackRateControls: [0.75, 1, 1.25, 1.5, 1.75, 2],
  autostart: true,
  muted: true,
  // ... additional config
});
```

---

## CSS Styling

### Main Player Styles (`public/css/player.css`)

#### 1. **Big Play Button (Center Play Icon)**
```css
.video-js .vjs-big-play-button {
    background-color: rgba(88, 139, 62, 0.3);
    font-size: 3.5em;
    border-radius: 12%;
    height: 1.4em !important;
    line-height: 1.4em !important;
    margin-top: -0.7em !important;
    /* Positioned at center by JW Player default */
    top: 50%;
    left: 50%;
    margin-left: -1em;
    margin-top: -1em;
    width: 2em;
    height: 2em;
    line-height: 2em;
    border: none;
    border-radius: 50%;
}

/* Hover state */
.video-js:hover .vjs-big-play-button,
.video-js .vjs-big-play-button:focus,
.video-js .vjs-big-play-button:active {
    background-color: rgba(88, 139, 62, 0.9);
}
```

#### 2. **Loading Spinner**
```css
.video-js .vjs-loading-spinner {
    border-color: rgba(88, 139, 62, 0.84);
}
```

#### 3. **Control Bar (Bottom Controls)**
```css
.video-js .vjs-control-bar {
    background-color: #588b3e !important;
    color: #ffffff;
    font-size: 11px;
}

/* Progress bar and volume colors */
.video-js .vjs-play-progress,
.video-js .vjs-volume-level {
    background-color: rgba(14, 34, 61, 0.8);
}
```

#### 4. **Progress Bar Positioning**
```css
.video-js .vjs-progress-control {
    position: absolute;
    left: 0;
    right: 0;
    width: 100%;
    height: .5em;
    top: -.5em;  /* Positioned above control bar */
}

/* Hover effect on progress bar */
.video-js .vjs-progress-control:hover {
    height: 1.5em;
    top: -1.5em;
}
```

#### 5. **Auto-Hide Control Bar (User Inactive)**
```css
/* Hide control bar when user is inactive during playback */
.video-js.vjs-has-started.vjs-playing.vjs-user-inactive .vjs-control-bar {
    visibility: visible;
    opacity: 1;
    -webkit-transform: translateY(3em);  /* Slide down */
    transform: translateY(3em);
    transition: transform 1s ease 0s;
}

/* Thin progress bar when controls hidden */
.video-js.vjs-has-started.vjs-playing.vjs-user-inactive .vjs-progress-control {
    height: .25em;
    top: -.25em;
    pointer-events: none;
    transition: height 1s, top 1s;
}

/* Completely hide progress bar in fullscreen when inactive */
.video-js.vjs-has-started.vjs-playing.vjs-user-inactive.vjs-fullscreen .vjs-progress-control {
    opacity: 0;
    transition: opacity 1s ease 1s;
}
```

#### 6. **Menu Popup Styles**
```css
.video-js .vjs-menu-button-popup .vjs-menu .vjs-menu-content {
    background-color: transparent;
    width: 12em;
    left: -3.5em;
    padding-bottom: .5em;
}

.video-js .vjs-menu-button-popup .vjs-menu .vjs-menu-item {
    background-color: #151b17;
    margin: .3em 0;
    padding: .5em;
    border-radius: .3em;
}

.vjs-menu-item:hover {
    background-color: #434343;
}

.video-js .vjs-menu-button-popup .vjs-menu .vjs-menu-item.vjs-selected {
    background-color: #2483d5;
}
```

---

## Responsive Behavior

### Container Layout

**Watch Page:**
```html
<div class="col-md-8" id="playerCol" style="padding-left: 0px !important;">
  <div class="single-video-left">
    <div class="single-video-title box mb-3 clearfix">
      <div class="single-video">
        <div style="width: 100%;">
          <video style="width: 100%" controls="controls" id="player"></video>
        </div>
      </div>
    </div>
  </div>
</div>
```

- Player is in an 8-column Bootstrap grid (col-md-8)
- 100% width fills the container
- Sidebar with video info/comments occupies remaining 4 columns

**Embed Page:**
```html
<body class="bg-dark" style="height: 100%">
  <video controls="controls" id="player" style="width: 100%; aspect-ratio: 16/9"></video>
</body>
```

- Full viewport width
- Uses CSS `aspect-ratio: 16/9` for embed sizing

### JW Player Aspect Ratio CSS

**Embed Layout (views/embed/layout.twig):**
```css
.jwplayer.jw-flag-aspect-mode {
  max-height: 100vh;
}

body {
  margin: 0 !important;
}
```

**Key Point:** JW Player uses the `.jw-flag-aspect-mode` class to maintain aspect ratio based on video dimensions.

---

## Vertical Video Handling

### How JW Player Handles Vertical Videos

**JW Player automatically detects video dimensions from the HLS manifest** and applies appropriate aspect ratio styling. Here's how it works:

#### 1. **Automatic Aspect Ratio Detection**
- JW Player reads the video dimensions from the HLS stream metadata
- It adds the class `.jw-flag-aspect-mode` to the player container
- The player maintains the correct aspect ratio (9:16 for vertical, 16:9 for horizontal)

#### 2. **Container Width = 100%**
```html
<video style="width: 100%" id="player"></video>
```

**This is the key:** By setting only the width to 100% (not height), JW Player:
- Scales the player to fill the container width
- Automatically calculates the height based on video aspect ratio
- Centers vertical videos properly within the available space

#### 3. **Embed Page Vertical Video Behavior**
```css
.jwplayer.jw-flag-aspect-mode {
  max-height: 100vh;  /* Prevents vertical videos from exceeding viewport */
}
```

This ensures vertical videos don't overflow the viewport height.

#### 4. **No Manual Dimension Calculation Needed**
3Speak does **NOT** manually detect or handle vertical videos. JW Player handles it automatically by:
- Reading video metadata from HLS manifest
- Applying responsive aspect ratio
- Maintaining proper centering

### Example: How It Works

**Horizontal Video (16:9 - 1920x1080):**
- Container width: 100% (e.g., 800px)
- JW Player calculates height: 800px × (9/16) = 450px
- Result: 800x450 player

**Vertical Video (9:16 - 1080x1920):**
- Container width: 100% (e.g., 800px)
- JW Player calculates height: 800px × (16/9) = 1422px
- If exceeds viewport: `max-height: 100vh` kicks in
- Result: Player is centered, proper aspect ratio maintained

---

## Player Configuration

### Complete Setup Object (from views/new/watch.twig)

```javascript
const player = jwplayer('player').setup({
  // Video source
  file: '{{video.playUrl}}',  // HLS manifest URL
  image: "{{ video.baseThumbUrl }}",  // Poster thumbnail
  
  // Playback controls
  playbackRateControls: [0.75, 1, 1.25, 1.5, 1.75, 2],
  autostart: true,
  muted: true,
  
  // Branding
  abouttext: "Powered by 3Speak",
  aboutlink: "https://3speak.tv/?utm_source=player_context",
  
  // Logo watermark
  logo: {
    "file": "https://images.hive.blog/p/[...]/logo.png",
    "link": "https://3speak.tv/?utm_souce=player_brand",
    "hide": "true",  // Hidden by default, shows on hover
    "position": "top-left"
  },
  
  // Chromecast support
  cast: {},
  
  // Google Analytics
  ga: {},
  
  // Floating player (picture-in-picture)
  floating: {
    dismissible: true
  },
  
  // Related videos
  related: {
    onclick: 'link',
    oncomplete: 'show',
  },
  
  // Social sharing
  sharing: {
    code: '<iframe width="560" height="315" src="https://3speak.tv/embed?v={{ video.owner }}/{{ video.permlink }}" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>',
    sites: ["facebook", "twitter", "reddit", "email"]
  }
});
```

---

## Key Implementation Details

### 1. **HLS Manifest Structure**
JW Player expects HLS manifests with proper video metadata:

```
/{{ permlink }}/default.m3u8  # Master playlist
/{{ permlink }}/720p/index.m3u8
/{{ permlink }}/480p/index.m3u8
/{{ permlink }}/360p/index.m3u8
```

The manifest includes:
- Video dimensions (width x height)
- Bitrate information
- Codec data

JW Player reads this to determine aspect ratio.

### 2. **Poster Images**
```javascript
image: "{{ video.baseThumbUrl }}"
```

Poster path: `{{ APP_IMAGE_CDN_DOMAIN }}/{{ video.permlink }}/poster.png`

### 3. **Fullscreen Behavior**
```css
.video-js.vjs-fullscreen,
.video-js.vjs-fullscreen .vjs-tech {
    width: 100%!important;
    height: 100%!important;
}
```

In fullscreen:
- Player fills entire screen
- Vertical videos are centered with pillarboxing (black bars on sides)
- Horizontal videos fill width

### 4. **Theatre Mode CSS**
```css
#theatreButton {
  padding-top: 10px;
}
```

Theatre mode likely expands the player column width dynamically.

### 5. **Event Listeners**
```javascript
player.on("pause", (event) => {
  running = false;
});

player.on("play", (event) => {
  running = true;
  timer();
});

player.on('adError', (event) => {
  console.log("Ad Request Error: \n" + event.message);
});
```

---

## Tips for Video.js Implementation

Based on JW Player's approach, here's how to handle vertical videos in Video.js:

### 1. **Use Fluid Mode with Aspect Ratio**
```javascript
videojs('player', {
  fluid: true,  // Maintains aspect ratio
  aspectRatio: '16:9',  // Default, will auto-adjust
  responsive: true
});
```

### 2. **Let Video.js Detect Dimensions**
Video.js automatically reads video metadata. Don't manually set height:

```html
<!-- Good -->
<video id="player" class="video-js vjs-default-skin" style="width: 100%"></video>

<!-- Bad - Don't force height -->
<video id="player" style="width: 100%; height: 500px"></video>
```

### 3. **CSS for Vertical Videos**
```css
.video-js {
  width: 100%;
  height: auto;  /* Let it calculate based on aspect ratio */
  max-height: 100vh;  /* Prevent overflow on vertical videos */
}

/* Center vertical videos */
.video-js.vjs-fluid {
  padding-top: 0 !important;  /* Remove default padding-top hack */
  height: auto;
}
```

### 4. **Detect and Handle Vertical Videos**
```javascript
player.ready(function() {
  const videoWidth = player.videoWidth();
  const videoHeight = player.videoHeight();
  
  if (videoHeight > videoWidth) {
    // Vertical video
    player.addClass('vertical-video');
    player.aspectRatio(`${videoWidth}:${videoHeight}`);
  }
});
```

### 5. **Container Structure**
```html
<div class="video-container" style="max-width: 800px; margin: 0 auto;">
  <video id="player" class="video-js vjs-fluid"></video>
</div>
```

The key is letting the player calculate dimensions based on video metadata, not forcing fixed heights.

---

## Summary

**JW Player's Vertical Video Strategy:**
1. Set container width to 100%
2. Let JW Player read video dimensions from HLS manifest
3. Apply `.jw-flag-aspect-mode` class for responsive aspect ratio
4. Use `max-height: 100vh` to prevent viewport overflow
5. No manual dimension calculation or detection needed

**For Video.js:**
- Use `fluid: true` mode
- Read `player.videoWidth()` and `player.videoHeight()` on ready
- Dynamically set `aspectRatio` for vertical videos
- Use similar CSS approach with `max-height: 100vh`

The magic is in **not forcing dimensions** and letting the player library handle aspect ratios based on actual video metadata.
