# UI/UX Design System & Style Guide
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0  
**Date:** December 2024  
**Document Type:** Design System Specification  
**Scope:** Complete System UI/UX Standards

---

## Executive Summary

This document establishes the comprehensive UI/UX design system for the CleanStation digitalization project, ensuring consistent user experience across all interfaces, roles, and devices. The design system prioritizes usability in industrial environments while maintaining professional aesthetics and regulatory compliance requirements.

### Key Components
- **Design Principles** - Core philosophy and user-centered approach
- **Visual Identity** - Colors, typography, iconography, and branding
- **Component Library** - Reusable UI components and patterns
- **Responsive Design** - Multi-device and screen size adaptations
- **Accessibility Standards** - WCAG 2.1 compliance and inclusive design
- **Interaction Patterns** - User flows and behavior guidelines

### Success Criteria
- Consistent user experience across all system interfaces
- Efficient task completion for all user roles
- Accessibility compliance for diverse user needs
- Mobile-optimized interfaces for shop floor use
- Professional appearance supporting regulatory compliance

---

## Design Principles

### 1. User-Centered Design
```typescript
interface DesignPrinciples {
  primaryPrinciple: 'TaskEfficiency';
  userFocus: 'ProductionWorkforce';
  environment: 'IndustrialShopFloor';
  accessibility: 'UniversalAccess';
  compliance: 'MedicalDeviceStandards';
}
```

**Core Principles:**
1. **Efficiency First** - Minimize clicks and cognitive load for repetitive tasks
2. **Context Awareness** - Interfaces adapt to user role and current workflow stage
3. **Error Prevention** - Proactive validation and clear feedback systems
4. **Progressive Disclosure** - Show relevant information at the right time
5. **Accessibility by Design** - Inclusive interfaces for all users and abilities

### 2. Industrial Environment Considerations
- **High Contrast** - Readable in various lighting conditions
- **Touch-Friendly** - Optimized for gloved hands and industrial tablets
- **Durability Focus** - Interfaces that work on ruggedized devices
- **Noise Environments** - Visual feedback over audio cues
- **Time Pressure** - Quick access to critical functions

### 3. Regulatory Compliance Design
- **Audit Trail Visibility** - Clear tracking of all user actions
- **Data Integrity** - Visual confirmation of data accuracy
- **Role-Based Access** - Clear indication of user permissions
- **Document Traceability** - Easy access to supporting documentation
- **Quality Focus** - Emphasis on quality checkpoints and validation

---

## Visual Identity

### Color Palette

#### Primary Colors
```css
:root {
  /* Torvan Medical Brand Colors */
  --primary-50: #f0f9ff;
  --primary-100: #e0f2fe;
  --primary-200: #bae6fd;
  --primary-300: #7dd3fc;
  --primary-400: #38bdf8;
  --primary-500: #0ea5e9;  /* Primary brand blue */
  --primary-600: #0284c7;
  --primary-700: #0369a1;
  --primary-800: #075985;
  --primary-900: #0c4a6e;
  --primary-950: #082f49;
}
```

#### Semantic Colors
```css
:root {
  /* Success Colors - for approvals, completed tasks */
  --success-50: #f0fdf4;
  --success-500: #22c55e;
  --success-600: #16a34a;
  --success-700: #15803d;
  
  /* Warning Colors - for attention, pending items */
  --warning-50: #fffbeb;
  --warning-500: #f59e0b;
  --warning-600: #d97706;
  --warning-700: #b45309;
  
  /* Error Colors - for failures, rejections */
  --error-50: #fef2f2;
  --error-500: #ef4444;
  --error-600: #dc2626;
  --error-700: #b91c1c;
  
  /* Info Colors - for information, help */
  --info-50: #eff6ff;
  --info-500: #3b82f6;
  --info-600: #2563eb;
  --info-700: #1d4ed8;
}
```

#### Neutral Palette
```css
:root {
  /* Neutral Colors - for backgrounds, text, borders */
  --neutral-50: #f9fafb;
  --neutral-100: #f3f4f6;
  --neutral-200: #e5e7eb;
  --neutral-300: #d1d5db;
  --neutral-400: #9ca3af;
  --neutral-500: #6b7280;
  --neutral-600: #4b5563;
  --neutral-700: #374151;
  --neutral-800: #1f2937;
  --neutral-900: #111827;
  --neutral-950: #030712;
}
```

