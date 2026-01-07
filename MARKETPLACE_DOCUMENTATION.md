# R&D Asset Marketplace - Forum Oceano
## Implementation Documentation

---

## 1. Overview

The R&D Asset Marketplace is a comprehensive digital catalog for Hub Azul partner institutions to publish and share research infrastructure, equipment, and services. Built with scalability in mind for Phase 2 expansion.

### Technology Stack
- **Frontend:** React 18.3.1 + TypeScript
- **Styling:** Tailwind CSS v4
- **Icons:** Lucide React
- **Charts:** Recharts (ready for Phase 2 dashboards)
- **External API:** Custom integration service

---

## 2. Project Structure

```
/src/app/
├── App.tsx                          # Main application with routing
├── types/
│   └── cms-schema.ts                # TypeScript definitions for CMS
├── data/
│   └── cms-assets.ts                # Mock CMS data (8 sample assets)
├── pages/
│   ├── MarketplaceCatalog.tsx       # Main catalog page
│   └── AssetDetailPage.tsx          # Individual asset pages
├── components/
│   ├── FilterSidebar.tsx            # Advanced filtering UI
│   ├── ActiveFilters.tsx            # Filter chips display
│   ├── AssetGrid.tsx                # Asset card grid
│   └── ContactFormModal.tsx         # Contact form with file upload
├── services/
│   └── externalApi.ts               # External API integration
└── utils/
    └── filterUtils.ts               # Filter logic and URL state
```

---

## 3. Routing Structure

The application uses hash-based routing to support deployment on static hosts:

- **Catalog Page:** `#/rd-asset-marketplace` or `#/marketplace`
- **Asset Detail:** `#/rd-asset-marketplace/assets/{slug}`

Examples:
- `https://forumoceano.pt/en#/rd-asset-marketplace`
- `https://forumoceano.pt/en#/rd-asset-marketplace/assets/marine-robotics-testing-tank`

### URL State Management
Filters are persisted in URL query parameters:
```
?assetType=infrastructure,specific-service&city=Lisboa&skills=marine%20robotics&technicalSupport=yes
```

This enables:
- ✅ Shareable filtered views
- ✅ Browser back/forward navigation
- ✅ Bookmark specific searches

---

## 4. CMS Schema

### Asset Data Structure

```typescript
interface Asset {
  // Core identifiers
  id: string;
  slug: string;                      // URL-friendly identifier
  name: string;
  
  // Classification
  assetType: 'infrastructure' | 'infrastructure-with-support' | 'specific-service';
  subType: 'equipment' | 'laboratory' | 'technical-support' | 'consulting' | 'analysis';
  requiresTechnicalSupport: boolean;
  accessScope: 'hub-only' | 'external' | 'both';
  
  // Location
  institution: string;
  city: string;
  country?: string;
  
  // Content
  description: string;
  functionalities: string[];
  capacityLimitations?: string;
  accessibility?: string;
  availability: string;
  conditionsOfUse: string;
  
  // Tags and categorization
  skillsTags: string[];
  
  // Media
  photos: string[];
  technicalFiles?: FileAttachment[];
  certificates?: FileAttachment[];
  
  // Service-specific fields (for assetType === 'specific-service')
  serviceMethodology?: string;
  serviceDeliveryTime?: string;
  serviceCustomization?: string;
  
  // External API integration
  externalApiId?: string;
  
  // Metadata
  lastUpdated: string;
  featured?: boolean;
}
```

### Content Management Instructions

#### Adding a New Asset

1. **Edit** `/src/app/data/cms-assets.ts`
2. **Add** a new object to the `cmsAssets` array following the schema
3. **Required fields:**
   - Unique `id` and `slug`
   - Name, description, functionalities
   - Asset type, sub type, location
   - At least one photo URL
   - Availability and conditions of use
   - Skills tags (minimum 3)
   - lastUpdated (YYYY-MM-DD)

