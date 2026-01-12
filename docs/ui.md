# CostPie UI/UX Design System

## **🍎 Apple-Inspired Theme**
**CostPie uses an Apple-inspired theme for UI layout to achieve a simplified, clean, and intuitive structure. This design philosophy emphasizes minimalism, clarity, and premium user experience with smooth animations, glass morphism effects, and refined shadows.**

## Overview
CostPie follows a modern, clean design system built with React, TypeScript, Tailwind CSS, and shadcn/ui components. The interface emphasizes clarity, usability, and visual hierarchy with a sophisticated gradient-based color scheme, following Apple's design principles of simplicity and elegance.

## Technology Stack
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS with custom configuration
- **Component Library**: shadcn/ui (Radix UI primitives)
- **Icons**: Lucide React
- **Animations**: Tailwind CSS animations + class-variance-authority
- **Charts**: Recharts
- **State Management**: React Query (TanStack Query)
- **Routing**: React Router v6

## Component Standards

### ONLY shadcn/ui Components
- **ABSOLUTELY NO custom components should be created**
- Use ONLY shadcn/ui components for all UI elements
- All components must come from the shadcn/ui library
- No exceptions to this rule
- Components are located in `/src/components/ui/`

### Available shadcn/ui Components
- accordion, alert, alert-dialog, aspect-ratio, avatar
- badge, breadcrumb, button, calendar, card, carousel
- chart, checkbox, collapsible, command, context-menu
- dialog, drawer, dropdown-menu, error-boundary, form
- hover-card, input, input-otp, label, menubar
- navigation-menu, pagination, popover, progress, radio-group
- resizable, scroll-area, select, separator, sheet
- sidebar, skeleton, slider, sonner, switch
- table, tabs, textarea, toast, toaster
- toggle, toggle-group, tooltip

## Color System