#### Status Colors for Order Workflow
```css
:root {
  /* Order Status Colors */
  --status-created: #3b82f6;      /* Blue */
  --status-parts-sent: #f59e0b;   /* Amber */
  --status-pre-qc: #8b5cf6;       /* Purple */
  --status-production: #06b6d4;   /* Cyan */
  --status-testing: #ec4899;      /* Pink */
  --status-packaging: #84cc16;    /* Lime */
  --status-final-qc: #f97316;     /* Orange */
  --status-ready-ship: #22c55e;   /* Green */
  --status-shipped: #64748b;      /* Slate */
}
```

### Typography

#### Font System
```css
/* Primary font stack */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

:root {
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
  --font-mono: 'SF Mono', Monaco, 'Cascadia Code', 'Roboto Mono', Consolas, monospace;
}

/* Font sizes following a modular scale */
:root {
  --text-xs: 0.75rem;     /* 12px */
  --text-sm: 0.875rem;    /* 14px */
  --text-base: 1rem;      /* 16px */
  --text-lg: 1.125rem;    /* 18px */
  --text-xl: 1.25rem;     /* 20px */
  --text-2xl: 1.5rem;     /* 24px */
  --text-3xl: 1.875rem;   /* 30px */
  --text-4xl: 2.25rem;    /* 36px */
  --text-5xl: 3rem;       /* 48px */
}

/* Font weights */
:root {
  --font-light: 300;
  --font-normal: 400;
  --font-medium: 500;
  --font-semibold: 600;
  --font-bold: 700;
}

/* Line heights */
:root {
  --leading-tight: 1.25;
  --leading-snug: 1.375;
  --leading-normal: 1.5;
  --leading-relaxed: 1.625;
  --leading-loose: 2;
}
```

#### Typography Scale Application
```css
/* Heading styles */
.heading-1 {
  font-size: var(--text-4xl);
  font-weight: var(--font-bold);
  line-height: var(--leading-tight);
  letter-spacing: -0.025em;
}

.heading-2 {
  font-size: var(--text-3xl);
  font-weight: var(--font-semibold);
  line-height: var(--leading-tight);
}

.heading-3 {
  font-size: var(--text-2xl);
  font-weight: var(--font-semibold);
  line-height: var(--leading-snug);
}

.heading-4 {
  font-size: var(--text-xl);
  font-weight: var(--font-medium);
  line-height: var(--leading-snug);
}

/* Body text styles */
.body-large {
  font-size: var(--text-lg);
  font-weight: var(--font-normal);
  line-height: var(--leading-relaxed);
}

.body-base {
  font-size: var(--text-base);
  font-weight: var(--font-normal);
  line-height: var(--leading-normal);
}

.body-small {
  font-size: var(--text-sm);
  font-weight: var(--font-normal);
  line-height: var(--leading-normal);
}

/* Utility text styles */
.text-caption {
  font-size: var(--text-xs);
  font-weight: var(--font-medium);
  line-height: var(--leading-normal);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.text-code {
  font-family: var(--font-mono);
  font-size: var(--text-sm);
  background: var(--neutral-100);
  padding: 0.125rem 0.25rem;
  border-radius: 0.25rem;
}
```

### Spacing System

#### Spacing Scale
```css
:root {
  /* Spacing scale based on 0.25rem (4px) base unit */
  --space-0: 0;
  --space-px: 1px;
  --space-0-5: 0.125rem;  /* 2px */
  --space-1: 0.25rem;     /* 4px */
  --space-1-5: 0.375rem;  /* 6px */
  --space-2: 0.5rem;      /* 8px */
  --space-2-5: 0.625rem;  /* 10px */
  --space-3: 0.75rem;     /* 12px */
  --space-3-5: 0.875rem;  /* 14px */
  --space-4: 1rem;        /* 16px */
  --space-5: 1.25rem;     /* 20px */
  --space-6: 1.5rem;      /* 24px */
  --space-7: 1.75rem;     /* 28px */
  --space-8: 2rem;        /* 32px */
  --space-9: 2.25rem;     /* 36px */
  --space-10: 2.5rem;     /* 40px */
  --space-11: 2.75rem;    /* 44px */
  --space-12: 3rem;       /* 48px */
  --space-14: 3.5rem;     /* 56px */
  --space-16: 4rem;       /* 64px */
  --space-20: 5rem;       /* 80px */
  --space-24: 6rem;       /* 96px */
  --space-32: 8rem;       /* 128px */
}
```

