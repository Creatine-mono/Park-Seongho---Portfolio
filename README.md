# Park Seongho — Portfolio

A 3D interactive portfolio built with Three.js, React, and TypeScript. Features a fully navigable 3D room scene with a monitor that renders an in-world profile terminal.

**Live:** [pk42ac.com](https://pk42ac.com)

---

## Tech Stack

- **Three.js** — 3D scene rendering
- **React + TypeScript** — UI overlay and monitor screen
- **GLSL** — Custom shaders (coffee steam, monitor screen)
- **Express + Node.js** — Static file server + contact form email API
- **Webpack** — Bundler
- **PM2** — Process manager
- **Nginx** — Reverse proxy + HTTPS
- **AWS EC2** — Deployment

---

## File Structure

```
Park-Seongho---Portfolio/
├── server/
│   └── index.ts                  # Express server (port 8080, email API)
│
├── src/
│   ├── index.html                # Entry HTML
│   ├── script.ts                 # App entry point
│   ├── style.css
│   ├── types.d.ts
│   ├── tsconfig.json
│   └── Application/
│       ├── Application.ts        # Main application class
│       ├── Renderer.ts           # Three.js WebGL renderer
│       ├── sources.ts            # Asset source definitions
│       │
│       ├── Audio/
│       │   ├── AudioManager.ts   # Audio playback controller
│       │   └── AudioSources.ts   # Audio source definitions
│       │
│       ├── Camera/
│       │   ├── Camera.ts         # Camera controls and transitions
│       │   └── CameraKeyframes.ts
│       │
│       ├── Shaders/
│       │   ├── coffee/           # Coffee steam shader
│       │   │   ├── vertex.glsl
│       │   │   └── fragment.glsl
│       │   └── screen/           # Monitor screen shader
│       │       ├── vertex.glsl
│       │       └── fragment.glsl
│       │
│       ├── UI/
│       │   ├── App.tsx           # React app root
│       │   ├── Animation.ts      # UI animation helpers
│       │   ├── EventBus.ts       # UI event bus
│       │   ├── index.ts
│       │   ├── style.css
│       │   └── components/
│       │       ├── InfoOverlay.tsx     # Name/title overlay (top-left)
│       │       ├── InterfaceUI.tsx     # Main UI wrapper
│       │       ├── LoadingScreen.tsx   # Loading screen
│       │       ├── HelpPrompt.tsx      # Mouse/keyboard hints
│       │       ├── FreeCamToggle.tsx   # Free camera toggle button
│       │       └── MuteToggle.tsx      # Mute/unmute button
│       │
│       ├── Utils/
│       │   ├── BakedModel.ts     # Baked texture model loader
│       │   ├── Debug.ts          # Debug GUI (dev only)
│       │   ├── EventEmitter.ts   # Base event emitter
│       │   ├── Loading.ts        # Asset loading manager
│       │   ├── Mouse.ts          # Mouse position tracker
│       │   ├── Resources.ts      # Resource loader
│       │   ├── Sizes.ts          # Viewport size manager
│       │   └── Time.ts           # Animation frame timer
│       │
│       └── World/
│           ├── World.ts          # Scene composition
│           ├── Computer.ts       # Computer mesh + monitor screen
│           ├── MonitorScreen.ts  # In-world monitor rendering
│           ├── CoffeeSteam.ts    # Particle coffee steam effect
│           ├── Cursor.ts         # 3D cursor object
│           ├── Decor.ts          # Decorative objects
│           ├── Environment.ts    # Lighting and environment map
│           └── Hitboxes.ts       # Click detection hitboxes
│
├── static/                       # Static assets (copied to public/ on build)
│   ├── audio/
│   │   ├── atmosphere/           # Ambient office sounds
│   │   ├── cc/                   # Typing sounds
│   │   ├── computer/             # Computer idle sounds
│   │   ├── keyboard/             # Key press sounds
│   │   ├── mouse/                # Mouse click sounds
│   │   ├── radio/                # Background music
│   │   └── startup/              # Startup sound
│   │
│   ├── docs/
│   │   └── resume.pdf
│   │
│   ├── draco/                    # Draco decoder (GLTF compression)
│   │
│   ├── images/                   # Favicon and preview images
│   │
│   ├── models/
│   │   ├── Computer/             # Computer GLTF + baked texture
│   │   ├── Decor/                # Decor GLTF + baked texture
│   │   └── World/                # Environment GLTF + baked texture
│   │
│   ├── monitor-page/
│   │   └── index.html            # In-world monitor profile terminal (self-contained HTML)
│   │
│   ├── profile/
│   │   ├── activities/           # Activity images
│   │   ├── awards/               # Award certificate images
│   │   └── profile-photo.jpg
│   │
│   └── textures/
│       ├── UI/                   # SVG icons
│       ├── environmentMap/       # Cubemap (6 faces)
│       └── monitor/
│           ├── cursors/          # In-world cursor texture
│           ├── layers/           # Monitor overlay layers (reflection, shadow, smudges)
│           └── video/            # Monitor screen video textures
│
├── bundler/
│   ├── webpack.common.js
│   ├── webpack.dev.js
│   └── webpack.prod.js
│
├── public/                       # Build output (served by Express)
├── package.json
├── buildspec.yaml                # AWS CodeBuild spec
├── .babelrc
├── .gitignore
└── .prettierrc
```

---

## Getting Started

### Development
```bash
npm install
npm run dev
```

### Production Build
```bash
npm run build
```

### Run Server
```bash
npm install -g ts-node pm2
pm2 start server/index.ts --interpreter ts-node --name portfolio
```

---

## Deployment (AWS EC2 + Nginx + HTTPS)

### 1. Build locally and push
```bash
npm run build
git add public/
git commit -m "Add production build"
git push
```

### 2. On the server
```bash
git pull
sudo apt install nginx certbot python3-certbot-nginx -y
```

### 3. Nginx config (`/etc/nginx/sites-available/portfolio`)
```nginx
server {
    listen 80;
    server_name pk42ac.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/portfolio /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl restart nginx
sudo certbot --nginx -d pk42ac.com
```

### 4. Start server
```bash
pm2 start server/index.ts --interpreter ts-node --name portfolio
pm2 save && pm2 startup
```

---

## Environment Variables

Required for the contact form email feature:

| Variable | Description |
|---|---|
| `FOLIO_EMAIL` | Gmail address used to send emails |
| `FOLIO_PASSWORD` | Gmail App Password (not account password) |
