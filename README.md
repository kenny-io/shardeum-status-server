# Shardeum Network Status Monitor

A real-time monitoring system built with Next.js, Prometheus, and Express that tracks service availability and performance metrics across multiple endpoints.

## Features

- 🔍 Real-time service monitoring
- 📈 Latency tracking and visualization
- 🔄 Automatic retry mechanism for failed requests
- 🚦 Status indicators with tooltips
- 🎯 Group-based service organization

## Architecture

### Components

1. **Prometheus Exporter (Express Server)**
   - Runs on port 3002
   - Collects metrics:
     - `service_up`: Service availability (1 for up, 0 for down)
     - `service_response_time`: Response time in milliseconds
   - Implements retry logic for failed requests
   - Handles concurrent service checks

2. **Prometheus Server**
   - Runs on port 9090
   - Scrapes metrics from the exporter
   - Stores time-series data
   - Handles metric queries via PromQL

3. **Next.js Frontend**
   - Server-side rendered React (Next.js) application
   - Real-time metric visualization
   - Responsive status indicators
   - Latency graphs

### Monitoring Logic

#### Service Checks
- Concurrent service checking (3 services at a time)
- Configurable retry mechanism:
  - Maximum retries: 3
  - Retry delay: 2000ms (doubles with each retry)
  - Timeout: 10 seconds per request

#### Status Determination & Alerting
- Service status changes have a 5-minute grace period
- Service is considered "down" if:
  - Response status is not 200-299, OR
  - Response doesn't match expected format/content, AND
  - Service remains in failed state for >5 minutes
- Notification Logic:
  - DOWN notifications are sent only after 5-minute grace period
  - RECOVERY notifications are sent immediately after a confirmed downtime
  - Transient failures (<5 minutes) are ignored to reduce noise
  - Downtime duration is included in recovery notifications

#### State Management
- Each service maintains its own state:
  - Current up/down status
  - Grace period status
  - Last state change timestamp
  - Pending alert status
- Transient failures during grace period:
  - Do not trigger notifications
  - Are automatically cleared if service recovers
  - Reset grace period without generating alerts

#### Refresh Intervals
- Service checks: Every 60 seconds
- Realtime view: 1 second refresh
- Service status Indicator bars: Refresh every 5 minutes

## Setup

1. **Install Dependencies**
   ```bash
   npm install
   ```

2. **Configure Endpoints**
   - Edit `server/endpoints.json` to define monitored services
   - Each service can specify:
     - URL
     - Expected response format
     - Custom headers
     - Request body
     - Group assignment

3. **Start Prometheus**
   ```bash
   ./prometheus
   ```

4. **Start the Exporter and Next.js**
   ```bash
   npm run start:all
   ```

## Configuration

### Endpoint Configuration

```json
{
  "urls": [
    {
      "group": "Group Name",
      "servers": [
        {
          "url": "https://api.example.com",
          "name": "Service Name",
          "help": "Service description",
          "expectedResponse": {
            "field": "value"
          },
          "body": {
            "key": "value"
          }
        }
      ]
    }
  ]
}
```

### Environment Variables

- `PORT`: Exporter port (default: 3002)
- `PROMETHEUS_URL`: Prometheus server URL (default: http://localhost:9090)
- `ENDPOINTS_FILE`: Path to endpoints configuration (default: ./endpoints.json)
- `SLACK_WEBHOOK_URL`: Slack webhook URL for notifications (optional)

## Development

### Project Structure

```
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   ├── layout.tsx         # Root layout
│   └── page.tsx           # Main page
├── components/            # React components
├── hooks/                 # Custom React hooks
├── lib/                   # Utility functions
├── server/               # Backend services
│   ├── exporter.js       # Prometheus exporter
│   └── endpoints.json    # Service configuration
└── scripts/              # Helper scripts
```

## Error Handling

1. **Network Errors**
   - Automatic retry with exponential backoff
   - Maximum 3 retry attempts
   - Failed services marked as down after all retries exhausted

2. **Response Validation**
   - Checks HTTP status codes
   - Validates response format against expected schema
   - Handles partial matches for text responses


## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

MIT License - feel free to use this project for any purpose.



# Improvements for Dark/Light Mode Integration:

If you want to implement a working dark/light toggle functionality, here’s an update for your ThemeProvider and Header:

1. Update ThemeProvider: Ensure your ThemeProvider supports toggling themes and persisting preferences. Use libraries like next-themes if needed.

Example:

`// components/ui/providers/theme-provider.tsx
'use client';
import { ThemeProvider as NextThemesProvider } from 'next-themes';
export function ThemeProvider({ children, attribute }: { children: React.ReactNode; attribute: string }) {
    return <NextThemesProvider attribute={attribute}>{children}</NextThemesProvider>;
}`


2. Add Toggle Button: Include a toggle button in the Header component to switch themes.

`// components/Header.tsx
'use client';

import { useTheme } from 'next-themes';

export function Header() {
    const { theme, setTheme } = useTheme();

    return (
        <header className="flex justify-between p-4">
            <h1>Shardeum Network Status Monitor</h1>
            <button
                onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
                className="p-2 border rounded"
            >
                {theme === 'dark' ? 'Switch to Light Mode' : 'Switch to Dark Mode'}
            </button>
        </header>
    );
}`


3. Tailwind Configuration: Make sure your tailwind.config.js is set up to support dark mode via the class strategy:

`// tailwind.config.js
module.exports = {
    darkMode: 'class', // Enable class-based dark mode
    content: ['./pages//*.{js,ts,jsx,tsx}', './components//*.{js,ts,jsx,tsx}'],
    theme: {
        extend: {},
    },
    plugins: [],
};`


4. CSS Setup: Ensure your global styles include base styles for dark and light modes.

`/* globals.css */
body {
    @apply bg-white text-black;
}
body.dark {
    @apply bg-black text-white;
}`

This will enable users to toggle between dark and light modes seamlessly, and their preference will persist.