#### Layout Spacing Guidelines
```css
/* Component spacing */
.component-padding-sm { padding: var(--space-3); }
.component-padding-md { padding: var(--space-4); }
.component-padding-lg { padding: var(--space-6); }
.component-padding-xl { padding: var(--space-8); }

/* Stack spacing (vertical) */
.stack-xs > * + * { margin-top: var(--space-2); }
.stack-sm > * + * { margin-top: var(--space-3); }
.stack-md > * + * { margin-top: var(--space-4); }
.stack-lg > * + * { margin-top: var(--space-6); }
.stack-xl > * + * { margin-top: var(--space-8); }

/* Inline spacing (horizontal) */
.inline-xs > * + * { margin-left: var(--space-2); }
.inline-sm > * + * { margin-left: var(--space-3); }
.inline-md > * + * { margin-left: var(--space-4); }
.inline-lg > * + * { margin-left: var(--space-6); }
```

---

## Component Library

### 1. Button Components

#### Button Variants
```tsx
// Primary button for main actions
const ButtonPrimary = styled.button`
  background: var(--primary-500);
  color: white;
  border: none;
  border-radius: 0.5rem;
  padding: var(--space-3) var(--space-6);
  font-size: var(--text-base);
  font-weight: var(--font-medium);
  cursor: pointer;
  transition: all 0.2s ease;
  
  &:hover:not(:disabled) {
    background: var(--primary-600);
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgb(14 165 233 / 0.4);
  }
  
  &:active {
    transform: translateY(0);
  }
  
  &:disabled {
    background: var(--neutral-300);
    cursor: not-allowed;
  }
  
  &:focus-visible {
    outline: 2px solid var(--primary-500);
    outline-offset: 2px;
  }
`;

// Secondary button for secondary actions
const ButtonSecondary = styled.button`
  background: white;
  color: var(--primary-600);
  border: 1px solid var(--primary-300);
  border-radius: 0.5rem;
  padding: var(--space-3) var(--space-6);
  font-size: var(--text-base);
  font-weight: var(--font-medium);
  cursor: pointer;
  transition: all 0.2s ease;
  
  &:hover:not(:disabled) {
    background: var(--primary-50);
    border-color: var(--primary-400);
  }
  
  &:disabled {
    color: var(--neutral-400);
    border-color: var(--neutral-200);
    cursor: not-allowed;
  }
`;

// Danger button for destructive actions
const ButtonDanger = styled.button`
  background: var(--error-500);
  color: white;
  border: none;
  border-radius: 0.5rem;
  padding: var(--space-3) var(--space-6);
  font-size: var(--text-base);
  font-weight: var(--font-medium);
  cursor: pointer;
  transition: all 0.2s ease;
  
  &:hover:not(:disabled) {
    background: var(--error-600);
  }
`;
```

#### Button Sizes
```tsx
interface ButtonProps {
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
  variant?: 'primary' | 'secondary' | 'danger' | 'ghost';
  children: React.ReactNode;
  disabled?: boolean;
  loading?: boolean;
}

const buttonSizes = {
  xs: {
    padding: 'var(--space-2) var(--space-3)',
    fontSize: 'var(--text-xs)',
    height: '2rem'
  },
  sm: {
    padding: 'var(--space-2) var(--space-4)',
    fontSize: 'var(--text-sm)',
    height: '2.25rem'
  },
  md: {
    padding: 'var(--space-3) var(--space-6)',
    fontSize: 'var(--text-base)',
    height: '2.5rem'
  },
  lg: {
    padding: 'var(--space-3) var(--space-8)',
    fontSize: 'var(--text-lg)',
    height: '3rem'
  },
  xl: {
    padding: 'var(--space-4) var(--space-10)',
    fontSize: 'var(--text-xl)',
    height: '3.5rem'
  }
};
```

### 2. Form Components