### Primary Brand Colors
- **Emerald/Green Gradient**: Primary brand color used for active states, CTAs, and success indicators
  - `from-emerald-500 via-green-500 to-teal-500` (gradient)
  - `emerald-500` (#10b981)
  - `green-500` (#22c55e)
  - `teal-500` (#14b8a6)

### Secondary Colors
- **Blue**: Information, links, and secondary actions
  - `blue-500` (#3b82f6)
  - `blue-600` (#2563eb)
  - `indigo-600` (#4f46e5)
  
- **Red/Rose**: Errors, destructive actions, notifications
  - `red-500` (#ef4444)
  - `rose-500` (#f43f5e)
  
- **Purple**: User avatars, dark mode toggle
  - `purple-600` (#9333ea)
  
- **Amber/Yellow**: Warnings, highlights
  - `amber-500` (#f59e0b)
  - `yellow-500` (#eab308)

### Neutral Colors
- **Gray Scale**: Base UI elements, text, borders
  - `gray-50` (#f9fafb) - Background
  - `gray-100` (#f3f4f6) - Light backgrounds
  - `gray-200` (#e5e7eb) - Borders
  - `gray-400` (#9ca3af) - Muted icons
  - `gray-500` (#6b7280) - Secondary text
  - `gray-600` (#4b5563) - Regular text
  - `gray-900` (#111827) - Primary text

### CSS Variables (Light Theme)
```css
--background: 0 0% 100%;
--foreground: 222.2 84% 4.9%;
--primary: 222.2 47.4% 11.2%;
--secondary: 210 40% 96.1%;
--muted: 210 40% 96.1%;
--accent: 210 40% 96.1%;
--destructive: 0 84.2% 60.2%;
--border: 214.3 31.8% 91.4%;
--ring: 222.2 84% 4.9%;
```

## Typography

### Font System
- **Font Family**: System font stack (defined by Tailwind defaults)
- **Font Sizes**:
  - `text-xs`: 12px - Labels, metadata, badges
  - `text-sm`: 14px - Secondary text, descriptions
  - `text-base`: 16px - Body text
  - `text-lg`: 18px - Section headers
  - `text-xl`: 20px - Page titles
  - `text-2xl`: 24px - Card values
  - `text-3xl`: 30px - Large metrics

### Font Weights
- `font-medium`: 500 - Headers, labels
- `font-semibold`: 600 - Important text
- `font-bold`: 700 - Metrics, emphasis

## Spacing System

### Standard Spacing Scale
- `p-2`: 8px
- `p-3`: 12px
- `p-4`: 16px
- `p-6`: 24px
- `gap-2`: 8px
- `gap-3`: 12px
- `gap-4`: 16px
- `gap-5`: 20px
- `gap-6`: 24px

### Common Patterns
- **Page Padding**: `p-6` (24px)
- **Card Padding**: `p-4` (16px)
- **Button Padding**: `px-4 py-2` (16px horizontal, 8px vertical)
- **Section Margins**: `mb-6` (24px bottom margin)

## Layout Structure

### Page Layout
```tsx
<div className="flex h-screen bg-gray-50">
  <Sidebar /> {/* Collapsible sidebar with navigation */}
  <div className="flex-1 flex flex-col overflow-hidden">
    <Header /> {/* Fixed header with actions */}
    <div className="flex-1 overflow-y-auto p-6">
      <div className="max-w-7xl mx-auto">
        {/* Page content */}
      </div>
    </div>
  </div>
  <SupportChat /> {/* Optional floating chat */}
</div>
```

### Grid System
- **Dashboard Grid**: `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-5`
- **Responsive Breakpoints**:
  - Mobile: Default (full width)
  - Tablet: `md:` (768px+)
  - Desktop: `lg:` (1024px+)
  - Wide: `xl:` (1280px+)

## Component Patterns

### 1. Sidebar Component
```tsx
<div className="w-64 bg-white border-r border-gray-200 h-screen flex flex-col transition-all duration-300 shadow-lg">
  {/* Collapsible with animation */}
  {/* Gradient header: from-emerald-50 to-teal-50 */}
  {/* Active state: bg-gradient-to-r from-emerald-500 via-green-500 to-teal-500 */}
  {/* Hover state: hover:bg-emerald-50 hover:text-emerald-700 hover:scale-105 */}
</div>
```

Key Features:
- Collapsible with smooth transitions
- Gradient active states
- Icon + text navigation items
- User avatar dropdown at bottom
- Support & billing section

### 2. Header Component
```tsx
<div className="h-16 bg-white flex items-center justify-between px-6 relative">
  {/* Gradient border line at bottom */}
  {/* Icon buttons with colored hover states */}
  {/* Notification badge with count */}
</div>
```

Key Features:
- Fixed height (64px)
- Action buttons with tooltip
- Theme toggle (light/dark)
- Notification center
- Tutorial/help button

### 3. Stats Cards
```tsx
import { Card, CardContent } from "@/components/ui/card";
import { HoverCard, HoverCardTrigger, HoverCardContent } from "@/components/ui/hover-card";
import { InfoIcon } from "lucide-react";

<div className="bg-white rounded-md border border-gray-200 p-4 flex flex-col">
  <div className="flex justify-between items-start">
    <h3 className="text-sm font-medium text-gray-600 flex items-center">
      {title}
      {tooltip && (
        <HoverCard>
          <HoverCardTrigger asChild>
            <InfoIcon className="ml-1 h-4 w-4 text-gray-400 cursor-help" />
          </HoverCardTrigger>
          <HoverCardContent className="text-xs">
            {tooltip}
          </HoverCardContent>
        </HoverCard>
      )}
    </h3>
    {icon && <div>{icon}</div>}
  </div>
  <div className="mt-2 flex-1 flex items-center justify-between">
    <div className="text-2xl font-bold">{value}</div>
  </div>
  {description && (
    <div className="mt-2 text-xs">{description}</div>
  )}
</div>
```

Variants:
- Default: White background
- Success: `bg-green-500 text-white`
- Info: `bg-blue-500 text-white`
- Warning: Custom gradient backgrounds

### 4. Button Component
```tsx
import { Button } from "@/components/ui/button";

// Primary Button
<Button className="bg-blue-500 hover:bg-blue-600">

// Outline Button
<Button variant="outline" className="border-gray-200">

// Icon Button
<Button variant="outline" size="icon" className="h-9 w-9">

// Ghost Button
<Button variant="ghost">
```

Button Sizes:
- `sm`: h-9 px-3
- `default`: h-10 px-4 py-2
- `lg`: h-11 px-8
- `icon`: h-10 w-10

### 5. Input Fields
```tsx
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

<div className="space-y-2">
  <Label>Field Label</Label>
  <Input className="h-10 rounded-md border-input bg-background px-3 py-2" />
  <p className="text-sm text-muted-foreground">Helper text</p>
</div>
```

States:
- Default: `border-gray-200`
- Focus: `focus-visible:ring-2 focus-visible:ring-ring`
- Disabled: `disabled:opacity-50 disabled:cursor-not-allowed`
- Error: `border-red-500`

### 6. Cards
```tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";

<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent className="p-4">
    {/* Content */}
  </CardContent>
</Card>
```

Styling:
- `rounded-lg border bg-card shadow-sm`
- Header: `p-6`
- Content: `p-6 pt-0` or custom

### 7. Tables
```tsx
import { Table, TableHeader, TableBody, TableHead, TableRow, TableCell } from "@/components/ui/table";

<Table>
  <TableHeader className="bg-gray-50">
    <TableRow>
      <TableHead>Column</TableHead>
    </TableRow>
  </TableHeader>
  <TableBody>
    <TableRow className="hover:bg-gray-50">
      <TableCell>Data</TableCell>
    </TableRow>
  </TableBody>
</Table>
```

Features:
- Alternating row colors option
- Hover states
- Sortable columns
- Action buttons in rows

### 8. Modals/Dialogs
```tsx
import { Dialog, DialogContent, DialogDescription, DialogFooter, DialogHeader, DialogTitle, DialogTrigger } from "@/components/ui/dialog";

<Dialog>
  <DialogTrigger>Open</DialogTrigger>
  <DialogContent className="sm:max-w-[425px]">
    <DialogHeader>
      <DialogTitle>Title</DialogTitle>
      <DialogDescription>Description</DialogDescription>
    </DialogHeader>
    {/* Content */}
    <DialogFooter>
      <Button>Action</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

### 9. Alerts & Toasts
```tsx
import { Alert, AlertDescription, AlertTitle } from "@/components/ui/alert";
import { useToast } from "@/components/ui/use-toast";

// Alert Component
<Alert className="border-l-4 border-blue-500">
  <AlertTitle>Title</AlertTitle>
  <AlertDescription>Message</AlertDescription>
</Alert>

// Toast Notification
const { toast } = useToast();
toast({
  title: "Success",
  description: "Action completed",
  variant: "default" // or "destructive"
})
```

### 10. Badges
```tsx
import { Badge } from "@/components/ui/badge";

<Badge variant="default" className="bg-emerald-100 text-emerald-700">
  Status
</Badge>
```

Variants:
- Success: `bg-emerald-100 text-emerald-700`
- Warning: `bg-amber-100 text-amber-700`
- Error: `bg-red-100 text-red-700`
- Info: `bg-blue-100 text-blue-700`

## Date Formatting Standards

### Library
- Use `date-fns` for all date formatting operations

### Format Specification
Dates must be formatted using ordinal indicators as follows:
- 1st Sep 2025
- 2nd Aug 2025
- 3rd Jan 2026
- 4th Jun 2024

### Format Pattern
- Day with ordinal suffix (1st, 2nd, 3rd, 4th, etc.)
- Abbreviated month name (3 letters)
- Full year (4 digits)

### Implementation
```tsx
import { format } from 'date-fns';

// Custom format function from lib/utils
export const formatDate = (date: Date | string): string => {
  const d = typeof date === 'string' ? new Date(date) : date;
  const day = d.getDate();
  const suffix = getDaySuffix(day);
  return `${day}${suffix} ${format(d, 'MMM yyyy')}`;
};
```

## Animation Patterns

### Transitions
- **Duration**: `transition-all duration-300` (standard)
- **Fast**: `duration-200`
- **Slow**: `duration-500`

### Hover Effects
- Scale: `hover:scale-105`
- Shadow: `hover:shadow-lg`
- Color: `hover:bg-emerald-50`
- Combined: `hover:scale-105 hover:shadow-lg`

### Loading States
```tsx
import { Loader2 } from 'lucide-react';
<Loader2 className="h-8 w-8 animate-spin text-primary" />
```

### Skeleton Loading
```tsx
import { Skeleton } from "@/components/ui/skeleton";
<Skeleton className="h-4 w-[250px]" />
```

## Icons Usage

### Icon Library: Lucide React
Common icons and their usage:
- `LayoutGrid`: Dashboard
- `HeartPulse`: Health/Monitoring
- `BarChart3`: Analytics
- `Box`: Resources
- `Network`: Architecture
- `Settings`: Settings
- `User`: Profile
- `Bell`: Notifications
- `ChevronRight/Left`: Navigation
- `TrendingUp/Down`: Trends
- `Plus/PlusCircle`: Add actions
- `Trash2`: Delete
- `Pencil/Edit`: Edit
- `Search`: Search
- `RefreshCw`: Refresh/Sync
- `Loader2`: Loading spinner
- `Info`: Information/Tooltip
- `Building`: Organization
- `CreditCard`: Billing
- `FileText`: Documentation
- `MessageCircle`: Support/Chat

### Icon Sizes
- Small: `size={16}`
- Default: `size={18}` or `size={20}`
- Large: `size={24}`

### Icon Colors
- Default: Inherit from text color
- Muted: `text-gray-400`
- Active: `text-white` (in colored backgrounds)
- Colored: Match theme colors (e.g., `text-blue-600`)

## Responsive Design

### Breakpoints
- `sm:` 640px - Small devices
- `md:` 768px - Tablets
- `lg:` 1024px - Desktop
- `xl:` 1280px - Large screens
- `2xl:` 1536px - Extra large

### Mobile-First Approach
```tsx
// Stack on mobile, grid on larger screens
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4">

// Hide on mobile, show on desktop
<div className="hidden lg:block">

// Different padding by screen size
<div className="p-4 md:p-6 lg:p-8">
```

## Dark Mode Support

### Theme Toggle Implementation
- CSS variables switch between light/dark values
- Components use semantic color tokens
- Smooth transitions between themes
- System preference detection

### Dark Mode Classes
```tsx
// Component that adapts to theme
<div className="bg-background text-foreground">
  {/* Automatically switches colors */}
</div>

// Explicit dark mode styling
<div className="bg-white dark:bg-gray-900">
  <p className="text-gray-900 dark:text-gray-100">
```

## Form Patterns

### Form Layout
```tsx
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Button } from "@/components/ui/button";

<form className="space-y-4">
  <div className="space-y-2">
    <Label>Field Label</Label>
    <Input />
    <p className="text-sm text-muted-foreground">Helper text</p>
  </div>
  
  <div className="flex gap-2">
    <Button type="submit">Submit</Button>
    <Button variant="outline" type="button">Cancel</Button>
  </div>
</form>
```

### Validation States
- Error: Red border + error message
- Success: Green check icon
- Warning: Amber background
- Info: Blue helper text

## Chart Styling

### Chart Configuration (Recharts)
```tsx
{
  colors: {
    primary: '#10b981', // Emerald
    secondary: '#3b82f6', // Blue
    tertiary: '#f59e0b', // Amber
  },
  grid: {
    stroke: '#e5e7eb', // gray-200
  },
  axis: {
    style: {
      fontSize: 12,
      fill: '#6b7280', // gray-500
    }
  }
}
```

## Best Practices

### 1. Consistency
- Use predefined color tokens
- Maintain consistent spacing
- Follow established patterns
- Always use shadcn/ui components

### 2. Accessibility
- Proper contrast ratios
- Focus visible states
- ARIA labels where needed
- Keyboard navigation support

### 3. Performance
- Lazy load heavy components
- Use React.memo for expensive renders
- Optimize images and assets
- Code splitting for routes

### 4. User Experience
- Clear visual hierarchy
- Intuitive navigation
- Responsive feedback
- Loading states for async operations
- Error boundaries for graceful failures

### 5. Code Organization
- Component composition
- Reusable utility functions
- Consistent file structure
- Type safety with TypeScript

## Complete Page Template

```tsx
import React, { useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { Badge } from '@/components/ui/badge';
import { useToast } from '@/components/ui/use-toast';
import Sidebar from '@/components/Sidebar';
import Header from '@/components/Header';
import { Search, Plus, Loader2 } from 'lucide-react';

const ExamplePage = () => {
  const [loading, setLoading] = useState(false);
  const { toast } = useToast();
  
  return (
    <div className="flex h-screen bg-gray-50">
      <Sidebar />
      
      <div className="flex-1 flex flex-col overflow-hidden">
        <Header />
        
        <div className="flex-1 overflow-y-auto p-6">
          <div className="max-w-7xl mx-auto">
            {/* Page Title and Actions */}
            <div className="flex justify-between items-center mb-6">
              <h1 className="text-2xl font-semibold">Page Title</h1>
              <Button className="flex items-center gap-2 h-9 bg-blue-500 hover:bg-blue-600">
                <Plus size={16} />
                <span className="text-sm">Add New</span>
              </Button>
            </div>
            
            {/* Stats Grid */}
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-5 mb-6">
              {/* Stats cards here */}
            </div>
            
            {/* Search and Filters */}
            <Card className="mb-6">
              <CardContent className="p-4">
                <div className="flex gap-4">
                  <div className="flex-1">
                    <Input
                      placeholder="Search..."
                      className="max-w-sm"
                      prefix={<Search size={16} />}
                    />
                  </div>
                  <Button variant="outline">Filter</Button>
                </div>
              </CardContent>
            </Card>
            
            {/* Data Table */}
            <Card>
              <CardHeader>
                <CardTitle>Data Table</CardTitle>
              </CardHeader>
              <CardContent>
                {loading ? (
                  <div className="flex items-center justify-center py-8">
                    <Loader2 className="h-8 w-8 animate-spin text-primary" />
                  </div>
                ) : (
                  <Table>
                    <TableHeader>
                      <TableRow>
                        <TableHead>Column 1</TableHead>
                        <TableHead>Column 2</TableHead>
                        <TableHead>Status</TableHead>
                        <TableHead className="text-right">Actions</TableHead>
                      </TableRow>
                    </TableHeader>
                    <TableBody>
                      <TableRow>
                        <TableCell>Data 1</TableCell>
                        <TableCell>Data 2</TableCell>
                        <TableCell>
                          <Badge className="bg-emerald-100 text-emerald-700">
                            Active
                          </Badge>
                        </TableCell>
                        <TableCell className="text-right">
                          <Button variant="ghost" size="sm">Edit</Button>
                        </TableCell>
                      </TableRow>
                    </TableBody>
                  </Table>
                )}
              </CardContent>
            </Card>
          </div>
        </div>
      </div>
    </div>
  );
};

export default ExamplePage;
```

This comprehensive design system ensures consistency, maintainability, and a professional user experience across the entire CostPie application.