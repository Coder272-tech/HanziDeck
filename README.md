# HanziDeck

A lightweight Chinese vocabulary flashcard web app focused on fast review, repetition, and retention.  
Built with **SvelteKit**, **Vite**, and **Tailwind CSS**.

The goal is a no-friction study tool for Mandarin learners:
- Quick sessions
- Minimal UI
- Keyboard- and mobile-friendly
- Works well as a PWA later if desired

---

## Features (Current / Planned)

- Flashcard-style vocabulary review
- Chinese characters + pinyin + English
- Simple reveal / flip interaction
- Progress through decks
- Clean, distraction-free UI

Planned:
- HSK-based decks
- Spaced repetition
- Offline support
- Import/export word lists
- Mobile-first optimizations

---

## Tech Stack

- **SvelteKit** – app framework
- **Vite** – dev server & bundler
- **Tailwind CSS** – styling
- **PostCSS** – CSS processing
- **Node.js** – runtime

---

## Project Structure

HanziDeck/  
├─ src/  
│ ├─ routes/  
│ │ ├─ \+layout.svelte  
│ │ ├─ \+page.svelte  
│ │ └─ layout.css  
│ ├─ lib/  
│ │ └─ components/  
│ └─ app.html  
├─ static/  
├─ postcss.config.cjs  
├─ tailwind.config.js  
├─ vite.config.js  
├─ package.json  
└─ README.md


---

## Getting Started

### Prerequisites

- Node.js 18+
- npm

---

### Install Dependencies

```bash
npm install

```

### **Run the Dev Server**

`npm run dev`

By default, the app runs on:

`http://localhost:5173`

If running inside Docker, ensure the port is published to the host.

---

## **Tailwind Setup Notes**

* Global styles are defined in:

`src/routes/layout.css`

`@tailwind base;`  
`@tailwind components;`  
`@tailwind utilities;`

* The file is imported in `+layout.svelte`.

* PostCSS uses the Tailwind v4 plugin:

`// postcss.config.cjs`  
`module.exports = {`  
  `plugins: [`  
    `require('@tailwindcss/postcss'),`  
    `require('autoprefixer')`  
  `]`  
`};`

---

## **Docker Notes (Optional)**

When running inside Docker:

* Publish the dev port (`5173`) to the host

* Bind-mount the project directory so changes persist

* Use `--host` if needed:

`npm run dev -- --host`

---

## **Roadmap**

* Flashcard engine

* Deck management

* HSK word lists

* Progress tracking

* Offline-first support

* Mobile optimizations

---

## **License**

MIT (or TBD)

---

## **Disclaimer**

This project is under active development.  
 Expect breaking changes until the core flashcard flow stabilizes.