#### Input Fields
```tsx
const InputField = styled.input`
  width: 100%;
  padding: var(--space-3);
  border: 1px solid var(--neutral-300);
  border-radius: 0.5rem;
  font-size: var(--text-base);
  background: white;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
  
  &:focus {
    outline: none;
    border-color: var(--primary-500);
    box-shadow: 0 0 0 3px rgb(14 165 233 / 0.1);
  }
  
  &:invalid {
    border-color: var(--error-500);
  }
  
  &:disabled {
    background: var(--neutral-50);
    color: var(--neutral-500);
    cursor: not-allowed;
  }
  
  /* Touch device optimization */
  @media (pointer: coarse) {
    min-height: 44px; /* Touch target size */
    font-size: 16px; /* Prevent zoom on iOS */
  }
`;

const FormGroup = styled.div`
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
`;

const FormLabel = styled.label`
  font-size: var(--text-sm);
  font-weight: var(--font-medium);
  color: var(--neutral-700);
`;

const FormError = styled.span`
  font-size: var(--text-xs);
  color: var(--error-600);
  display: flex;
  align-items: center;
  gap: var(--space-1);
`;

const FormHelp = styled.span`
  font-size: var(--text-xs);
  color: var(--neutral-500);
`;
```

#### Select Components
```tsx
const SelectField = styled.select`
  width: 100%;
  padding: var(--space-3);
  border: 1px solid var(--neutral-300);
  border-radius: 0.5rem;
  font-size: var(--text-base);
  background: white;
  cursor: pointer;
  
  &:focus {
    outline: none;
    border-color: var(--primary-500);
    box-shadow: 0 0 0 3px rgb(14 165 233 / 0.1);
  }
  
  /* Custom dropdown arrow */
  appearance: none;
  background-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3E%3Cpath stroke='%236b7280' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='M6 8l4 4 4-4'/%3E%3C/svg%3E");
  background-position: right var(--space-3) center;
  background-repeat: no-repeat;
  background-size: 1rem;
  padding-right: var(--space-10);
`;
```

### 3. Data Display Components

#### Card Component
```tsx
const Card = styled.div`
  background: white;
  border: 1px solid var(--neutral-200);
  border-radius: 0.75rem;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
  overflow: hidden;
  transition: box-shadow 0.2s ease;
  
  &:hover {
    box-shadow: 0 4px 6px rgb(0 0 0 / 0.1);
  }
`;

const CardHeader = styled.div`
  padding: var(--space-6);
  border-bottom: 1px solid var(--neutral-200);
`;

const CardTitle = styled.h3`
  font-size: var(--text-lg);
  font-weight: var(--font-semibold);
  color: var(--neutral-900);
  margin: 0;
`;

const CardDescription = styled.p`
  font-size: var(--text-sm);
  color: var(--neutral-600);
  margin: var(--space-1) 0 0 0;
`;

const CardContent = styled.div`
  padding: var(--space-6);
`;

const CardFooter = styled.div`
  padding: var(--space-6);
  border-top: 1px solid var(--neutral-200);
  background: var(--neutral-50);
`;
```

#### Badge Component
```tsx
interface BadgeProps {
  variant?: 'default' | 'success' | 'warning' | 'error' | 'info';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
}

const Badge = styled.span<BadgeProps>`
  display: inline-flex;
  align-items: center;
  border-radius: 9999px;
  font-weight: var(--font-medium);
  
  ${props => {
    const sizes = {
      sm: {
        padding: 'var(--space-1) var(--space-2)',
        fontSize: 'var(--text-xs)'
      },
      md: {
        padding: 'var(--space-1) var(--space-3)',
        fontSize: 'var(--text-sm)'
      },
      lg: {
        padding: 'var(--space-2) var(--space-4)',
        fontSize: 'var(--text-base)'
      }
    };
    
    const variants = {
      default: {
        background: 'var(--neutral-100)',
        color: 'var(--neutral-800)'
      },
      success: {
        background: 'var(--success-100)',
        color: 'var(--success-800)'
      },
      warning: {
        background: 'var(--warning-100)',
        color: 'var(--warning-800)'
      },
      error: {
        background: 'var(--error-100)',
        color: 'var(--error-800)'
      },
      info: {
        background: 'var(--info-100)',
        color: 'var(--info-800)'
      }
    };
    
    const size = sizes[props.size || 'md'];
    const variant = variants[props.variant || 'default'];
    
    return `
      padding: ${size.padding};
      font-size: ${size.fontSize};
      background: ${variant.background};
      color: ${variant.color};
    `;
  }}
