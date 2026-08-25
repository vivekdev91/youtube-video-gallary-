# Family Video Gallery

A premium, cinematic, private family video gallery built with React and Vite. This application provides an elegant way to display a curated collection of YouTube videos with built-in casual copy deterrents and a privacy mode.

## 🚀 Getting Started

### Prerequisites
- Node.js (v18 or higher recommended)

### Installation
1. Clone or download the repository.
2. Open a terminal in the project directory.
3. Run the following command to install dependencies:
   ```bash
   npm install
   ```

### Running Locally
To start the development server:
```bash
npm run dev
```
Open your browser and navigate to the local URL provided (usually `http://localhost:5173`).

### Building for Production
To create a production build:
```bash
npm run build
```
The compiled files will be located in the `dist` directory. You can deploy this folder to any static hosting provider (e.g., Vercel, Netlify, GitHub Pages, AWS S3).

## 📹 Video Management

The most important feature of this gallery is how easy it is to manage. You do **not** need to touch any React code to add or remove videos.

All video links are stored in `public/videos.json`.

### How to Add or Remove a Video
1. Open `public/videos.json`.
2. To **add** a video, simply add its YouTube URL to the array.
3. To **remove** a video, delete its URL from the array.

Example `videos.json`:
```json
[
  "https://www.youtube.com/watch?v=ABC12345",
  "https://youtu.be/XYZ98765",
  "https://www.youtube.com/embed/DEF45678"
]
```
The application will automatically parse the URLs, generate the YouTube thumbnails, and update the memory numbers (e.g., "Memory 01", "Memory 02").

## 🔒 Security & Privacy

### How YouTube Private Permissions Work
The videos displayed here should be uploaded to your YouTube channel as **Private**. To allow family members to watch them, you must add their Google Account email addresses in the YouTube Studio video visibility settings.

If a visitor to this website is not logged into an authorized Google account in their browser, the YouTube player will naturally show YouTube's access restriction message. **The frontend application does not replace YouTube authorization.**

### Frontend Protection Limitations
This website includes several casual copy deterrents, including:
- Disabled right-click (context menu) on the gallery area.
- Prevention of ordinary image dragging.
- Interception of common casual shortcuts (Ctrl+S, Ctrl+U, Ctrl+C, F12).
- Privacy Mode: The gallery content is obscured when you switch to another browser tab.
- Watermarked thumbnails ("PRIVATE FAMILY ARCHIVE").

**🚨 IMPORTANT SECURITY DISCLAIMER 🚨**
Private YouTube videos are protected by YouTube/Google account permissions. The frontend application does not replace YouTube authorization. Right-click blocking, keyboard shortcut blocking, privacy overlays, and similar browser-side measures only discourage casual copying. A user who is authorized to watch a video may still potentially record their screen, inspect source code, DevTools, or URLs. No frontend-only website can guarantee that displayed video content cannot be copied. Do not use frontend-only mechanisms for absolute security.

### Deployment Recommendations
- **HTTPS is strongly recommended** for deployment to protect the transmission of the webpage and ensure modern browser security features function correctly.
- **Content Security Policy (CSP)**: For advanced deployment, configure your web server to include a CSP header that restricts embedding to YouTube.
