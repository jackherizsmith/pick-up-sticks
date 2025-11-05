# Pick Up Sticks

A mobile-first web game where players must select sticks in the correct order they can be picked up. Built with NuxtJS and ready for production deployment with Docker.

## Features

- 🎮 Interactive stick selection gameplay
- 📱 Mobile-first responsive design
- 🎨 Colourful stick visualisation with SVG
- ✅ Order validation algorithm
- 📋 Share results to clipboard
- 🐳 Dockerised for easy deployment
- ⚡ Built with Nuxt 3 and Vue 3

## How to Play

1. Click "Start Game" to begin
2. Observe the sticks laid out on the canvas
3. Select sticks in the order they can be picked up (sticks that aren't blocked by others)
4. Once all sticks are selected, click "Check Order"
5. Share your results with friends!

## Development

### Prerequisites

- Node.js 20+
- Docker & Docker Compose (for containerised deployment)

### Local Development

Install dependencies:

```bash
npm install
```

Start the development server on `http://localhost:3000`:

```bash
npm run dev
```

### Build for Production

Build the application:

```bash
npm run build
```

Preview production build locally:

```bash
npm run preview
```

## Docker Deployment

### Build and Run with Docker Compose

```bash
docker-compose up -d
```

The application will be available at `http://localhost:3000`

### Stop the Application

```bash
docker-compose down
```

### Manual Docker Build

Build the image:

```bash
docker build -t pick-up-sticks .
```

Run the container:

```bash
docker run -p 3000:3000 pick-up-sticks
```

## Production Deployment

The application is production-ready and can be deployed to:

- Any Docker-compatible hosting platform (AWS ECS, Google Cloud Run, Azure Container Instances)
- Kubernetes clusters
- VPS with Docker installed
- Cloud platforms with Nuxt support (Vercel, Netlify, etc.)

### Environment Variables

The application uses NODE_ENV=production in the Docker container. No additional environment variables are required.

## Project Structure

```
pick-up-sticks/
├── app/
│   ├── app.vue              # Main game component
│   └── assets/
│       └── css/
│           └── global.css   # Global styles
├── public/                  # Static assets
├── Dockerfile              # Production Docker configuration
├── docker-compose.yml      # Docker Compose configuration
├── nuxt.config.ts          # Nuxt configuration
└── package.json            # Dependencies
```

## Technology Stack

- **Framework**: Nuxt 3
- **UI Library**: Vue 3 with Composition API
- **Graphics**: SVG for stick rendering
- **Styling**: Scoped CSS with mobile-first approach
- **Container**: Docker with multi-stage builds
- **Runtime**: Node.js 20 Alpine

## License

MIT