`;
```

### 4. Navigation Components

#### Sidebar Navigation
```tsx
const Sidebar = styled.nav`
  width: 16rem;
  height: 100vh;
  background: white;
  border-right: 1px solid var(--neutral-200);
  display: flex;
  flex-direction: column;
  position: fixed;
  left: 0;
  top: 0;
  z-index: 40;
  
  @media (max-width: 768px) {
    transform: translateX(-100%);
    transition: transform 0.3s ease;
    
    &.open {
      transform: translateX(0);
    }
  }
`;

const SidebarHeader = styled.div`
  padding: var(--space-6);
  border-bottom: 1px solid var(--neutral-200);
`;

const SidebarContent = styled.div`
  flex: 1;
  overflow-y: auto;
  padding: var(--space-4);
`;

const SidebarFooter = styled.div`
  padding: var(--space-4);
  border-top: 1px solid var(--neutral-200);
`;

const NavItem = styled.a`
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3);
  border-radius: 0.5rem;
  color: var(--neutral-700);
  text-decoration: none;
  font-weight: var(--font-medium);
  transition: all 0.2s ease;
  
  &:hover {
    background: var(--neutral-100);
    color: var(--neutral-900);
  }
  
  &.active {
    background: var(--primary-100);
    color: var(--primary-700);
  }
  
  svg {
    width: 1.25rem;
    height: 1.25rem;
  }
`;
```

#### Breadcrumb Navigation
```tsx
const Breadcrumb = styled.nav`
  display: flex;
  align-items: center;
  gap: var(--space-2);
  margin-bottom: var(--space-4);
`;

const BreadcrumbItem = styled.a`
  color: var(--neutral-600);
  text-decoration: none;
  font-size: var(--text-sm);
  
  &:hover {
    color: var(--primary-600);
  }
  
  &:last-child {
    color: var(--neutral-900);
    font-weight: var(--font-medium);
  }
`;

const BreadcrumbSeparator = styled.span`
  color: var(--neutral-400);
  font-size: var(--text-sm);
`;
```

### 5. Feedback Components

#### Alert Component
```tsx
interface AlertProps {
  variant?: 'info' | 'success' | 'warning' | 'error';
  title?: string;
  children: React.ReactNode;
  onClose?: () => void;
}

const Alert = styled.div<AlertProps>`
  border-radius: 0.5rem;
  padding: var(--space-4);
  border-left: 4px solid;
  
  ${props => {
    const variants = {
      info: {
        background: 'var(--info-50)',
        borderColor: 'var(--info-500)',
        titleColor: 'var(--info-800)',
        textColor: 'var(--info-700)'
      },
      success: {
        background: 'var(--success-50)',
        borderColor: 'var(--success-500)',
        titleColor: 'var(--success-800)',
        textColor: 'var(--success-700)'
      },
      warning: {
        background: 'var(--warning-50)',
        borderColor: 'var(--warning-500)',
        titleColor: 'var(--warning-800)',
        textColor: 'var(--warning-700)'
      },
      error: {
        background: 'var(--error-50)',
        borderColor: 'var(--error-500)',
        titleColor: 'var(--error-800)',
        textColor: 'var(--error-700)'
      }
    };
    
    const variant = variants[props.variant || 'info'];
    
    return `
      background: ${variant.background};
      border-left-color: ${variant.borderColor};
      color: ${variant.textColor};
    `;
  }}
`;

const AlertTitle = styled.h4`
  font-size: var(--text-sm);
  font-weight: var(--font-semibold);
  margin: 0 0 var(--space-1) 0;
`;
```

#### Progress Component
```tsx
interface ProgressProps {
  value: number; // 0-100
  size?: 'sm' | 'md' | 'lg';
  color?: 'primary' | 'success' | 'warning' | 'error';
}

const ProgressContainer = styled.div<ProgressProps>`
  width: 100%;
  background: var(--neutral-200);
  border-radius: 9999px;
  overflow: hidden;
  
  ${props => {
    const sizes = {
      sm: '0.25rem',
      md: '0.5rem',
      lg: '0.75rem'
    };
    
    return `height: ${sizes[props.size || 'md']};`;
  }}
