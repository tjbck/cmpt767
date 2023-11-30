# Amazon Review Dataset Visualization Project

This project aims to provide an effective visualization of the Amazon review dataset. It consists of two main components: a React-based visualization using D3 and a Svelte-based visualization using the Canvas API via p5.js and native HTML input, integrated with Chart.js to create a unified view.

## Setup Instructions

### Clone Repositories

Ensure that you have both repositories cloned to your local machine:

1. Clone the React repository:

   ```bash
   git clone https://github.com/SiyuAn166/vamazon
   ```

2. Clone the Svelte repository:

   ```bash
   git clone https://github.com/tjbck/cmpt767
   ```

### Setting up Svelte Visualization

1. Navigate to the Svelte repository:

   ```bash
   cd cmpt767
   ```

2. Install dependencies using npm:

   ```bash
   npm install
   ```

3. Run the Svelte development server:

   ```bash
   npm run dev
   ```

   The Svelte server will be running on `http://localhost:5173`.

### Setting up React Visualization

1. Navigate to the React repository:

   ```bash
   cd vamazon
   ```

2. Install dependencies using npm:

   ```bash
   npm install
   ```

3. Start the React server:

   ```bash
   npm run start
   ```

   The React server will be running on `http://localhost:3000`.

### Viewing the Visualizations

1. Both the Svelte and React visualizations will be displayed on the React end.

2. The integration is achieved by using an iframe to embed the Svelte server into the React application. Communication between the two components is facilitated through the `window.postMessage` function.

3. Open your web browser and navigate to `http://localhost:3000` to view the complete visualization.

## Note

Make sure to run both projects simultaneously to ensure proper communication between the Svelte and React components.

Enjoy exploring the Amazon review dataset through the interactive visualizations!
