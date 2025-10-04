# Task Management Frontend

Vue 3 + Vite frontend with a futuristic dark theme UI.

## What's Inside

- **Vue 3** with TypeScript and Composition API
- **Tailwind CSS** for styling with custom animations
- **shadcn-vue** components (built on Radix Vue)
- **Vue Router** with protected routes
- JWT authentication

## Quick Start

```bash
npm install
npm run dev
```

Login with `admin` / `admin123`

## Features

The app has a login page and a full task management system:
- Create, read, update, delete tasks
- Filter by status, search by title
- Sort by date or title
- Pagination
- Everything's responsive and animated

## Design Choices

I went with Vue 3 because it has great TypeScript support and the Composition API makes the code cleaner. Tailwind was a no-brainer for rapid styling, and I added custom animations to make everything feel smooth.

For the UI components, I used shadcn-vue instead of a component library like Vuetify because you get full control over the code - the components live in your project, not hidden in node_modules. Plus they're accessible out of the box.

The futuristic dark theme with gradient backgrounds and glassmorphism effects makes it look modern. Every interaction has smooth animations - buttons scale on hover, cards slide in with staggered delays, status badges have gradient backgrounds.

## Project Structure

```
src/
├── components/ui/    # shadcn-vue components
├── views/           # Login and Tasks pages
├── router/          # Route config with auth guards
└── lib/             # Utils (cn for class merging)
```

## API Connection

The frontend talks to the backend at `http://localhost:3000`. If you deploy, update the URLs in:
- `src/views/Login.vue`
- `src/views/Tasks.vue`

## Deployment

Build with `npm run build` and deploy the `dist/` folder.

Don't forget to update the API URL to your deployed backend before building.