`;

const ProgressBar = styled.div<ProgressProps>`
  height: 100%;
  border-radius: 9999px;
  transition: width 0.3s ease;
  
  ${props => {
    const colors = {
      primary: 'var(--primary-500)',
      success: 'var(--success-500)',
      warning: 'var(--warning-500)',
      error: 'var(--error-500)'
    };
    
    return `
      width: ${props.value}%;
      background: ${colors[props.color || 'primary']};
    `;
  }}
`;
```

---

## Responsive Design Patterns

### Breakpoint System
```css
:root {
  /* Breakpoints */
  --breakpoint-sm: 640px;   /* Small devices */
  --breakpoint-md: 768px;   /* Medium devices */
  --breakpoint-lg: 1024px;  /* Large devices */
  --breakpoint-xl: 1280px;  /* Extra large devices */
  --breakpoint-2xl: 1536px; /* 2X large devices */
}

/* Media query mixins */
@custom-media --sm (min-width: 640px);
@custom-media --md (min-width: 768px);
@custom-media --lg (min-width: 1024px);
@custom-media --xl (min-width: 1280px);
@custom-media --2xl (min-width: 1536px);
```

### Grid System
```css
.container {
  width: 100%;
  margin: 0 auto;
  padding: 0 var(--space-4);
  
  @media (--sm) {
    max-width: 640px;
    padding: 0 var(--space-6);
  }
  
  @media (--md) {
    max-width: 768px;
  }
  
  @media (--lg) {
    max-width: 1024px;
    padding: 0 var(--space-8);
  }
  
  @media (--xl) {
    max-width: 1280px;
  }
  
  @media (--2xl) {
    max-width: 1536px;
  }
}

.grid {
  display: grid;
  gap: var(--space-4);
  
  @media (--sm) {
    gap: var(--space-6);
  }
  
  @media (--lg) {
    gap: var(--space-8);
  }
}

/* Grid column classes */
.col-1 { grid-column: span 1; }
.col-2 { grid-column: span 2; }
.col-3 { grid-column: span 3; }
.col-4 { grid-column: span 4; }
.col-6 { grid-column: span 6; }
.col-12 { grid-column: span 12; }

/* Responsive grid classes */
@media (--md) {
  .md\:col-1 { grid-column: span 1; }
  .md\:col-2 { grid-column: span 2; }
  .md\:col-3 { grid-column: span 3; }
  .md\:col-4 { grid-column: span 4; }
  .md\:col-6 { grid-column: span 6; }
  .md\:col-8 { grid-column: span 8; }
  .md\:col-12 { grid-column: span 12; }
}
```

### Mobile-First Layouts
```tsx
// Mobile-optimized layout component
const ResponsiveLayout = styled.div`
  /* Mobile-first approach */
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  
  /* Tablet and desktop */
  @media (--md) {
    flex-direction: row;
  }
`;

const MobileHeader = styled.header`
  padding: var(--space-4);
  background: white;
  border-bottom: 1px solid var(--neutral-200);
  position: sticky;
  top: 0;
  z-index: 50;
  
  @media (--md) {
    display: none;
  }
`;

const DesktopSidebar = styled.aside`
  display: none;
  
  @media (--md) {
    display: block;
    width: 16rem;
    flex-shrink: 0;
  }
`;

const MainContent = styled.main`
  flex: 1;
  padding: var(--space-4);
  
  @media (--md) {
    padding: var(--space-6);
  }
  
  @media (--lg) {
    padding: var(--space-8);
  }
`;
```

### Touch-Optimized Interfaces
```css
/* Touch target optimization */
.touch-target {
  min-height: 44px;
  min-width: 44px;
  
  @media (pointer: coarse) {
    min-height: 48px;
    min-width: 48px;
  }
}

/* Touch-friendly spacing */
.touch-stack > * + * {
  margin-top: var(--space-4);
  
  @media (pointer: coarse) {
    margin-top: var(--space-6);
  }
}

/* Larger text for mobile */
.mobile-text {
  font-size: var(--text-base);
  
  @media (max-width: 640px) {
    font-size: var(--text-lg);
  }
}
```

---

## Accessibility Standards

### WCAG 2.1 Compliance