4. **Example:**
```typescript
{
  id: 'asset-009',
  slug: 'new-laboratory-name',
  name: 'Your Laboratory Name',
  assetType: 'infrastructure-with-support',
  subType: 'laboratory',
  requiresTechnicalSupport: true,
  accessScope: 'both',
  institution: 'Your Institution',
  city: 'Your City',
  description: 'Detailed description...',
  functionalities: [
    'Functionality 1',
    'Functionality 2',
    'Functionality 3'
  ],
  availability: 'Monthly booking slots',
  conditionsOfUse: 'Training required...',
  skillsTags: ['tag1', 'tag2', 'tag3'],
  photos: ['https://url-to-image.jpg'],
  lastUpdated: '2026-01-07'
}
```

#### Updating Asset Information

Simply edit the corresponding object in `cms-assets.ts`. Changes appear immediately.

#### Managing Skills Tags

- Tags auto-populate in filter sidebar
- Use lowercase, hyphenated format: `marine-biotechnology`
- Keep tags specific but not too narrow
- Reuse existing tags when possible for better filtering

---

## 5. Filtering System

### Filter Types

1. **Asset Type**
   - Infrastructure
   - Infrastructure with Mandatory Support
   - Specific Services

2. **Category (Sub Type)**
   - Equipment
   - Laboratory
   - Technical Support
   - Consulting
   - Analysis

3. **Technical Support**
   - All / With Support / Without Support

4. **Access Scope**
   - Hub Azul Only
   - External Users
   - Both

5. **Location**
   - By Institution (auto-populated)
   - By City (auto-populated)

6. **Skills Tags** (multi-select, auto-populated)

7. **Search Query** (searches across name, description, institution, city, skills)

### Filter Behavior

- **AND Logic:** All selected filters must match
- **Within Categories:** OR logic (e.g., multiple cities = any of those cities)
- **Active Filter Chips:** Shows all active filters with remove buttons
- **Result Count:** Real-time update
- **Clear All:** Single button to reset all filters
- **URL Persistence:** Filters saved in URL for sharing

---

## 6. Contact Form

### Fields

1. **Name*** - Required
2. **Entity/Institution*** - Required
3. **Email*** - Required, validated
4. **Service/Infrastructure** - Auto-filled, read-only
5. **Message*** - Required, min 10 characters
6. **Attachments** - Optional, multiple files

### Submission Workflow

```javascript
// Submission object structure
{
  name: "User Name",
  entity: "Organization",
  email: "user@example.com",
  assetSlug: "asset-slug",
  assetName: "Asset Name",
  message: "User message...",
  files: [File, File],
  timestamp: "2026-01-07T12:00:00Z",
  pageUrl: "https://forumoceano.pt/en#/rd-asset-marketplace/assets/asset-slug"
}
```

### Integration Points

**Current (Mock):**
```javascript
console.log('Contact Form Submission:', submission);
```

**Production Integration:**
```javascript
// Option 1: Email via SendGrid/Mailgun
await fetch('/api/send-email', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(submission)
});

// Option 2: CMS (Airtable)
await fetch('https://api.airtable.com/v0/{baseId}/{tableName}', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_API_KEY',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    fields: {
      Name: submission.name,
      Entity: submission.entity,
      Email: submission.email,
      AssetName: submission.assetName,
      Message: submission.message,
      Timestamp: submission.timestamp
    }
  })
});

// Option 3: HubSpot
await fetch('https://api.hubapi.com/crm/v3/objects/contacts', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_HUBSPOT_TOKEN',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    properties: {
      firstname: submission.name,
      email: submission.email,
      company: submission.entity,
      message: submission.message
    }
  })
});
```

---

## 7. External API Integration

### Purpose
Fetch real-time data for each asset (availability status, calibration dates, etc.)

### Implementation

**File:** `/src/app/services/externalApi.ts`

**Features:**
- ✅ Client-side caching (5 minutes)
- ✅ Loading states
- ✅ Graceful fallback on errors
- ✅ Non-blocking (page renders without API)

