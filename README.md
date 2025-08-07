# Vue.js Appointment Booking System

A modern, minimal appointment booking interface built with Vue.js 3 and Tailwind CSS. Features a clean green and white design with a custom toast notification system.

## 🌟 Features

- **Multi-step Booking Process**: Guided 7-step appointment booking flow
- **Department Selection**: Choose from various service categories
- **Service Selection**: Pick specific services with duration and staff info
- **Guest Management**: Specify number of guests (1-10)
- **Staff Selection**: Choose preferred stylist or any available staff
- **Date & Time Picker**: Interactive calendar with real-time slot availability
- **Contact Form**: Comprehensive form with validation
- **Custom Toast System**: Beautiful, animated notifications
- **Responsive Design**: Works seamlessly on all devices
- **Real-time Validation**: Instant form validation feedback

## 🎨 Design Features

- **Minimal Aesthetic**: Clean, award-winning design
- **Green Primary Color**: Professional green color scheme (#16a34a)
- **Black Typography**: High contrast black text on white backgrounds
- **No Gradients**: Pure, clean backgrounds throughout
- **Enhanced Toasts**: Modern notification system with colored borders
- **Smooth Animations**: Subtle transitions and hover effects

## 🚀 Quick Start

### Prerequisites

- Node.js 16+ 
- Vue.js 3
- Nuxt.js (if using Nuxt UI components)
- Tailwind CSS

### Installation

1. **Clone or copy the component**:
```bash
# Copy the index.vue file to your Vue.js project
cp index.vue src/pages/booking/
```

2. **Install required dependencies**:
```bash
# For Vue 3 + Composition API
npm install vue@^3.0.0

# For date handling
npm install @internationalized/date

# For Nuxt UI components (if using Nuxt)
npm install @nuxt/ui
```

3. **Configure Tailwind CSS**:
```javascript
// tailwind.config.js
module.exports = {
  content: [
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        green: {
          50: '#f0fdf4',
          100: '#dcfce7',
          500: '#b9243f',
          600: '#951a30',
          700: '#751a29',
        }
      }
    },
  },
  plugins: [],
}
```

## 📁 Project Structure

```
src/
├── pages/
│       └── index.vue          # Main booking component
├── components/                   # UI components (if using component library)
└── assets/
    └── main.css # Tailwind CSS imports
```

## 🔧 Configuration

### API Endpoints

The component connects to these API endpoints:

```javascript
// Base URL
const API_BASE = 'https://restyle-api.netlify.app/.netlify/functions'

// Endpoints used:
// GET /getGroups - Fetch service departments
// GET /getDynamicService?id={groupId} - Fetch services for department
// GET /getStaff?id={userId} - Fetch staff information
// GET /getAllstaffslot?calendarId={id}&startDate={timestamp}&endDate={timestamp} - Fetch available slots
// GET /createContact?firstName={}&lastName={}&email={}&phone={}&notes={} - Create contact
// GET /bookAppointment?contactId={}&calendarId={}&startTime={}&endTime={} - Book appointment
```

### Environment Variables

No environment variables required - all API endpoints are hardcoded for the demo.

## 🎨 Customization

### Styling

The component uses Tailwind CSS classes. Key customization points:

```vue
<!-- Primary buttons -->
<UButton class="bg-green-600 hover:bg-green-700 text-white">

<!-- Selected states -->
<div class="bg-green-50 border-green-500">

<!-- Icons -->
<UIcon class="text-green-600">
```

### Toast Notifications

Customize toast appearance in the style section:

```vue
<style scoped>
/* Toast container positioning */
.fixed.top-4.right-4 {
  /* Adjust positioning */
}

/* Toast styling */
.bg-white.shadow-lg.rounded-xl {
  /* Customize appearance */
}
</style>
```

## 🔍 API Integration

### Custom API Endpoints

To use your own API, update the fetch URLs:

```javascript
// Replace in the component
const API_BASE = 'https://your-api.com/api'

// Update fetch calls
fetch(`\${API_BASE}/departments`)
fetch(`\${API_BASE}/services/\${departmentId}`)
// ... etc
```

### Data Format

Expected API response formats:

```javascript
// Departments/Groups
{
  "groups": [
    {
      "id": "dept1",
      "name": "Hair Services",
      "description": "Professional hair styling"
    }
  ]
}

// Services
{
  "calendars": [
    {
      "id": "service1",
      "name": "Haircut & Style",
      "slotDuration": 60,
      "teamMembers": [{"userId": "staff1"}]
    }
  ]
}

// Time Slots
{
  "formattedSlots": {
    "2024-01-15": ["9:00 AM", "10:00 AM", "11:00 AM"]
  }
}
```

## 🐛 Troubleshooting

### Common Issues

1. **Toast notifications not showing**:
   - Ensure z-index is high enough (z-50)
   - Check if container is properly positioned

2. **Calendar not loading**:
   - Verify @internationalized/date is installed
   - Check browser console for errors

3. **API calls failing**:
   - Check network tab for failed requests
   - Verify CORS settings on your API

4. **Styling issues**:
   - Ensure Tailwind CSS is properly configured
   - Check if all required classes are available

### Debug Mode

Enable console logging by uncommenting debug statements:

```javascript
// Uncomment these lines for debugging
console.log('API Response:', data)
console.log('Selected values:', { department, service, staff })
```

## 📦 Dependencies

### Required Dependencies

```json
{
  "dependencies": {
    "vue": "^3.0.0",
    "@internationalized/date": "^3.0.0"
  }
}
```

### Optional Dependencies (for Nuxt UI)

```json
{
  "devDependencies": {
    "@nuxt/ui": "^2.0.0",
    "tailwindcss": "^3.0.0"
  }
}
```

## 🚀 Deployment

### Netlify Deployment

1. **Build the project**:
```bash
npm run build
```

2. **Deploy to Netlify**:
```bash
# Using Netlify CLI
netlify deploy --prod --dir=dist
```

3. **Environment Variables** (if needed):
```bash
# In Netlify dashboard, add:
NUXT_PUBLIC_API_BASE=https://your-api.com
```

### Vercel Deployment

1. **Install Vercel CLI**:
```bash
npm i -g vercel
```

2. **Deploy**:
```bash
vercel --prod
```

## 🔒 Security Considerations

- **Input Validation**: All form inputs are validated client-side and should be validated server-side
- **API Security**: Ensure your API endpoints have proper authentication and rate limiting
- **CORS**: Configure CORS properly for your domain
- **Data Sanitization**: Sanitize all user inputs before processing

## 🎯 Performance Optimization

### Lazy Loading

```vue
<script setup>
// Lazy load heavy components
const Calendar = defineAsyncComponent(() => import('./Calendar.vue'))
</script>
```

### Bundle Size Optimization

```javascript
// Use tree-shaking for icons
import { CheckCircle, XCircle } from 'lucide-vue-next'
```

## 📊 Analytics Integration

### Google Analytics

```vue
<script setup>
// Track booking steps
const trackStep = (step) => {
  gtag('event', 'booking_step', {
    step_name: step,
    step_number: currentStep.value
  })
}
</script>
```

## 🧪 Testing

### Unit Tests

```javascript
// Example test with Vitest
import { mount } from '@vue/test-utils'
import BookingSystem from './BookingSystem.vue'

test('should render booking steps', () => {
  const wrapper = mount(BookingSystem)
  expect(wrapper.find('h1').text()).toBe('Book Your Appointment')
})
```

### E2E Tests

```javascript
// Example with Playwright
test('complete booking flow', async ({ page }) => {
  await page.goto('/booking')
  await page.click('[data-testid="department-hair"]')
  await page.click('[data-testid="continue-btn"]')
  // ... continue test flow
})
```

## 📄 License

MIT License

Copyright (c) 2024 Vue.js Appointment Booking System

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## 🤝 Contributing

### Development Setup

1. **Fork the repository**
2. **Clone your fork**:
```bash
git clone https://github.com/yourusername/vue-booking-system.git
cd vue-booking-system
```

3. **Install dependencies**:
```bash
npm install
```

4. **Start development server**:
```bash
npm run dev
```

### Contribution Guidelines

- Follow Vue.js 3 Composition API patterns
- Use TypeScript for type safety
- Write tests for new features
- Follow conventional commit messages
- Update documentation for new features

### Pull Request Process

1. Create a feature branch: `git checkout -b feature/amazing-feature`
2. Commit your changes: `git commit -m 'Add amazing feature'`
3. Push to the branch: `git push origin feature/amazing-feature`
4. Open a Pull Request

## 📞 Support

### Getting Help

- **Documentation**: Check this README first
- **Issues**: Open an issue on GitHub
- **Discussions**: Use GitHub Discussions for questions
- **Email**: contact@yourdomain.com

### Reporting Bugs

When reporting bugs, please include:

- Vue.js version
- Browser and version
- Steps to reproduce
- Expected vs actual behavior
- Console errors (if any)