#### Color Contrast Requirements
```css
/* Ensure minimum contrast ratios */
:root {
  /* AA compliant color combinations */
  --text-on-primary: white;        /* 4.5:1 on primary-500 */
  --text-on-secondary: #374151;    /* 4.5:1 on neutral-100 */
  --text-on-success: white;        /* 4.5:1 on success-500 */
  --text-on-warning: #374151;      /* 4.5:1 on warning-500 */
  --text-on-error: white;          /* 4.5:1 on error-500 */
}

/* High contrast mode support */
@media (prefers-contrast: high) {
  :root {
    --primary-500: #0056b3;
    --success-500: #006600;
    --warning-500: #cc6600;
    --error-500: #cc0000;
  }
}
```

#### Focus Management
```css
/* Visible focus indicators */
.focus-visible {
  outline: 2px solid var(--primary-500);
  outline-offset: 2px;
  border-radius: 0.25rem;
}

/* Skip links for keyboard navigation */
.skip-link {
  position: absolute;
  top: -40px;
  left: 6px;
  background: var(--primary-500);
  color: white;
  padding: var(--space-2) var(--space-4);
  border-radius: 0.25rem;
  text-decoration: none;
  font-weight: var(--font-medium);
  z-index: 1000;
  
  &:focus {
    top: 6px;
  }
}

/* Focus trap for modals */
.focus-trap {
  &:focus {
    outline: none;
  }
}
```

#### Screen Reader Support
```tsx
// Screen reader utilities
const ScreenReaderOnly = styled.span`
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
`;

// ARIA live regions for dynamic content
const LiveRegion = styled.div`
  position: absolute;
  left: -10000px;
  top: auto;
  width: 1px;
  height: 1px;
  overflow: hidden;
`;

// Usage example
const AccessibleButton = ({ children, ...props }) => (
  <button
    {...props}
    aria-describedby={props['aria-describedby']}
    aria-expanded={props['aria-expanded']}
    aria-pressed={props['aria-pressed']}
  >
    {children}
    {props.loading && (
      <ScreenReaderOnly>Loading...</ScreenReaderOnly>
    )}
  </button>
);
```

#### Keyboard Navigation
```css
/* Keyboard navigation styles */
.keyboard-navigation {
  /* Tab order management */
  &[tabindex="-1"] {
    outline: none;
  }
  
  /* Custom focus styles for complex components */
  &:focus-within {
    box-shadow: 0 0 0 2px var(--primary-500);
  }
}

/* Arrow key navigation for lists */
.arrow-navigation {
  &[role="listbox"] [role="option"] {
    &:focus {
      background: var(--primary-100);
      outline: none;
    }
  }
}
```

### Motion and Animation Preferences
```css
/* Respect reduced motion preferences */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* Subtle animations for better UX */
@media (prefers-reduced-motion: no-preference) {
  .smooth-transition {
    transition: all 0.2s ease;
  }
  
  .fade-in {
    animation: fadeIn 0.3s ease;
  }
  
  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(0.5rem); }
    to { opacity: 1; transform: translateY(0); }
  }
}
```

---

## Interaction Patterns

### Loading States
```tsx
// Loading button state
const LoadingButton = ({ loading, children, ...props }) => (
  <Button disabled={loading} {...props}>
    {loading && (
      <Spinner size="sm" className="mr-2" />
    )}
    {children}
  </Button>
);

// Skeleton loading for content
const SkeletonLoader = styled.div`
  background: linear-gradient(
    90deg,
    var(--neutral-200) 25%,
    var(--neutral-100) 50%,
    var(--neutral-200) 75%
  );
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
  border-radius: 0.25rem;
  
  @keyframes loading {
    0% { background-position: 200% 0; }
    100% { background-position: -200% 0; }
  }
`;
```

### Error Handling
```tsx
// Error boundary component
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return (
        <ErrorFallback 
          error={this.state.error}
          onRetry={() => this.setState({ hasError: false, error: null })}
        />
      );
    }

    return this.props.children;
  }
}

// Error display component
const ErrorFallback = ({ error, onRetry }) => (
  <Card>
    <CardContent className="text-center">
      <AlertCircle className="mx-auto mb-4 text-error-500" size={48} />
      <h3 className="mb-2 text-lg font-semibold">Something went wrong</h3>
      <p className="mb-4 text-neutral-600">
        {error?.message || 'An unexpected error occurred'}
      </p>
      <Button onClick={onRetry}>Try Again</Button>
    </CardContent>
  </Card>
);
```