**Current (Mock):**
```typescript
const MOCK_API_RESPONSES: Record<string, ExternalApiResponse> = {
  'AMRI-TANK-001': {
    assetId: 'AMRI-TANK-001',
    liveAvailability: 'available',
    lastCalibrationDate: '2025-12-15',
    nextMaintenanceDate: '2026-03-01',
    realTimeMetadata: { ... }
  }
};
```

**Production Integration:**
```typescript
export const fetchExternalAssetData = async (
  externalApiId: string
): Promise<ExternalApiResponse | null> => {
  try {
    const response = await fetch(
      `https://api.hubazul.pt/assets/${externalApiId}`,
      {
        headers: {
          'Authorization': `Bearer ${API_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    if (!response.ok) throw new Error('API Error');
    
    const data = await response.json();
    
    // Cache the response
    API_CACHE.set(externalApiId, {
      data,
      timestamp: Date.now()
    });
    
    return data;
  } catch (error) {
    console.error('API Error:', error);
    return null; // Graceful fallback
  }
};
```

### API Response Structure

```typescript
interface ExternalApiResponse {
  assetId: string;
  liveAvailability?: 'available' | 'booked' | 'maintenance';
  lastCalibrationDate?: string;
  nextMaintenanceDate?: string;
  ownerContact?: string;
  realTimeMetadata?: Record<string, any>;
}
```

---

## 8. Deployment

### Static Hosting (GitHub Pages, Netlify, Vercel)

1. **Build:**
   ```bash
   npm run build
   ```

2. **Output:** `/dist` directory

3. **Configure routes:**
   - Single page app
   - Redirect all routes to `index.html`
   - Hash routing handles navigation

4. **Netlify Example:**
   ```toml
   # netlify.toml
   [[redirects]]
     from = "/*"
     to = "/index.html"
     status = 200
   ```

### Integration with Forum Oceano Website

**Option 1: Iframe Embed**
```html
<iframe 
  src="https://marketplace.hubazul.pt/#/rd-asset-marketplace"
  width="100%"
  height="800px"
  frameborder="0"
></iframe>
```

**Option 2: Direct Link**
```html
<a href="https://forumoceano.pt/en#/rd-asset-marketplace">
  Browse R&D Assets
</a>
```

**Option 3: Subdomain**
```
marketplace.forumoceano.pt
```

---

## 9. Phase 2 Scalability

### Architecture Readiness

The codebase is structured for easy Phase 2 expansion:

#### 1. **Reservation System**
- Add `ReservationCalendar` component
- Extend API to support booking endpoints
- Add user authentication

#### 2. **Payment Integration**
- Integrate Stripe/PayPal
- Add payment flow components
- Implement pricing model

#### 3. **User Accounts**
- Add authentication (Auth0, Firebase)
- Create user dashboard
- Implement role-based access

#### 4. **Bilateral Exchange**
- Add credit system
- Transaction history
- Center-to-center transfers

#### 5. **Evaluation System**
- Add review component
- Rating aggregation
- Mandatory post-usage reviews

#### 6. **Contract Templates**
- PDF generation
- E-signature integration
- Contract management

#### 7. **Dashboards**
- Analytics component (already has Recharts)
- Usage statistics
- Revenue tracking

### Database Migration Path

**Current:** Static JSON
**Phase 2:** Backend with Database

Suggested stack:
- **Backend:** Node.js + Express or Supabase
- **Database:** PostgreSQL
- **File Storage:** AWS S3 or Cloudinary
- **Auth:** Auth0 or Supabase Auth

---

## 10. Accessibility

### Implemented Features

- ✅ Keyboard navigation for all filters
- ✅ Focus states on interactive elements
- ✅ Semantic HTML structure
- ✅ ARIA labels where needed
- ✅ Proper heading hierarchy
- ✅ Color contrast compliance (WCAG AA)
- ✅ Responsive for screen readers

### Testing Checklist

- [ ] Tab through entire page
- [ ] Screen reader navigation
- [ ] Zoom to 200%
- [ ] Test on mobile screen readers
- [ ] Keyboard-only form submission

---

## 11. Performance Optimizations

- ✅ Lazy image loading (browser native)
- ✅ Client-side filtering (no server roundtrips)
- ✅ API response caching
- ✅ Responsive images via Unsplash
- ✅ Component code splitting ready
- ✅ Minimal bundle size

### Future Optimizations
- Virtualized list for 100+ assets
- Image CDN integration
- Service worker for offline support

---

## 12. Browser Support

- ✅ Chrome 90+
- ✅ Firefox 88+
- ✅ Safari 14+
- ✅ Edge 90+
- ✅ Mobile Safari (iOS 14+)
- ✅ Chrome Mobile

---

## 13. Testing

### Manual Testing Checklist

**Filtering:**
- [ ] Apply single filter
- [ ] Combine multiple filters
- [ ] Clear individual filters
- [ ] Clear all filters
- [ ] Search query
- [ ] Empty state shows correctly

**Navigation:**
- [ ] Click asset card → detail page
- [ ] Back button returns to catalog with filters intact
- [ ] Share URL with filters → filters load correctly
- [ ] Browser back/forward works

**Contact Form:**
- [ ] Validation errors show
- [ ] Submit with all required fields
- [ ] File upload works
- [ ] Success message displays
- [ ] Form closes after success

**External API:**
- [ ] Live badge shows correctly
- [ ] Loading state displays
- [ ] Error state handles gracefully
- [ ] Data displays on detail page

---

## 14. Common Admin Tasks

### Update Asset Photo
```typescript
// In cms-assets.ts, find the asset and update photos array
photos: [
  'https://new-image-url.jpg',
  'https://another-image.jpg'
]
```

### Add New Skills Tag
```typescript
// Add to any asset's skillsTags array
skillsTags: ['existing-tag', 'new-tag-name']
// Tag automatically appears in filter sidebar
```

### Mark Asset as Featured
```typescript
{
  id: 'asset-001',
  name: '...',
  featured: true,  // Add this line
  // ... rest of fields
}
```

### Change Asset Type
```typescript
// Update both assetType and subType
assetType: 'specific-service', // Changed from 'infrastructure'
subType: 'consulting',          // Changed from 'laboratory'
```

---

## 15. Troubleshooting

### Filters Not Working
- Check URL parameters are updating
- Verify filter state in browser DevTools
- Console should show no errors

### Asset Not Showing
- Verify `slug` is unique
- Check asset is in `cmsAssets` array
- Verify no filtering is excluding it

### External API Data Not Loading
- Check browser console for errors
- Verify `externalApiId` is set
- API service logs to console

### Contact Form Not Submitting
- Check browser console for validation errors
- Verify all required fields filled
- Check email format is valid

---

## 16. Support & Maintenance

### Updating Dependencies
```bash
npm update
npm audit fix
```

### Environment Variables
Currently none required. For production:
- `REACT_APP_API_URL` - External API endpoint
- `REACT_APP_HUBSPOT_KEY` - HubSpot integration
- `REACT_APP_AIRTABLE_KEY` - Airtable integration

---

## 17. Future Enhancements

### Recommended Next Steps

1. **Backend Integration**
   - Move from static JSON to database
   - Implement asset CRUD API
   - Add user authentication

2. **Advanced Search**
   - Fuzzy search
   - Search autocomplete
   - Search history

3. **Notification System**
   - Email notifications for contact requests
   - Asset availability alerts
   - New asset announcements

4. **Analytics**
   - Track popular assets
   - Filter usage statistics
   - Contact form conversion rates

5. **Internationalization**
   - Portuguese/English toggle
   - Multi-language asset content
   - Localized dates/numbers

---

## 18. Contact

For questions or support:
- **Email:** info@forumoceano.pt
- **Website:** https://forumoceano.pt/en
- **GitHub:** [Repository URL]

---

**Document Version:** 1.0  
**Last Updated:** January 7, 2026  
**Author:** Figma Make Development Team