### Form Validation
```tsx
// Real-time validation feedback
const ValidatedInput = ({ value, validation, ...props }) => {
  const [isValid, setIsValid] = useState(true);
  const [errorMessage, setErrorMessage] = useState('');

  useEffect(() => {
    if (value && validation) {
      const result = validation(value);
      setIsValid(result.isValid);
      setErrorMessage(result.message || '');
    }
  }, [value, validation]);

  return (
    <FormGroup>
      <FormLabel>{props.label}</FormLabel>
      <InputField
        {...props}
        value={value}
        className={`${!isValid ? 'border-error-500' : ''}`}
        aria-invalid={!isValid}
        aria-describedby={!isValid ? `${props.id}-error` : undefined}
      />
      {!isValid && (
        <FormError id={`${props.id}-error`}>
          <AlertCircle size={16} />
          {errorMessage}
        </FormError>
      )}
    </FormGroup>
  );
};
```

---

## Implementation Guidelines

### Component Development Standards
```tsx
// Standard component structure
interface ComponentProps {
  // Required props first
  children: React.ReactNode;
  
  // Optional props with defaults
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  
  // Event handlers
  onClick?: (event: React.MouseEvent) => void;
  
  // Accessibility props
  'aria-label'?: string;
  'aria-describedby'?: string;
  
  // Styling props
  className?: string;
  style?: React.CSSProperties;
}

const Component = React.forwardRef<HTMLElement, ComponentProps>(
  ({ children, variant = 'primary', size = 'md', ...props }, ref) => {
    return (
      <StyledComponent
        ref={ref}
        variant={variant}
        size={size}
        {...props}
      >
        {children}
      </StyledComponent>
    );
  }
);

Component.displayName = 'Component';
```

### Testing Approach
```tsx
// Accessibility testing
import { render, screen } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('component is accessible', async () => {
  const { container } = render(<Component />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

// Responsive testing
test('component adapts to different screen sizes', () => {
  // Test mobile
  Object.defineProperty(window, 'innerWidth', {
    writable: true,
    configurable: true,
    value: 375,
  });
  
  render(<Component />);
  // Assert mobile behavior
  
  // Test desktop
  Object.defineProperty(window, 'innerWidth', {
    writable: true,
    configurable: true,
    value: 1024,
  });
  
  // Assert desktop behavior
});
```

### Performance Considerations
```tsx
// Component optimization
const OptimizedComponent = React.memo(({ data, onAction }) => {
  // Memoize expensive calculations
  const processedData = useMemo(() => {
    return data.map(item => processItem(item));
  }, [data]);

  // Memoize event handlers
  const handleAction = useCallback((id) => {
    onAction(id);
  }, [onAction]);

  return (
    <div>
      {processedData.map(item => (
        <Item 
          key={item.id} 
          data={item} 
          onAction={handleAction}
        />
      ))}
    </div>
  );
});

// Lazy loading for large components
const LazyComponent = React.lazy(() => import('./LazyComponent'));

const ComponentWithSuspense = () => (
  <Suspense fallback={<SkeletonLoader />}>
    <LazyComponent />
  </Suspense>
);
```

---

## Quality Assurance

### Design Review Checklist
- [ ] **Visual Consistency** - Components follow design system standards
- [ ] **Accessibility** - WCAG 2.1 AA compliance verified
- [ ] **Responsive Design** - Works across all target devices
- [ ] **Touch Optimization** - Appropriate for industrial touch devices
- [ ] **Performance** - Optimized loading and interaction times
- [ ] **Browser Compatibility** - Tested across target browsers
- [ ] **User Testing** - Validated with actual user workflows

### Implementation Standards
- [ ] **TypeScript Types** - Complete type definitions
- [ ] **Component Props** - Proper prop validation and defaults
- [ ] **Error Handling** - Graceful error states and recovery
- [ ] **Loading States** - Appropriate loading indicators
- [ ] **Testing Coverage** - Unit and integration tests
- [ ] **Documentation** - Component usage documentation
- [ ] **Performance** - Optimized rendering and memory usage

This design system provides the foundation for a consistent, accessible, and efficient user experience across the entire CleanStation digitalization platform, ensuring both usability in industrial environments and compliance with medical device manufacturing standards. 